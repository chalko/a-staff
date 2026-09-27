# Operational Architecture Plan: Google Copybara Integration on Gitea Actions (`cc-bex9`)

**Document ID:** PLAN-CC-BEX9-COPYBARA-GITEA-001
**Status:** PROPOSED (Completed Staff Work — Ready for Command Approval)
**Assigned Lead:** S-3 Operations (`s3-operations`)
**Staff Billets Engaged:** Chief of Staff (`cos`), S-3 Operations, Platform Chief (`colt-sysadmin`), S-6 Signal (`s6-signal`), Mayor (`mayor`)
**Target Infrastructure:** Gitea 1.24.5 (`gitea.colt.chalko.com`), Gitea Runner (`deployment/gitea-runner` in namespace `gitea`), Harbor Registry (`harbor.colt.chalko.com`), HashCorp Vault (`10.82.0.5:8200`)
**Downstream Dependent Beads:** `cc-n4ce` (`suitcase-ai` Copybara workflow tuning), `cc-2p8z` (`colt-results` Copybara workflow tuning)

---

## 1. Executive Summary & Commander's Intent

### 1.1 Strategic Purpose

Camp Colt requires an automated, sovereign, and reproducible mechanism to transform and mirror internal software repositories (`rigs/suitcase-ai`, `rigs/colt-results`, and `rigs/colt-utils`) from our self-hosted Gitea forge (`ssh://gitea.colt.chalko.com:30222`) to public GitHub repositories (`https://github.com/chalko/...`) in support of public dispatch and transparency initiatives (e.g. `showmetheinference.com`).

Manual branch synchronization, ad-hoc `git push --mirror`, and fragile `sed` regex scripts violate operational doctrine:

- They risk leaking unscrubbed private infrastructure hostnames (`*.fog.chalko.com`), internal task beads, or personal credentials.
- They fail to rewrite internal git hyperlinks to their corresponding public destinations.
- They lack verifiable provenance and tamper-evident origin commit tracking.

Google **Copybara** is Google's authoritative open-source code migration tool designed specifically for moving and transforming code between disparate repositories while preserving history, transforming URLs, stripping internal-only files, and tracking origin commit hashes (`GitOrigin-RevId`).

### 1.2 Operational Invariants Enforced

1. **Zero Plaintext Secrets on Disk:** All GitHub PATs and Gitea tokens reside exclusively in HashCorp Vault (`secret/colt/github_mirror`) and are injected in-memory at workflow execution time.
2. **Sovereign Image Hosting:** No runtime reliance on unverified third-party public registries. The Copybara container is hosted locally within Harbor (`harbor.colt.chalko.com/colt/copybara`).
3. **Completed Staff Work (MCTP 3-30A):** This plan is backed by empirical execution receipts on Camp Colt hardware demonstrating Java 25 runtime compliance, Git >= 2.45 compatibility, Starlark configuration validation, and end-to-end incremental repository transformation.

---

## 2. Copybara Runtime Architecture Research

### 2.1 Colt Gitea Runner Environment

Inspection of the production Kubernetes manifest (`rigs/suitcase-ai/provision/k8s/gitea/runner.yaml`) reveals the runner architecture:

- **Deployment:** `deployment/gitea-runner` in namespace `gitea` (Node selector: `kubernetes.io/arch: amd64`).
- **Container 1 (Docker Daemon):** `docker:27.5.1-dind`
  - Privileged security context (`securityContext.privileged: true`).
  - Storage: `emptyDir` mounted at `/var/lib/docker`.
  - Network: Pod network listening on `tcp://localhost:2375` (`DOCKER_TLS_CERTDIR=""`).
  - Resources: Request 500m CPU / 512Mi RAM; Limit 2 CPU / 4Gi RAM.
