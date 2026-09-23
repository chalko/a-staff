# MCRP 3-10A.1 — Infantry Battalion Operations

**Full title:** Marine Corps Reference Publication 3-10A.1, _Infantry Battalion
Operations_ **Publication date:** 31 July 2020 **PCN:** 144 000292 00
**Publishing HQ:** Headquarters, United States Marine Corps, Washington, D.C.
20380-1775

## Source & References

- **Official USMC Public Portal PDF:**
  <https://www.marines.mil/Portals/1/Publications/MCRP%203-10A.1.pdf>
- **Converted Markdown (`marker-pdf`):** [MCRP_3-10A.1.md](MCRP_3-10A.1.md)

> [!NOTE] MCRP 3-10A.1 is a publicly released, unclassified USMC publication.
> The local text copy and summary extract are retained in this repository for
> offline-first reference when configuring battalion-level (S-Staff) agent
> orchestration architectures for **Suitcase AI**.

---

## Executive Overview: Battalion Level (S-Staff) vs. General Staff (G-Staff)

While the **General Staff (G-Staff)** structure (MCTP 3-30A) is designed for
high-echelon formations (MEF/Division) with 8+ major staff sections, the
**Infantry Battalion (S-Staff)** structure (MCRP 3-10A.1) is optimized for
compact, lean tactical units led by a Battalion Commander, Executive Officer
(XO), and primary **S-Staff Section Officers (S-1 through S-4, S-6, and optional
S-9)**.

For **Suitcase AI**, downsizing from a 14-agent G-Staff model to a 4-to-5 agent
**Battalion S-Staff model** dramatically reduces token consumption, latency, and
model memory pressure while retaining clear command relationships, explicit task
ownership, and strict administrative discipline.

---

## Primary S-Staff & Command Billets (MCRP 3-10A.1, pp. 1-8 to 1-12)

### 1. Battalion Command & Executive Leadership

- **Commanding Officer (CO)** (p. 1-8):
  - **AI Translation**: _Human Commander / User Interface_.
  - **Role**: Exercises command authority over the battalion. Sets Commander's
    Intent, approves operational plans, and receives synthesized briefings.
- **Executive Officer (XO)** (p. 1-9):
  - **AI Translation**: _Chief of Staff & Information Management Overseer
    (Executive Tier Model)_.
  - **Role**: Second-in-command and principal assistant. Supervises the staff,
    coordinates decision-making, and serves as the overseer and enforcer of the
    **Information Management (IM) Plan**. Oversees special staff (Security, IMO,
    LNOs).
- **Sergeant Major (SgtMaj)** (p. 1-10):
  - **AI Translation**: _Tactical SOP & Discipline Auditor_.
  - **Role**: Senior enlisted advisor; enforces tactical SOP compliance, field
    discipline, and operational execution standards across worker nodes.

---

### 2. Primary Staff Officers (The S-Sections)

| Staff Billet | Name & Title                                  | AI Translation & Core Responsibilities                                                                                                                                                                                                                    |
| :----------- | :-------------------------------------------- | :-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **S-1**      | **Adjutant** (p. 1-10)                        | **Personnel & Access Auditor**: Manages agent lifecycle/allocations, worker prompts, tool access bounds, and administrative status tracking. Works closely with XO and S-4.                                                                               |
| **S-2**      | **Intelligence Officer** (p. 1-10)            | **Intel, RAG & Context Analyst**: Manages vector databases (RAG), indexes workspace context, surfaces codebase facts, and provides threat/environmental analysis to XO and S-3.                                                                           |
| **S-3**      | **Operations Officer** (p. 1-11)              | **Operations & Execution Orchestrator**: Principal planner for operations, training, and execution. Integrates all warfighting functions, decomposes CoS intent into worker tasks, and operates the Combat Operations Center (COC).                       |
| **S-4**      | **Logistics Officer** (p. 1-11)               | **Logistics & Resource Manager**: Manages hardware/compute resources, monitors GPU/CPU metrics, enforces token budgets, and configures LiteLLM model routing.                                                                                             |
| **S-6**      | **Communications System Officer** (p. 1-12)   | **Signal & Network Gateway**: Responsible for the design, installation, monitoring, and maintenance of communication channels (e.g., Google Meet, Telegram, inter-agent message brokers, API gateways, and HMI/dashboard telemetry). Direct report to XO. |
| **S-9**      | **Civil Affairs Officer** (Optional, p. 1-12) | **External Integration / User Liaison**: Manages external user interactions, public-facing summaries, and third-party API adapters (when augmented).                                                                                                      |

