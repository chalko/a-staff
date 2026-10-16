# Standard Operating Procedure: Operational Planning Cadence, Time Horizons, & Lifecycle Gates

## 1. Purpose & Context

In an autonomous multi-agent software engineering enclave like Camp Colt, tactical polecats and staff billets operate at compute speeds far exceeding traditional human development cycles. However, speed without synchronized planning horizons leads to strategic drift, obsolete task beads, merge friction, and wasted token budgets.

Per Commander's Planning Guidance (CPG) and Command directive, this Standard Operating Procedure (SOP) formalizes:

1. **The Three Operational Planning Horizons**: Scaled definitions and lifecycle bounds for Campaign Plans, Operations Plans (OPORDs), and Fragmentary Orders (FRAGOs).
2. **The 14-Day Expiration & Re-baselining Gate**: A mandatory programmatic circuit-breaker that freezes stale workstreams, re-evaluates codebase terrain, and enforces re-authorization.
3. **Staff Roles & Tooling Integration**: Clear mapping between staff billets, Gas City constructs (Epics, Convoys, Beads), and target repository runbooks.

---

## 2. The Three Operational Planning Horizons

Camp Colt organizes all operational activity into three distinct, interconnected planning horizons:

```
┌───────────────────────────────────────────────────────────────────────────┐
│                     HORIZON 1: CAMPAIGN PLAN                              │
│  • Duration: 30 to 90 Days (1–3 Months)                                   │
│  • Custodian: Chief of Staff (colt-cos)                                   │
│  • Location: camp-colt/plans/                                          │
│  • Focus: Strategic Platform Jumps, Architecture Baselines, CPG Alignment │
└─────────────────────────────────────┬─────────────────────────────────────┘
                                      │ Decomposes Into
                                      ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                     HORIZON 2: OPERATIONS PLAN (OPORD)                    │
│  • Duration: 3 to 14 Days                                                 │
│  • Custodian: S-3 Operations (s3-operations) with Staff Concurrence       │
│  • Tooling Map: Gas City Epics & Convoys                                  │
│  • Location: camp-colt/plans/ & target rig runbooks (e.g. plans/)      │
│  • Focus: Multi-Bead Tactical Workloads & System Capabilities             │
└─────────────────────────────────────┬─────────────────────────────────────┘
                                      │ Adjusts Intraday
                                      ▼
┌───────────────────────────────────────────────────────────────────────────┐
│                     HORIZON 3: FRAGMENTARY ORDER (FRAGO)                  │
│  • Duration: 1 to 24 Hours                                                │
│  • Custodian: S-3 Operations (s3-operations) or Chief of Staff (colt-cos) │
│  • Dispatch: gc mail ... --notify                                         │
│  • Tooling Map: Specific Task Beads & Blocker Beads                       │
│  • Focus: Immediate Tactical Adjustments, Blocker Bypasses, Scope Fixes   │
└───────────────────────────────────────────────────────────────────────────┘
```

---

### 2.1 Horizon 1: Campaign Plan (Macro Strategic Horizon: 30 to 90 Days)

- **Definition**: The long-range strategic blueprint establishing major technical milestones, hardware capability jumps, and sovereign operational posture.
- **Authority & Custody**: Authored and maintained exclusively by the **Chief of Staff (`colt-cos`)** under the direct guidance of the Commander.
- **Canonical Location**: `rigs/a-staff/plans/CAMPAIGN_PLAN_<NAME>.md`.
- **Scope & Focus**:
  - Major hardware migrations (e.g., Package B "Tri-Pillar Jump", Minisforum MS-01, MikroTik 10G SFP+ networking).
  - Cross-cutting benchmark campaigns (e.g., Sparkleader qualification, reasoning effort scaling on NVIDIA Grace Blackwell GB10).
  - High-level platform invariants, token expenditure ceilings, and operational security boundaries.