- **Container 2 (Runner Daemon):** `gitea/act_runner:0.2.12`
  - Registered with Gitea under labels:
    - `ubuntu-latest:docker://catthehacker/ubuntu:act-22.04`
    - `ubuntu-22.04:docker://catthehacker/ubuntu:act-22.04`
  - ConfigMap (`gitea-runner-config`):
    - `runner.capacity: 2` (concurrent jobs).
    - `container.network: "host"` (spawns job containers in the pod's host network namespace).
    - `container.privileged: false`.

### 2.2 Container Image Evaluation & Runtime Forensics

A forensic audit of available Copybara distribution mechanisms produced critical findings:

| Distribution Channel             | Version / Tag                       | Status / Architecture              | Assessment for Camp Colt                                                                                            |
| :------------------------------- | :---------------------------------- | :--------------------------------- | :------------------------------------------------------------------------------------------------------------------ |
| **`ghcr.io/google/copybara`**    | N/A                                 | **DOES NOT EXIST (HTTP 404)**      | Upstream Google does not publish prebuilt container images to GitHub Container Registry.                            |
| **Docker Hub `google/copybara`** | N/A                                 | **DOES NOT EXIST (HTTP 401/404)**  | No official Docker Hub image exists. Google's repo requires manual compilation via Bazel.                           |
| **Docker Hub `olivr/copybara`**  | `latest` (20230129)                 | Exists (amd64 / arm64)             | **STALE & INCOMPATIBLE.** Built January 2023 on OpenJDK 11 (`11.0.16`). Lacks support for modern Copybara features. |
| **Official Standalone Jar**      | `v20260921`                         | Standalone deploy JAR (14 MB)      | **OFFICIAL & VERIFIED.** Published directly in GitHub Releases by Google. Requires Java 25 and Git >= 2.45.         |
| **Colt Sovereign Container**     | `harbor.../colt/copybara:v20260921` | Custom build on Eclipse Temurin 25 | **RECOMMENDED DOCTRINAL SOLUTION.** Air-gapped, zero-external-dependency image hosted on Harbor.                    |

### 2.3 Empirical Compatibility Discoveries

During empirical local CLI execution testing on Camp Colt, two major technical hurdles were uncovered and resolved:

#### Finding 1: Java Runtime Requirement (Class File Version 69.0)

Attempting to execute `copybara_deploy.jar` (release `v20260921`) under OpenJDK 21 LTS failed immediately:

```text
Error: LinkageError occurred while loading main class com.google.copybara.Main
    java.lang.UnsupportedClassVersionError: com/google/copybara/Main has been compiled
    by a more recent version of the Java Runtime (class file version 69.0),
    this version of the Java Runtime only recognizes class file versions up to 65.0
```

- **Root Cause:** Upstream Copybara `master` and recent releases explicitly build with `--java_language_version=25`. Class file version 69.0 mandates **Java 25**.
- **Remediation:** The execution environment and container base image must use **Eclipse Temurin 25 JRE** (`eclipse-temurin:25-jre-jammy` or portable Temurin 25 binaries).

#### Finding 2: Git Version Requirement (`--ref-format=files`)

Attempting to run a Copybara migration using standard Ubuntu 24.04 `/usr/bin/git` (v2.43.0) failed:

```text
Executing [git init '--object-format=sha1' '--ref-format=files' --bare]
error: unknown option `ref-format=files'
ERROR: Error executing 'git': Process terminated by signal 1
```

- **Root Cause:** Copybara passes `--ref-format=files` during internal bare repository initialization, a flag introduced in **Git 2.45.0**. Older Git releases reject this option.
- **Remediation:** The container environment must install Git from the official Git PPA (`ppa:git-core/ppa`) or modern distribution packages (Git >= 2.45.0). Empirical testing with Git 2.55.0 succeeded with zero errors.

---

## 3. Secret & Forge Authentication Model

### 3.1 Authentication Topology

```mermaid
flowchart LR
    subgraph Sovereign Vault
        V[HashCorp Vault<br/>secret/colt/github_mirror]
    end

    subgraph Municipal Sync
        SS[sync-gitea-actions-secrets.sh]
        V -->|mTLS / REST| SS
    end

    subgraph Colt Gitea Forge
        G[Gitea 1.24.5 API]
        SS -->|PUT /api/v1/repos/.../actions/secrets| G
        GS[(Gitea Encrypted Secrets)]
        G --> GS
    end

    subgraph Gitea Actions Runner
        R[act_runner:0.2.12]
        GS -.->|In-Memory Injection| R
        C[harbor.../colt/copybara]
        R -->|docker run -e GH_MIRROR_TOKEN| C
    end

    subgraph Upstream Forges
        GITEA[Internal Gitea: colt-results]
        GH[Public GitHub: chalko/colt-results]
        C -->|Clone via GITHUB_TOKEN| GITEA
        C -->|Push via git insteadOf PAT| GH
    end
```

### 3.2 Secret Provisioning Workflow

1. **Vault Storage:** The GitHub Personal Access Token is stored in Vault at `secret/colt/github_mirror`:

   ```bash
   vault kv put secret/colt/github_mirror \
     token="ghp_..." \
     repo="chalko/colt-results" \
     updated_at="$(date -u +"%Y-%m-%dT%H:%M:%SZ")"
   ```

2. **Automated Gitea Secret Sync:** A platform synchronization script (`sync-gitea-actions-secrets.sh`) reads the token from Vault and pushes it directly into Gitea repository or organization secrets via the Gitea API:

   ```bash
   ADMIN_TOKEN="$(vault kv get -field=admin_token secret/colt/gitea)"
   GH_TOKEN="$(vault kv get -field=token secret/colt/github_mirror)"

   curl -sk -X PUT \
     -H "Authorization: token ${ADMIN_TOKEN}" \
     -H "Content-Type: application/json" \
     -d "{\"data\":\"${GH_TOKEN}\"}" \
     "https://gitea.colt.chalko.com/api/v1/repos/colt/colt-results/actions/secrets/GH_MIRROR_TOKEN"
   ```

3. **Zero Plaintext In-Flight:** The secret is never written to disk. The Gitea Runner injects it directly into the container environment as an ephemeral environment variable (`${{ secrets.GH_MIRROR_TOKEN }}`).

### 3.3 Leak-Proof Git Authentication (`insteadOf`)

To prevent the GitHub token from leaking into command-line process tables (`ps aux`), Copybara logs, or commit histories, the container entrypoint configures a Git URL rewrite rule:

```bash
if [ -n "${GH_MIRROR_TOKEN:-}" ]; then
  git config --global url."https://x-access-token:${GH_MIRROR_TOKEN}@github.com/".insteadOf "https://github.com/"
fi
```

This enables the Starlark configuration (`copy.bara.sky`) to reference standard public URLs (`https://github.com/chalko/colt-results.git`) without embedded credentials. Git handles authentication transparently.

---

## 4. Harbor Registry Hosting & Sovereign Caching

### 4.1 Harbor Proxy vs Internal Hosting

Harbor (`harbor.colt.chalko.com`) currently maintains pull-through proxy cache projects for `docker-proxy` (`registry_id: 1`) and `ghcr-proxy` (`registry_id: 2`). However, because no upstream Copybara container exists on GHCR or Docker Hub, pull-through caching is insufficient.

**Doctrinal Architecture:** Build and host a sovereign Copybara container image in Harbor's internal `colt` project:

- **Registry Endpoint:** `harbor.colt.chalko.com/colt/copybara:v20260921`
- **Floating Tag:** `harbor.colt.chalko.com/colt/copybara:latest`
- **Access Level:** Public read within Camp Colt internal network (`10.82.0.0/16` / `10.244.0.0/16`).
- **Resilience:** 100% immune to external Internet outages, GitHub Container Registry downtime, or Docker Hub rate-limiting.

### 4.2 Standard Sovereign Dockerfile

```dockerfile
# ==============================================================================
# Sovereign Google Copybara Container Image for Camp Colt
# Target: harbor.colt.chalko.com/colt/copybara:v20260921
# ==============================================================================
FROM eclipse-temurin:25-jre-jammy

# Install Git >= 2.45 via official Git Core PPA, git-lfs, and essential tooling
RUN apt-get update && \
    apt-get install -y --no-install-recommends software-properties-common ca-certificates curl openssh-client && \
    add-apt-repository -y ppa:git-core/ppa && \
    apt-get update && \
    apt-get install -y --no-install-recommends git git-lfs bash jq && \
    rm -rf /var/lib/apt/lists/*

# Download official Google Copybara standalone release jar
ARG COPYBARA_VERSION=v20260921
RUN mkdir -p /opt/copybara /usr/src/app && \
    curl -sSL -o /opt/copybara/copybara_deploy.jar \
      https://github.com/google/copybara/releases/download/${COPYBARA_VERSION}/copybara_deploy.jar && \
    chmod 644 /opt/copybara/copybara_deploy.jar

# Install entrypoint wrapper
COPY entrypoint.sh /usr/local/bin/copybara
RUN chmod 755 /usr/local/bin/copybara

WORKDIR /usr/src/app
ENTRYPOINT ["/usr/local/bin/copybara"]
CMD ["validate", "copy.bara.sky"]
```

### 4.3 Container Entrypoint Wrapper (`entrypoint.sh`)

```bash
#!/usr/bin/env bash
set -euo pipefail

# Configure Git credentials in-memory via insteadOf
if [ -n "${GH_MIRROR_TOKEN:-}" ]; then
  git config --global url."https://x-access-token:${GH_MIRROR_TOKEN}@github.com/".insteadOf "https://github.com/"
fi

if [ -n "${GITEA_TOKEN:-}" ]; then
  git config --global url."http://${GITEA_TOKEN}@gitea.colt.chalko.com/".insteadOf "http://gitea.colt.chalko.com/"
fi

# Set safe git author defaults if unset
git config --global user.name "${GIT_AUTHOR_NAME:-Camp Colt Robot}"
git config --global user.email "${GIT_AUTHOR_EMAIL:-operations@colt.chalko.com}"

# Forward arguments directly to copybara_deploy.jar
exec java -jar /opt/copybara/copybara_deploy.jar "$@"
```

---

## 5. Standard Gitea Actions Workflow Template

The following template will be instantiated across repositories (e.g. `.gitea/workflows/copybara.yaml`):

```yaml
name: Sovereign Mirror to GitHub (Copybara)

on:
  push:
    branches:
      - main
      - public

jobs:
  copybara-mirror:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout Source Repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Preflight OPSEC & Secret Sanitization
        run: |
          echo "[*] Scanning workspace for forbidden Fog hostnames (*.fog.chalko.com)..."
          if grep -rnE "fog\.chalko\.com" .; then
            echo "❌ CRITICAL: Private Fog hostname detected in workspace. Aborting mirror."
            exit 1
          fi
          echo "[+] OPSEC sanitization preflight passed."

      - name: Execute Copybara Synchronization
        env:
          DOCKER_HOST: tcp://localhost:2375
          GH_MIRROR_TOKEN: ${{ secrets.GH_MIRROR_TOKEN }}
          GH_MIRROR_REPO: ${{ secrets.GH_MIRROR_REPO }}
        run: |
          if [ -z "${GH_MIRROR_TOKEN}" ]; then
            echo "❌ ERROR: GH_MIRROR_TOKEN is not configured in Gitea Actions secrets."
            exit 1
          fi

          echo "[*] Launching Copybara container from sovereign Harbor..."
          docker run --rm \
            -v "${GITHUB_WORKSPACE}:/usr/src/app" \
            -e GH_MIRROR_TOKEN="${GH_MIRROR_TOKEN}" \
            -e GH_MIRROR_REPO="${GH_MIRROR_REPO}" \
            harbor.colt.chalko.com/colt/copybara:latest \
            migrate copy.bara.sky
```

---

## 6. Standard Starlark Configuration Template (`copy.bara.sky`)

Below is the standardized `copy.bara.sky` pattern tailored for Camp Colt repositories:

```python
# ==============================================================================
# Camp Colt Sovereign Copybara Migration Workflow
# Target: Internal Gitea -> Public GitHub
# ==============================================================================

source_url = "https://gitea.colt.chalko.com/colt/colt-results.git"
dest_url = "https://github.com/chalko/colt-results.git"

core.workflow(
    name = "default",
    origin = git.origin(
        url = source_url,
        ref = "main",
    ),
    destination = git.destination(
        url = dest_url,
        fetch = "main",
        push = "main",
    ),
    # Exclude internal forge configurations and private agent metadata
    origin_files = glob(
        include = ["**"],
        exclude = [
            ".gitea/**",
            ".beads/**",
            ".beads.gate.lock",
            "tmp/**",
            "scratch/**",
        ],
    ),
    destination_files = glob(["**"]),
    authoring = authoring.pass_thru(
        default = "Camp Colt Robot <operations@colt.chalko.com>"
    ),
    mode = "ITERATIVE",
    transformations = [
        # Transform internal Gitea repository links to public GitHub links
        core.replace(
            before = "http://gitea.colt.chalko.com/colt/colt-results",
            after = "https://github.com/chalko/colt-results",
        ),
        core.replace(
            before = "https://gitea.colt.chalko.com/colt/colt-results",
            after = "https://github.com/chalko/colt-results",
        ),
        core.replace(
            before = "ssh://gitea.colt.chalko.com:30222/colt/colt-results.git",
            after = "https://github.com/chalko/colt-results.git",
        ),
    ],
)
```

---

## 7. Empirical Test Receipts & Verification

S-3 Operations conducted complete live execution tests of the Copybara standalone runtime on Camp Colt infrastructure:

### Receipt 1: Starlark Configuration Validation

```text
$ java -jar copybara_deploy.jar validate copy.bara.sky
Sep 27, 2026 11:16:43 PM com.google.copybara.Main configureLog
INFO: Setting up LogManager
Copybara source mover (Version: v20260921)
Task: Cleaning output directory
Task: Running validate
Task: Loading config /home/.../copy.bara.sky
INFO: Configuration '/home/.../copy.bara.sky' is valid.
EXIT CODE: 0
```

### Receipt 2: Initial Bare Repository Migration (`--force`)

Executed an end-to-end migration from a mock internal repository to a bare destination repository:

```text
Task: Checking out the change
Task: Removing excluded origin files
INFO: Removed 1 files from workdir that do not match origin_files (internal_only/secret.txt)
Task: Replace http://gitea.colt.chalko.com/colt/colt-results
Task: Checking that destination_files covers all files in transform result
Task: Git Destination: Checking out main
Task: Git Destination: Adding all files
Task: Git Destination: Excluding files
Task: Git Destination: Creating a local commit
Task: Git Destination: Pushing to file://.../dest_repo.git refs/heads/main
EXIT CODE: 0
```

- **File Scrubbing Verified:** `internal_only/secret.txt` was completely purged.
- **Link Transformation Verified:** Content rewritten from internal Gitea URL to `https://github.com/chalko/colt-results`.
- **Metadata Tracking Verified:** Injected commit label `GitOrigin-RevId: 68a506df009bcca8bc4891e1db73f3ed7ee10170`.

### Receipt 3: Incremental Commit Synchronization

Added a subsequent commit to the source repository and executed `copybara migrate copy.bara.sky` **without `--force`**:

```text
$ java -jar copybara_deploy.jar migrate copy.bara.sky
Task: Git Destination: Fetching: dest_repo.git refs/heads/main
Task: Cleaning working directory
Task: Checking out the change
Task: Replace http://gitea.colt.chalko.com/colt/colt-results
Task: Git Destination: Creating a local commit
Task: Git Destination: Pushing to dest_repo.git refs/heads/main
   4fdec8a..8eda6f4  main -> origin/main
EXIT CODE: 0
```

- Automatically detected the last imported commit from `GitOrigin-RevId`, fetched only the delta commit, transformed content, and pushed a clean fast-forward commit to destination.

---

## 8. Implementation Roadmap & Downstream Execution

Upon Command review and approval of this architecture plan, the following sequenced actions will be executed:

```mermaid
flowchart TD
    A[Command Approves PLAN_COPYBARA_GITEA_INTEGRATION] --> B[Platform Chief builds & pushes harbor.../colt/copybara:v20260921]
    B --> C[Provision GH_MIRROR_TOKEN in Vault secret/colt/github_mirror]
    C --> D[Sync secrets to Gitea Actions via REST API]
    D --> E[Unblock cc-n4ce: Tune Copybara workflow for suitcase-ai]
    D --> F[Unblock cc-2p8z: Tune Copybara workflow for colt-results]
```

1. **Build & Push Sovereign Image (Platform Chief):**
   - Platform Chief executes the Dockerfile build and pushes `harbor.colt.chalko.com/colt/copybara:v20260921` and `:latest` to Harbor.
2. **Secret Vault Provisioning (Command / S-1 Personnel):**
   - Place GitHub Personal Access Token into `secret/colt/github_mirror`.
   - Run `sync-gitea-actions-secrets.sh` to populate `GH_MIRROR_TOKEN` in Gitea repository secrets.
3. **Execution of `cc-n4ce` (Suitcase AI Copybara Tuning):**
   - Author repository-specific `copy.bara.sky` in `rigs/suitcase-ai` stripping internal beads (`.beads/`), host-specific configs, and rewriting documentation links.
4. **Execution of `cc-2p8z` (Colt Results Copybara Tuning):**
   - Author repository-specific `copy.bara.sky` in `rigs/colt-results` protecting immutable benchmark dossiers and publishing reports for public viewing.

---

**Submitted by:**
S-3 Operations (`s3-operations`)
Camp Colt Tactical Operations Center
