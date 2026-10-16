# Camp Colt Staff Style Guide & Operational Lexicon

## 1. Fundamental Policy

> _"Agent Staff takes its inspiration and form from the USMC staff process, but we are not playing soldiers here."_
> — Commander's Planning Guidance (CPG) & Command Clarification

Camp Colt employs the functional Marine Corps staff process to achieve structured division of responsibility, rigorous completed staff work, role specialization, and operational continuity across autonomous agent teams.

However, staff doctrine must directly serve **engineering reality**. Applying combat arms terminology to software engineering, cloud infrastructure, or customer data is actively harmful:

- It obscures empirical engineering receipts behind theatrical jargon.
- It introduces cognitive distraction and false analogies (e.g., viewing civilian users through an artillery "fires" mindset).
- It generates faux classification stamps that misrepresent an open-source, sovereign homelab.

The rule of thumb across all Camp Colt documentation and communications is:
**Retain functional, clarifying staff doctrine. Ruthlessly eliminate theatrical military roleplay (LARPing).**

---

## 2. The Operational Lexicon

### 2.1 Functional & Doctrinally Sound Staff Concepts (RETAIN & ENFORCE)

These concepts provide genuine structural discipline and are official Camp Colt doctrine:

1. **Staff Billets (S-1 through S-6, Platform SRE, Mayor, CoS)**:
   - Clear division of responsibility: S-1 (Personnel/Records/Admin), S-2 (Intel/Recon/Upstream Pipeline Scouting), S-3 (Operations/Execution), S-4 (Logistics/Repos/Compute), S-6 (Signal/Comms/Gateway), Platform SRE (Hardware/K8s), Mayor (Municipal plumbing), CoS (Coordination/Approvals).
2. **Completed Staff Work (MCTP 3-30A)**:
   - Work submitted to Command or CoS must be complete, tested, self-contained, and accompanied by empirical verification receipts and simple reproduction steps.
3. **Operations Watch & Communications Watch**:
   - The operational posture of monitoring queues, queue reconciliation, worker pinging, radio/net watch, and incident response.
4. **Commander's Planning Guidance (CPG)**:
   - Clear, authoritative articulation of strategic intent, operational invariants, hard boundaries, and explicit divestments.