- **Review Rhythm**: Evaluated bi-weekly during Commander's Update Briefs (CUBs) or upon major platform completion.

---

### 2.2 Horizon 2: Operations Plan / OPORD (Tactical Workload Horizon: 3 to 14 Days)

- **Definition**: Mid-tier operational execution plans bridging strategic campaign intent into organized agent swarms and technical deliverables.
- **Authority & Custody**: Authored by **S-3 Operations (`s3-operations`)** with functional staff concurrence (S-1 administrative compliance, S-4 logistics/storage validation, S-6 signal/gateway routing, Platform SRE infrastructure readiness).
- **Mapping to System Tooling**:
  - Maps 1:1 with **Gas City Epics** and **Convoys** (e.g., Epic `cc-32m`, Convoy `sa-cvo`).
  - High-level OPORDs reside in `rigs/a-staff/plans/` (e.g., `cc-32m-local-models.md`).
  - Detailed technical implementation plans and step-by-step checklists reside directly in the target repository's `plans/` directory (e.g., `rigs/suitcase-ai/plans/hardware_upgrade_and_procurement_plan.md`, `rigs/suitcase-ai/plans/talos_hot_cold_standby_pattern_plan.md`).
- **Core Requirements**:
  - Clear entry criteria, test-driven acceptance gates, and empirical verification receipts.
  - Identification of participating rigs, agent roles, and compute resource bounds.

---

### 2.3 Horizon 3: Fragmentary Order / FRAGO (Immediate Tactical Horizon: 1 to 24 Hours)

**Definition**: Rapid, intra-day operational directives that adjust active tactical execution without invalidating or rewriting the parent OPORD.

**Authority & Custody**: Issued by **S-3 Operations (`s3-operations`)** or the **Chief of Staff (`colt-cos`)**.

**Low-Noise Execution Mechanics**: To prevent administrative sprawl, redundant beads, and unnecessary notification spam across the colony, FRAGOs strictly follow a bifurcated execution pattern:

#### Pattern A: Intra-Task FRAGO (Course Correction on Existing Work)

Used when refining scope, updating implementation details, providing targeted feedback, or clearing local obstacles on an already assigned bead.

- **Zero New Beads / Zero Clutter**: Do **not** create a new bead or send extensive email threads.
- **Dolt Bead Comment**: S-3 appends a formal, timestamped order directly to the existing bead:

  ```bash
  bd comment <bead-id> "FRAGO <nn>: <specific adjustment or directive>"
  ```

- **Session Nudge Alert**: S-3 immediately wakes and directs the assigned worker session:

  ```bash
  gc session nudge <worker-session> "FRAGO <nn> posted on <bead-id>: see bead comments"
  ```

- **Receipt & Audit Trail**: The complete chronological audit trail is preserved permanently in the bead's Dolt version history without generating orphaned task records or email clutter.

#### Pattern B: Scope-Expanding FRAGO (New Workstream / Parallel Assignment)

Used only when an intra-day event requires spinning up a new parallel line of effort or dispatching a separate, independent worker.

- **Distinct Child Bead**: S-3 creates a distinct child bead attached directly to the governing parent Epic/Convoy:

  ```bash
  bd create --title "FRAGO <nn>: <Task Title>" --parent <epic-id> --assignee <worker>
  ```

- **High-Priority Mail Notification**:

  ```bash
  gc mail send <worker> -s "FRAGO <nn> to OPORD <id>: <Task Title>" -m "Assigned to bead <child-bead-id>. See details in bead." --notify
  ```

- **Lifecycle Bound**: Ephemeral; automatically retires upon test verification and closure of the associated task bead.

---

## 3. The 14-Day Expiration & Re-baselining Gate

### 3.1 The Terrain Drift Problem in Autonomous Swarms

Autonomous agent swarms iterate, refactor, and commit code continuously. In this high-velocity environment, an operations plan formulated two weeks prior faces severe **codebase terrain drift**:

