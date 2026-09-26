# Standard Operating Procedure: Repository & Supply Depot Provisioning

**Document ID:** `SOP-4.1-REPO-PROVISION`
**Effective Date:** September 22, 2026
**Lead Action Officer:** S-4 Logistics (`s4-logistics`), Camp Colt
**Reviewing Authority:** Chief of Staff (`colt-cos`), Camp Colt
**Doctrine Standard:** USMC Staff Organization (MCRP 3-10A.1) & Completed Staff Work (MCTP 3-30A)

---

## 1. Purpose & Doctrinal Authority

Under Camp Colt doctrine (established in `AGENTS.md` and `SOP-0.2-WATCH-POSTURE`), Git repositories are classified as **Logistics Supply Depots (Class IX/VII digital material)**.

The S-4 Logistics Officer (`s4-logistics`) holds sole administrative cognizance over:

1. Requisition intake and inventory tracking for all code depots under the `colt/` organization.
2. Creation, sizing, and configuration of remote repositories on in-cluster Gitea (`gitea.colt.chalko.com`).
3. Setting repository access policies, branch protection rules, and upstream synchronization mirrors.
4. Issuing verified depot coordinates to S-3 Operations and the Mayor for municipal integration.

Platform SRE (`colt-sysadmin`) maintains the underlying Gitea K8s pod/storage class facility; S-4 manages the inventory and depots inside the facility.

---

## 2. Depot Provisioning Workflow

```
[S-3 Operations]          [S-4 Logistics]                 [Mayor]
       │                         │                           │
       ├─ Requisition Depot ────►│                           │
       │  (Name, Scope, Quota)   │                           │
       │                         ├─ 1. Authenticate via Token│
       │                         ├─ 2. Provision Gitea Repo  │
       │                         ├─ 3. Set Teams & Quotas    │
       │                         ├─ 4. Verify SSH Read/Write │
       │                         │                           │
       │◄─ Requisition Green ────┼─ Depot Delivery ─────────►│
       │                         │  (SSH URL, Rig Prefix)    │
       │                         │                           ├─ Clone & Mount
       │                         │                           └─ city.toml Update
```

### Step 1: Requisition Intake

Requisitions arrive via `gc mail` from S-3 Operations (tactical requirement) or Chief of Staff (strategic directive). Requisitions must specify:

- **Repository Name:** e.g., `colt/a-staff.git`
- **Rig Prefix:** 2-3 character identifier (e.g., `as`)
- **Description:** Purpose and scope
- **Default Branch:** Standard is `main`
- **Access Level:** Internal (authenticated agents) vs. Public mirror

### Step 2: Gitea API Execution

S-4 executes depot creation using the scoped Gitea token (`repo:*`, `org:read`):

```bash
# Provision repository under the 'colt' organization
curl -s -X POST "http://gitea.colt.chalko.com:3000/api/v1/orgs/colt/repos" \
  -H "Authorization: token ${GITEA_S4_LOGISTICS_TOKEN}" \
  -H "Content-Type: application/json" \
  -d '{
    "name": "<repo-name>",
    "description": "<purpose>",
    "private": false,
    "default_branch": "main",
    "auto_init": true,
    "gitignores": "",
    "license": "",
    "readme": "Default"
  }'
```

### Step 3: Team Permissions & Branch Protection

1. Assign read/write permissions to the `Colt Agents` team.
2. Enable branch protection on `main` (requiring PR review and zero failed automated checks for production rigs).

### Step 4: Empirical Preflight Verification

Before handing off the depot, S-4 must empirically verify SSH transport and repository reachability:

```bash
git ls-remote ssh://git@gitea.colt.chalko.com:30222/colt/<repo-name>.git
```

_Verification criteria:_ Must exit code 0 and list `refs/heads/main`.

### Step 5: Depot Delivery & Handoff

S-4 sends a completion dispatch with `--notify` to both S-3 Operations and the Mayor:

```bash
gc mail send mayor -s "DEPOT ISSUED: colt/<repo-name>.git Provisioned" \
  -m "Mayor,

Depot colt/<repo-name>.git is provisioned and verified ready for municipal mounting.
- Remote URL: ssh://git@gitea.colt.chalko.com:30222/colt/<repo-name>.git
- Suggested Prefix: <prefix>
- Default Branch: main

Execute local clone into rigs/<repo-name>, register in city.toml, and verify with gc doctor.

v/r,
S-4 Logistics" --notify
```

---

## 3. Storage Quota & Capacity Management

1. **Default Quota:** Standard code repositories are allocated a soft ceiling of 1 GB.
2. **Heavy Artifact / Asset Depots:** Repositories holding binary packages, model benchmarks, or dataset caches must be registered in the S-4 Quartermaster Ledger (`docs/S4_DEPOT_INVENTORY.md`) with explicit storage bounds.
3. **No Weights in Git:** Large model weights ($>100\text{ MB}$) must be stored in the ReadyLocker NVMe pool or S3 object store, never committed directly to Gitea repositories.
