# SOP: S-3 Tactical Operations Watch Cadence & Stall Escalation Protocol

**Doctrinal Authority:** Commander's Planning Guidance (CPG) / MCTP 3-30A
**Custodian:** S-3 Operations (`s3-operations`)
**Scope:** Camp Colt Virtual Combat Operations Center (COC) Watchstander Protocol
**Effective Date:** 2026-09-29

---

## 1. Purpose

This Standard Operating Procedure (SOP) governs the dynamic sampling cadence, telemetry auditing, and stall-mitigation escalations executed by the S-3 Tactical Watch Officer during active autonomous task execution across Camp Colt rigs (specifically `rigs/gascity` and inference engine `colt-sparkrun`).

---

## 2. Dynamic Cadence State Machine

```
   [ 15-Minute Watch ]
           │
           │ (Stalled work or 0 TPS in last 5m)
           ▼
   [ 5-Minute Emergency Cycle ] ◄────────┐
           │                             │
           │ (1st Productive Cycle: P=1) │ (Stall/0 TPS
           ▼                             │  detected)
   [ 5-Minute Verification Cycle ]       │
           │                             │
           │ (2nd Productive Cycle: P=2) │
           ▼                             │
   [ 30-Minute Steady-State Watch ] ─────┘
```

### State Definitions

1. **15-Minute Cycle (`*/15 * * * *`) — Initial / Transitional Active Watch**:

   - Standard surveillance cadence during initial worker dispatch and complex multi-step execution.
   - Evaluates silicon telemetry (`power.draw`, `vllm:generation_tokens_total`, `vllm:num_requests_running`) and task progression.
   - **Trigger Condition**: If the check detects **no work in the last 5 minutes** (0 TPS generation token delta) or **otherwise stalled work** (reverted to OPEN without active lease, crashing turn loops, unserved nudges):
     - Immediately cancel the 15-minute cron.
     - Reset productive cycle counter ($P = 0$).
     - Transition immediately to the **5-Minute Emergency Cycle**.

2. **5-Minute Emergency Cycle (`*/5 * * * *`) — Tactical Unblock & Triage**:

   - Fast surveillance and active unblocking loop.
   - Watchstander peeks on worker session logs (`opencode.log`), supervisor state (`~/.gc/supervisor.log`), and bead metadata.
   - Coordinates with `mayor` (for session recycles, rig lifecycle, or municipal hygiene fixes) and alerts `cos`.
   - **Productivity Metric**: A 5-minute cycle is classified as **Productive** if and only if:
     1. Positive generation token throughput is confirmed ($\Delta \text{gen\_tokens} > 0$).
     2. Worker session is actively executing tool calls or streaming model completions.
     3. Target task bead has an active, valid lease.
   - **Transition Gate**:
     - Cycle 1 Productive: Increment $P = 1$. Continue 5-minute watch for verification.
     - Cycle 2 Productive: Increment $P = 2$. **Transition immediately to 30-Minute Steady-State Watch**.
     - Any cycle Unproductive / Stalled: Reset $P = 0$. Remain in 5-minute cycle; escalate to Mayor/CoS.

3. **30-Minute Steady-State Watch (`*/30 * * * *`) — Low-Overhead Sustained Watch**:
   - Long-horizon surveillance cadence entered after verified stability.
   - Minimizes agent turn overhead and token burn on watchstander operations while large compilation, testing, or multi-turn tasks run.
   - **Trigger Condition**: If any 30-minute check detects **no work in the last 5 minutes** or **stalled execution**:
     - Immediately cancel the 30-minute cron.
     - Reset productive cycle counter ($P = 0$).
     - Transition immediately to the **5-Minute Emergency Cycle**.

---

## 3. Telemetry Verification Standards (Receipts Over Assertions)

The watchstander verifies true work execution via linear silicon receipts:

1. **Token Generation Throughput**:

   ```bash
   curl -s http://10.82.0.3:8000/metrics | grep "vllm:generation_tokens_total"
   ```

2. **Active Batch Execution Gauge**:

   ```bash
   curl -s http://10.82.0.3:8000/metrics | grep "vllm:num_requests_running"
   ```

3. **Linear Board Power Draw**:

   ```bash
   ssh colt-sparkrun "nvidia-smi --query-gpu=power.draw,utilization.gpu --format=csv,noheader"
   ```

   - Idle Baseline: `~10.5 W`
   - Active Decode: `35 W – 42 W`
   - Heavy Batch Load: `65 W – 100+ W`

---

## 4. Communication & Sign-Off Protocol

- Informational updates and hourly telemetry rollups transmitted without `--notify`.
- Tactical blocker alerts and urgent unblock requests to `mayor` transmitted with `gc mail ... --notify`.
- Formal dispatches conclude with junior staff sign-off: `Respectfully, s3-operations`.