- Dependencies, APIs, and data structures may have moved.
- Upstream packages (Gas City core, vLLM, LiteLLM) may have evolved.
- Hardware topologies, network leases, or memory allocations may have shifted.

Allowing stale plans to execute without validation causes agents to hallucinate non-existent files, trigger syntax deadlocks, produce merge conflicts, and burn compute tokens fruitlessly.

---

### 3.2 The Mandatory 14-Day Expiration Rule

> [!IMPORTANT] > **The 14-Day Operational Gate**:
> Any Operations Plan (OPORD), Epic, or Convoy that remains uncompleted after **14 calendar days** from its effective date automatically expires.

Upon expiration:

1. **Immediate Execution Freeze**: All active task beads and polecat worker sessions linked to the expired OPORD must be suspended or placed on hold.
2. **Mandatory Terrain Reconnaissance (S-3 / S-2)**:
   - S-3 Operations conducts an immediate audit of codebase diffs, git branch divergence, and completed sub-tasks.
   - S-2 Intelligence executes codebase re-indexing if structural refactoring occurred.
3. **After Action Review (AAR) & Friction Assessment**:
   - S-3 authors a brief Friction AAR identifying why the operation exceeded its 14-day window (e.g., technical debt, ambiguous requirements, hardware constraints).
4. **Command Re-baselining Gate**:
   - The operation cannot resume until the Chief of Staff and Commander formally re-baseline the OPORD, issue an updated scope, or cancel the lingering backlog.

---

## 4. Governance & Staff Responsibilities Matrix

| Billet                               | Role in Planning Cadence | Primary Responsibilities                                                                                                          |
| :----------------------------------- | :----------------------- | :-------------------------------------------------------------------------------------------------------------------------------- |
| **Commander**                        | Strategic Sponsor        | Approves CPG, Campaign Plans, and 14-day re-baselining gates.                                                                     |
| **Chief of Staff (`colt-cos`)**      | Operational Lead         | Authors Campaign Plans; reviews and approves OPORDs; issues strategic FRAGOs; enforces the 14-day gate.                           |
| **S-1 Personnel (`s1-personnel`)**   | Administrative Custodian | Maintains this SOP; audits documentation format, style compliance, and records retention across all planning horizons.            |
| **S-2 Intelligence (`s2-intel`)**    | Terrain & Pipeline Recon | Scoutes upstream issues (`gastownhall/gascity`), performs codebase indexing, AST terrain analysis, and pre-plan risk assessments. |
| **S-3 Operations (`s3-operations`)** | Tactical Proponent       | Authors OPORDs; sequences task beads; issues tactical FRAGOs; conducts 14-day terrain re-baselining.                              |
| **S-4 Logistics (`s4-logistics`)**   | Resource Validation      | Audits token consumption, storage capacity, and repo provisioning feasibility for proposed plans.                                 |
| **S-6 Signal (`s6-signal`)**         | Communications & Routing | Validates gateway routing, inference telemetry export, and proxy sanitization requirements.                                       |
| **Platform SRE (`colt-sysadmin`)**   | Infrastructure Readiness | Validates K8s compute, GPU availability, and network node capacity before OPORD execution.                                        |
| **Mayor (`mayor`)**                  | Municipal Plumber        | Maintains workspace plumbing, Dolt SQL integrity, and session hygiene throughout the operational rhythm.                          |

---

## 5. Review & Doctrinal Compliance

1. **Doctrinal Alignment**: This SOP complies strictly with the Commander's Planning Guidance (CPG) and the Camp Colt Document Header Style Guide ([`STYLE_GUIDE_AND_LEXICON.md`](STYLE_GUIDE_AND_LEXICON.md)).
2. **Audit Rhythm**: S-1 will conduct a bi-weekly administrative scan of all active beads and plans to ensure zero un-gated operations exceed the 14-day expiration ceiling.
