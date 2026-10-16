# Standard Operating Procedure: Weekly Task Friction & Process Improvement Review (AAR)

## 1. Purpose & Guiding Principles

This Standard Operating Procedure establishes the cadence, reporting format, and calibration methodology for the **Weekly Task Friction & Process Improvement Review (After Action Review)** at Camp Colt.

### Core Principles

1. **Zero Supervisor Polling:** Operational leads and staff officers must never manually poll or peek into worker sessions to detect stalled execution. Gas City native supervisor daemons, watchdogs, and patrol orders (`gc patrol` / `gc order`) are the sole authorized automated detection layer.
2. **Systemic Root-Cause Analysis:** Failures, stuck states, or remediation cycles are treated as structural platform or workflow signals, not individual agent anomalies.
3. **Continuous Process Improvement:** Every friction pattern resolved during the week must be hardened into automated tooling, watchdog rules, or playbook SOPs in [`rigs/a-staff/playbooks/`](file:///home/camp-colt-mayor-agent/colony/rigs/a-staff/playbooks/) before closing the weekly cycle.

---

## 2. Battle Rhythm & Governance

- **Occurrence:** Weekly on **Fridays**.
- **Execution Window:** Post-lunch analysis by staff sections; final synthesized review presented to Command **NO LATER THAN 1400 PDT**.
- **Reviewing Authority:** Chief of Staff (`colt-cos`).
- **Coordinating Billets:**
  - **S-3 Operations (Lead):** Compiles cross-rig bead telemetry, calculates dwell times, and chairs the review.
  - **Platform Chief (`colt-sysadmin`):** Audits hypervisor, network, and storage friction.
  - **Mayor (`mayor`):** Audits municipal plumbing, rig mounts, and workspace hygiene.
  - **S-6 Signal (`s6-signal`):** Audits gateway latency, proxy routing, and credential governance.

---

## 3. Four-Pillar Review Scope

Each weekly review systematically evaluates four functional pillars:

### Pillar 1: Bead Dwell-Time & Bottleneck Audit

- **Cross-Rig Store Audit:** Scan all beads across active rigs (`cc-*`, `cr-*`, `sa-*`, `cu-*`, `as-*`, `gtr-*`).
- **Metrics Collected:**
  - Total beads created vs. closed.
  - Number of beads flagged `blocked`, `deferred`, or bounced back during verification.
  - Average Dwell Time: Duration spent in `in_progress` prior to first commit or completion report.
- **Root-Cause Classification:** Categorize delays into: Specification Ambiguity, Environmental Blocker, Upstream Dependency, or Model Hallucination/Looping.

### Pillar 2: Platform & Environmental Friction

- **Filesystem & Mount Boundaries:** Inspect cross-device filesystem behaviors (e.g. multipass-sshfs `EXDEV` link limitations, permissions, broken symlinks).
- **Identity & RBAC:** Review Gitea team access, SSH host key authentication, and unprivileged POSIX constraints.
- **Network & Gateways:** Review DNS resolution, LiteLLM proxy latency, and endpoint availability.
- **Tool-Calling Schema Compliance:** Track model schema violation error rates across OpenCode / Antigravity harnesses.

### Pillar 3: Gas City Watchdog & Patrol Calibration

- **Autonomous Watchdog Rules:** Evaluate active patrol orders and session timeout parameters.
- **Tuning Targets:**
  - Adjust timeout thresholds (e.g., maximum uncommitted turn limits, nudge retry intervals).
  - Verify automated reclaim behaviors for crashed or orphaned worker sessions.
  - Ensure deadlocked sessions trigger proactive mail alerts to S-3 without manual human discovery.

### Pillar 4: Playbook & Procedure Updates

- Formally document remediation recipes for encountered friction.
- Author or update operational procedures in `rigs/a-staff/playbooks/`.
- Ensure Gas City recipes and molecule definitions are updated with newly required verification gates.

---

## 4. Standard Friday 1400 AAR Template

All weekly submissions must utilize the following standardized Markdown format:

```markdown
# Weekly Task Friction & Process Improvement Review (AAR)

**Week Ending:** [YYYY-MM-DD]
**Lead Section:** S-3 Operations, Camp Colt
**Reviewing Authority:** Chief of Staff (`colt-cos`)
**Submission Time:** Friday [HH:MM] PDT (NLT 1400 PDT)

---

### 1. Executive Summary & Garrison Health

[High-level synthesis of weekly throughput, velocity, and top operational blockers encountered.]

### 2. Cross-Rig Bead Dwell-Time & Bottleneck Audit

| Rig / Domain            | Closed Beads | Blocked / Stalled | Dwell Time (>2h) | Primary Bottleneck / Root Cause |
| :---------------------- | :----------: | :---------------: | :--------------: | :------------------------------ | --- |
| `camp-colt` (HQ)        |              |                   |                  |                                 |
| `rigs/colt-results`     |              |                   |                  |                                 |
| `rigs/suitcase-ai`      |              |                   |                  |                                 |
| `rigs/colt-utils`       |              |                   |                  |                                 |
| `rigs/a-staff`          |              |                   |                  |                                 |
| `rigs/go_template_rule` |              |                   |                  |                                 |     |

### 3. Platform & Environmental Friction Log

- **Filesystem / Storage:** [Observations on mounts, I/O, permissions]
- **Authentication / RBAC:** [Gitea, SSH, Vault, sudo hygiene]
- **Tool & Schema Execution:** [Model tool-call errors, parsing exceptions]

### 4. Gas City Watchdog & Automation Tuning

- **Observed Stuck Sessions:** [List any sessions that required intervention]
- **Watchdog / Patrol Changes:** [Specific parameter adjustments to gc orders or session configs]

### 5. Playbook & Procedure Updates

- **New SOPs / Recipes:** [Link newly authored documents in rigs/a-staff/playbooks/]
- **Lessons Learned:** [Key operational takeaways for garrison continuity]
```

---

## 5. Process Compliance Block

**Authored By:**
_S-3 Operations Officer (`s3-operations`)_
Camp Colt Combat Operations Center
Date: 2026-09-23

**Endorsed By:**
_Chief of Staff (`colt-cos`)_
Camp Colt Battalion Headquarters
Authority: Directive `cc-wisp-s05f`