---

## Special Staff & Key Enablers (pp. 1-10 to 1-15)

- **Battalion Gunner** (p. 1-13): Infantry weapons officer and direct-fire SOME.
  In AI terms, acts as the **Domain SOME / Tool Safety Advisor**, advising S-3
  on precise tool parameters, script execution safety, and code sandbox
  boundaries prior to swarm deployment.
- **Sergeant Major (SgtMaj)** (p. 1-10): Senior Enlisted Advisor; enforces
  tactical SOP compliance, code quality standards, and field execution
  discipline across worker swarms under Section Chief TACON.
- **S-3 (Operations Officer)** (p. 1-11): Absorbs former G-5 long-range planning
  functions at the battalion level, acting as principal planner for task
  decomposition, execution orchestration, and Battle Damage Assessment (BDA).
- **Information Management Officer (IMO)** (p. 1-9): Managed under XO; custodian
  of the battalion's Information Management Plan (IMP) and inter-agent message
  schema formatting.

---

## Comparison: G-Staff vs. S-Staff Mapping for Suitcase AI

| Function               | G-Staff (Division/MEF - MCTP 3-30A)           | S-Staff (Battalion - MCRP 3-10A.1) | Suitcase AI Optimization Benefit                                                     |
| :--------------------- | :-------------------------------------------- | :--------------------------------- | :----------------------------------------------------------------------------------- |
| **Command**            | Chief of Staff (CoS) + SGS + Staff Sec        | **Executive Officer (XO)**         | Single executive routing agent instead of 3 administrative layers.                   |
| **Intel & RAG**        | G-2 (Intel) + G-5 (Plans) + Red Team          | **S-2 (Intel Officer)**            | Combines context scanning, RAG vector indexing, and environmental analysis into S-2. |
| **Operations**         | G-3 (Ops) + G-1 (Personnel) + G-7 (Commstrat) | **S-3 (Operations Officer)**       | S-3 directs execution and handles output summaries directly.                         |
| **Logistics & Budget** | G-4 (Logistics) + G-8 (Finance)               | **S-4 (Logistics Officer)**        | Combines token budget enforcement with local hardware/compute routing.               |
| **Comms & Telemetry**  | G-6 (Communications) + IMO                    | **S-6 (Comms Officer)**            | Merges transport layer monitoring, schema checks, and HMI status output.             |

---

## Strategic Application to Suitcase AI

Adopting the **MCRP 3-10A.1 Battalion S-Staff** layout allows **Suitcase AI** to
run a complete, highly structured military command architecture on a compact
local hardware footprint (such as the Asus Ascent GX10 + Minisforum MS-01
cluster):

1. **XO (Executive Agent) / Strategic Lead**: Runs on Tier 1 (Local 70B/49B
   reasoning models like `Llama-3.3-70B-Instruct` or
   `nvidia/Llama-3.3-Nemotron-Super-49B`, with optional Cloud Frontier API
   fallback).
2. **S-2 / S-3 (Intel & Ops)**: Run on Tier 1 / Tier 2
   (`mistralai/Mistral-Small-24B-Instruct-2501` or `Llama-3.1-8B-Instruct`) for
   planning, task orchestration, and tool execution.
3. **S-1 / S-4 / S-6 (Personnel, Logistics, Comms)**: Run on Tier 3
   (`llama3.2:3b`) for zero-cost background routing, telemetry, and budget
   auditing.
