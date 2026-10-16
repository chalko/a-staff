# Administrative Staff Roster & Non-Human Identity Directory

**Status:** Approved
**Date:** 2026-10-09

---

## 1. Doctrinal Purpose & Custody

In accordance with the Commander's Planning Guidance (CPG) and MCTP 3-30A (_Command and Staff Action_), **S-1 Personnel & Administration** maintains the authoritative administrative roster, Non-Human Identity (NHI) mappings, and access scopes across Camp Colt.

This directory formalizes the active command billets, agent template definitions, execution providers, watch postures, and operational responsibilities for all staff agents and ephemeral workers.

---

## 2. Command & Staff Billets Directory

| Billet                    | Session Alias      | Agent Template          | Execution Provider | Watch Posture | Primary Cognizance & Mission                                                                                                                    |
| :------------------------ | :----------------- | :---------------------- | :----------------- | :------------ | :---------------------------------------------------------------------------------------------------------------------------------------------- |
| **Chief of Staff**        | `cos` (`colt-cos`) | `agents/cos/`           | Antigravity        | `always`      | Operational lead; cross-staff coordination; reviewing and approving completed staff work; enforcing planning cadence and 14-day gates.          |
| **Mayor**                 | `mayor`            | `agents/mayor/`         | Antigravity        | `always`      | Headquarters commandant; municipal plumbing, Dolt database integrity, session reconciler hygiene, and order lifecycle management (`gc reload`). |
| **S-1 Personnel & Admin** | `s1-personnel`     | `agents/s1-personnel/`  | Antigravity        | `always`      | Doctrinal records custodian of `rigs/a-staff`; NHI permissions, RBAC, access policies, credential audits, and administrative style compliance.  |
| **S-2 Intelligence**      | `s2-intel`         | `agents/s2-intel/`      | Antigravity        | `on_demand`   | Upstream pipeline scouting (`gastownhall/gascity/issues`), terrain analysis, codebase indexing, and security/threat evaluation.                 |
| **S-3 Operations**        | `s3-operations`    | `agents/s3-operations/` | Antigravity        | `always`      | Tactical operations lead; convoy and task bead sequencing; worker stagnation detection; Operations Watch patrol protocol.                       |
| **S-4 Logistics**         | `s4-logistics`     | `agents/s4-logistics/`  | Antigravity        | `always`      | Digital supply depots (Gitea repo provisioning), token draw tracking, thermal/power telemetry rollups, and storage quotas.                      |
| **S-6 Signal & Cyber**    | `s6-signal`        | `agents/s6-signal/`     | Antigravity        | `always`      | Communications watchstander; gateway routing, proxy sanitization, automated priority mail watchdog (`signal_mail_watch.sh`), and net watch.     |
| **Platform SRE**          | `colt-sysadmin`    | `agents/colt-sysadmin/` | Antigravity        | `always`      | Platform infrastructure lead; Proxmox hypervisors, K8s cluster, GPU nodes (Grace Blackwell GB10), and hardware alerts.                          |

---

## 3. S-2 Intelligence Officer Billet Specification

Stood up formally per Command Directive `cc-wisp-bd3wa` on 2026-10-09:

- **Billet Title**: S-2 Intelligence Officer
- **Identifier**: `s2-intel`
- **Template Path**: `agents/s2-intel/prompt.template.md`
- **Execution Provider**: Antigravity
- **Watch Posture**: `on_demand` (Activated for recon sprints, research tasks, and backlog priming)
- **Reporting Chain**: Reports to Chief of Staff (`colt-cos`); directly supports S-3 Operations (`s3-operations`).

### 3.1 Mission & Core Responsibilities

1. **Upstream Pipeline Scouting**: Continually scout open issues on `gastownhall/gascity` (<https://github.com/gastownhall/gascity/issues>). Identify well-specified, high-value tasks, decompose them into candidate beads, and prime the backlog for S-3 tracking convoys.
2. **Terrain Analysis & Codebase Recon**: Analyze repository architecture, dependency trees, AST indices, and cross-rig interfaces across Camp Colt rigs (`colony`, `gascity`, `suitcase-ai`, `colt-utils`).
3. **Threat Evaluation & Security Posture**: Evaluate third-party dependencies, license compliance, credential leakage risks, and safe execution boundaries for tactical agent workers.
4. **Reconnaissance Sprints**: Conduct focused investigative sprints on complex bugs, performance anomalies, or architectural shifts upon tasking from `colt-cos` or `s3-operations`.

### 3.2 Operating Rules for S-2

- **Read-Only Recon Posture**: S-2 does not apply modifications to production codebases during reconnaissance sprints. Deliverables take the form of structured intelligence reports, architecture notes, and proposed task beads.
- **Empirical Receipts**: All findings must cite concrete file paths, line ranges, commit hashes, or external issue URLs.
- **Intel Delivery Tag**: Dispatches candidate tasks and reconnaissance findings to `s3-operations` via `gc mail send s3-operations` with the tag `[INTEL]`.

---

## 4. Least-Privilege Scoping & NHI Security

In compliance with CPG Operational Invariants #3 and #4:

1. **Unprivileged Execution**: All staff and worker sessions execute under unprivileged POSIX identities (`camp-colt-mayor-agent` or containerized worker UID).
2. **Zero Standing Root**: No agent session maintains standing sudo or root access.
3. **Zero Plaintext Secrets**: Credentials and API tokens are never written to disk unencrypted; memory-backed `/dev/shm` credentials or Vault-injected runtime variables are enforced.
4. **Tool Access Scoping**: On-demand staff agents are equipped only with tool suites required for their cognizance (e.g., read-only repo inspection tools for S-2 reconnaissance).

---

## 5. Audit & Maintenance Rhythm

- **Administrative Audit**: S-1 conducts periodic scans of active sessions (`gc session list --json`) against this roster to ensure all running sessions map to authorized billets.
- **Roster Updates**: Changes to billet allocations, prompt templates, or session postures require formal review by S-1 and approval by the Chief of Staff.