5. **Staff Cadence & Briefs**:
   - Regular, predictable coordination cadences (such as Commander's Update Briefs and After-Action Reviews).

---

### 2.2 Misapplied & Harmful Jargon (PROHIBITED & REPLACED)

The following terms are banned across technical reports, plans, telemetry documents, and agent communications:

| Prohibited Theatrical Jargon                                                            | Authorized Engineering Terminology                         | Context & Rationale                                                                                                              |
| :-------------------------------------------------------------------------------------- | :--------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------- |
| `Classification: Sovereign Operations` / `Classification: Sovereign Appliance Internal` | `Document Type: <Technical Category>`                      | Delete faux classification stamps. Security is enforced via OpenSSH CA, Vault, and RBAC—not text markings.                       |
| `Combat Operations Center (COC)` / `Virtual COC`                                        | `Operations Watch` or `Tactical Operations`                | We manage agent swarms and Kubernetes clusters, not live combat zones.                                                           |
| `Fires` / `Atmospherics` (for user metrics or analytics)                                | `User Sentiment`, `Product Telemetry`, or `System Metrics` | Borrowed from artillery doctrine during Operation Iraqi Freedom; completely inappropriate for human users or software telemetry. |
| `Lines of Effort (LOE)` (when misapplied to ad-hoc tasks)                               | `Workstream`, `Initiative`, or `Project Epic`              | Keep project planning clear and focused.                                                                                         |
| `War Order` / `Battle Order`                                                            | `Operational Directive`, `Task Order`, or `Execution Plan` | Standard engineering order terminology.                                                                                          |
| `War Room` (for technical incidents)                                                    | `Incident Triage` or `Swarm Coordination`                  | Professional systems engineering terminology.                                                                                    |
| `Battle Rhythm`                                                                         | `Staff Cadence` or `Operational Cadence`                   | Regular, predictable coordination cadences for meetings, briefs, and reviews.                                                    |
| `Authority: MCTP 3-30A` (used as a legalistic stamp)                                    | `Standard: Completed Staff Work`                           | MCTP 3-30A provides staff doctrine; it is not a legal statutory authority.                                                       |
| `MAGTF Authority` / `Doctrinal Seal`                                                    | `Reviewing Authority: <Billet>`                            | Retain clear billet accountability.                                                                                              |

---

### 2.3 System Taxonomy & Billet Nomenclature (A-Staff vs. S-Billets)

To prevent cognitive friction, semantic collisions, and broken pattern recognition, the following naming architecture is strictly established:

1. **System & Platform Name: "A-Staff"**

   - Refers to the overarching software platform and autonomous staff system (**Agentic Staff** / **Autonomous Staff in a Box**).
   - Used for repository naming (`colt/a-staff`), project references, and macro architecture.

2. **Internal Staff Sections: Retain "S-1" through "S-6"**

   - The functional sections within an A-Staff command post retain standard Marine Corps battalion staff designations (MCRP 3-10A.1):
     - **S-1**: Personnel & Administration (IAM, RBAC, access policies)
     - **S-2**: Intelligence (Codebase terrain analysis, AST indexing, RAG)
     - **S-3**: Operations (Swarm orchestration, task convoys, watchstanding)
     - **S-4**: Logistics (Digital depots, token budgets, compute/storage quotas)
     - **S-6**: Signal & Cyber (Gateway routing, telemetry, comms sanitization)
     - **Chief of Staff (COS)**: Staff director and coordination gate

3. **Doctrinal & Linguistic Rationale**:

   - **Avoid "A-1" through "A-6"**:
     - _Military Precedent_: In joint military taxonomy, the "A" prefix specifically denotes the Air Force Air Staff (A-1 Air Personnel, A-2 Air Intel, etc.), diverging from our infantry battalion command model.
     - _Pattern Recognition & Semantic Collision_: In civilian and engineering contexts, "A-1" triggers immediate, jarring collisions with steak sauce, credit ratings, or generic prime beef grading.
     - _Clarity_: Calling a billet `S-1` or `S-3` provides immediate, unambiguous recognition for developers and operators without gimmickry.

4. **Identifier & Handle Conventions**:
   - In code, configurations, prompt headers, and session aliases, prefer the direct, clean section handle:
     - Preferred general section handles: `s1`, `s2`, `s3`, `s4`, `s6`, `cos`
     - Authorized subsection & specialized billet handles: Specific functional billets and subsections within a staff section are completely valid (e.g. `s3-ops` for current-ops watch, `s3-plans` for future operations/planning, `s4-logistics`, `s4a-platform`, `s2-intel`).
     - **Prohibited**: Do not invent `a1`, `a-1`, or `astaff-s1`.

---

## 3. Mandatory Document Header Specifications

All formal reports, specifications, and telemetry records in Camp Colt repositories must begin with a standardized header block containing factual metadata.

### 3.1 Technical Benchmark & Telemetry Reports

Used in `rigs/colt-results/reports/`:

```markdown
# <Descriptive Technical Title>

**Document Reference**: `<FILENAME>.md`
**Document Type**: Technical Telemetry & Benchmark Report
**Author / Billet**: <Lead Billet> (e.g. `S-3 Operations (s3-operations)`)
**Governing Task Bead**: <Bead Link>
**Hardware Platform**: ASUS Ascent GX10 (NVIDIA Grace Blackwell GB10 @ `10.82.0.3:8000`)
**Serving Engine**: vLLM (<Engine Parameters>)
**Model Profile**: <Model Identifier>
**Verification Date**: YYYY-MM-DD
**Verification Status**: VERIFIED (<Empirical evidence summary>)

---
```

### 3.2 Engineering Specifications & Architectural Plans

Used in `rigs/suitcase-ai/plans/`, `rigs/suitcase-ai/docs/`, and `colony/docs/`:

````markdown
# <Document Title>

```text
Document ID:     ARCH-COLT-<PROJECT>-<NAME>
Document Type:   System Architecture & Specification
Author / Billet: <Author Billet>
Reviewing Gates: <Reviewing Billets>
Target Enclave:  Camp Colt Sovereign AI Enclave (10.82.0.0/16)
Date:            YYYY-MM-DD
Status:          DRAFT | APPROVED | ACTIVE | COMPLETED
```
````

---

````

### 3.3 After Action Reports (AARs) & Incident Root Cause Analyses
Used in `rigs/suitcase-ai/docs/`, `colony/docs/`, and `rigs/a-staff/plans/`:

```markdown
# After Action Report (AAR): <Incident Name>

**Document ID:** `AAR-YYYY-MM-DD-<NAME>`
**Report Type:** Infrastructure Incident Root Cause Analysis
**Incident Period:** <Timestamp Range>
**Author / Billet:** <Investigating Billet>
**Recipient:** Chief of Staff (`cos`), Commander
**Status:** Resolved / Stabilized
**Target System / Pod:** <Affected Infrastructure>

---
````

### 3.4 Commander's Update Briefs (CUBs)

Used in `colony/docs/`:

```markdown
# Commander's Update Brief (CUB) — Camp Colt

**Briefing ID:** CUB-YYYY-MM-DD-HHMM
**Date / Timestamp:** <Day, DD Month YYYY, HHMM PDT>
**Facilitator:** Chief of Staff (`colt-cos`), Camp Colt
**Operational Context:** Staff Coordination & Operational Status Update
**Target Environment:** Sovereign Command Post (`colt-hq` @ `10.82.0.20`)

---
```

---

## 4. Resolving Borderline or Ambiguous Terms

When authoring documentation or issuing task directives, billets must apply the following decision framework:

1. **Does the term describe a functional administrative/coordination structure?**
   - _Yes_: Retain (e.g. S-3 Operations, Completed Staff Work, Watchstander, Staff Cadence, AAR, CPG).
2. **Does the term borrow tactical combat or kinetic destruction concepts for software or humans?**
   - _Yes_: Replace immediately (e.g. Fires, Atmospherics, COC, War Order, Faux Classification).
3. **When Uncertain**:
   - Ask Command or the Chief of Staff via `gc mail send cos -s "Terminology Clarification: <Term>"`. Do not guess or proliferate theater.

---

## 5. Repository Custody & Governance

Per Command directive:

- **`rigs/a-staff` Custodian**: S-1 Personnel & Administration (`s1-personnel`). S-1 maintains and enforces all staff processes, style compliance, directives, and records management.
- **Underlying Repository Material**: S-4 Logistics (`s4-logistics`) provisions and manages repository assets in Gitea.
- **Infrastructure & Pod Platform**: Platform SRE (`colt-sysadmin`) maintains the Kubernetes runtime, databases, and persistent storage.
