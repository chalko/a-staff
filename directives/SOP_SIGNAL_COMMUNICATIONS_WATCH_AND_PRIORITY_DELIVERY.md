# Standard Operating Procedure: Signal Communications Watch & Priority Delivery Protocol

**Status:** Approved
**Date:** 2026-10-09
**Standard:** Completed Staff Work

---

## 1. Purpose & Doctrinal Intent

In accordance with the Commander's Planning Guidance (CPG) and MCTP 3-30A (_Command and Staff Action_), Camp Colt requires reliable, deterministic, non-blocking message delivery and communications oversight across all active command billets and tactical sessions.

Autonomous agents operating complex workloads can fail to check inbound communications promptly, resulting in dormant orders, blocked convoys, and delayed escalations. Prior interim practice relied on manual or multi-agent polling, expending unnecessary LLM tokens and creating noise.

This SOP codifies the **Signal Communications Watchdog Protocol** and establishes the municipal recurring Gas City order [`orders/signal_mail_watch.toml`](../../orders/signal_mail_watch.toml) executing [`commands/signal_mail_watch.sh`](../../commands/signal_mail_watch.sh). It enforces a deterministic, zero-LLM, priority-tiered nudge delivery engine with automated high-priority failure escalation.

---

## 2. Billet Boundaries & Division of Labor

To maintain clear operational separation of concerns:

- **S-6 Signal & Cyber Section (`s6-signal`)**: Sole custodian and doctrinal owner of message transport, signal delivery, inter-staff communications watch, priority threshold evaluation, and automated mail nudging.
- **S-3 Operations (`s3-operations`)**: Focuses exclusively on tactical bead execution, worker stagnation detection, task tree progression, and convoy tracking per `SOP_OPERATIONS_WATCH_AND_STAGNATION_DETECTION.md`. S-3 does not patrol peer staff mailboxes.
- **Mayor (`mayor`)**: Municipal lead holding sole authority for registering, updating, and reloading Gas City orders and lifecycle processes (`city.toml`, `orders/`).
- **S-1 Personnel (`s1-personnel`)**: Doctrinal custodian for administrative formatting, review, and publication into `rigs/a-staff`.

---

## 3. Architecture & Invariants

The watchdog engine operates under four immutable architectural invariants:

1. **Zero LLM / Zero Token Footprint**:
   - Implemented as a single, self-contained shell utility ([`commands/signal_mail_watch.sh`](../../commands/signal_mail_watch.sh)) utilizing native POSIX tools (`bash`, `jq`, `date`).
   - Requires no companion Python script and consumes zero inference tokens or LLM agent turns.
2. **Graduated Priority Thresholds**:
   - Messages are evaluated against their native bead priority or subject line prefix tags.
   - Nudges are delayed proportionally to priority to prevent low-priority alert fatigue.
3. **State Ledger & Idempotency**:
   - Tracks delivered nudges in `.gc/runtime/watch/signal_mail_watch_state.json`.
   - Enforces a 15-minute cooldown (`900s`) between successive nudges for the same message bead.
4. **Automated P1 Failure Escalation**:
   - Any script failure, bad argument, or non-zero exit triggers an error trap.
   - Automatically formats an alert with priority P1 and dispatches `gc mail send s6-signal --notify` to immediately wake the watchstander.

```mermaid
flowchart TD
    A[Tick: 5-Minute Cron Order] --> B[commands/signal_mail_watch.sh]
    B --> C[Query gc session list --json]
    B --> D[Query gc bd list --type message --status open --json]
    C --> E[Map Live Sessions]
    D --> F[Evaluate Age vs Priority Threshold]
    F -- Age < Threshold --> G[Ignore: Not Breached]
    F -- Age >= Threshold --> H{Check Cooldown Ledger}
    H -- Nudged < 15m Ago --> I[Skip: Cooldown Active]
    H -- First Breach or >15m --> J{Recipient Session Active?}
    J -- No Active Session --> K[Log Notice & Safely Skip]
    J -- Active Session Found --> L[gc session nudge TARGET_SID]
    L --> M[Update signal_mail_watch_state.json]
    B -- Any Execution Error / Trap --> N[Phase Escalation: [P1] ALERT]
    N --> O[gc mail send s6-signal --notify]
    O --> P[gc bd update BEAD_ID -p P1]
```

---

## 4. Graduated Priority Threshold Matrix

The watchdog inspects all open message beads (`type == "message"` and `status == "open"`). Priority is extracted in order of precedence:

1. Native bead priority field (`priority` = 0, 1, 2, 3, 4, or `"P0"`-`"P4"`).
2. Explicit bracketed tags in title: `[P0]`, `[P1]`, `[P2]`, `[P3]`, `[P4]`.
3. Keyword heuristics: `URGENT` / `CRITICAL` (P0), `PRIORITY` / `ALERT` (P1), `FYI` (P4).
4. Default fallback: **P2 (Normal)**.

| Priority Level | Description                  | Breach Threshold            | Action on Breach                               |
| :------------- | :--------------------------- | :-------------------------- | :--------------------------------------------- |
| **P0**         | **Immediate / Emergency**    | **> 0 Minutes** (Immediate) | Immediate `gc session nudge` to active session |
| **P1**         | **High Priority / Alerts**   | **> 15 Minutes** (`900s`)   | Nudge delivered if unread after 15m            |
| **P2**         | **Normal Operational Mail**  | **> 1 Hour** (`3600s`)      | Nudge delivered if unread after 1h             |
| **P3**         | **Low Priority / Backlog**   | **> 8 Hours** (`28800s`)    | Nudge delivered if unread after 8h             |
| **P4**         | **Deferred / FYI / Archive** | **> 24 Hours** (`86400s`)   | Nudge delivered if unread after 24h            |

### Nudge Delivery Formatting

When a breach occurs, the watchdog resolves the recipient to an active session ID (e.g. `cc-bba` for `cos`, `cc-c93` for `mayor`) and transmits:

```text
gc session nudge <session-id> "Unread [P{x}] mail {id} from {from} pending {age}. Process via gc mail read {id}."
```

---

## 5. State Ledger & Cooldown Protocol

To prevent terminal input spamming of active agent sessions:

- The state ledger is maintained at `.gc/runtime/watch/signal_mail_watch_state.json`.
- When a nudge is successfully delivered, the timestamp is recorded:

  ```json
  {
    "nudges": {
      "cc-wisp-0yzzy": 1791515900
    },
    "last_run": "2026-10-09T03:18:20Z"
  }
  ```

- Subsequent runs within 15 minutes of the last nudge skip delivery.
- Entries older than 48 hours are automatically pruned during each cycle to prevent unbounded state growth.

---

## 6. Automated Failure Escalation Trap (Command Directives)

Per **Directive `cc-wisp-0yzzy`** and **FRAGO `cc-wisp-i96l0`**, any failure encountered by the watchdog script must wake the S-6 Signal Officer:

1. **Trap Definition**:

   ```bash
   trap 'handle_failure $LINENO $?' ERR
   ```

2. **Subject Formatting**:

   ```text
   [P1] ALERT: signal_mail_watch.sh Execution Failure (Line $LINENO, Code $EXIT_CODE)
   ```

3. **Bead Priority Stamping**:
   The alert message bead generated via `gc mail send` must be explicitly updated to priority `P1`:

   ```bash
   gc bd update "${bead_id}" -p P1
   ```

4. **Immediate Wakeup**:
   Dispatched with `--notify` to guarantee immediate S-6 session interrupt.

---

## 7. Municipal Deployment & Order Configuration

The recurring order is defined in [`orders/signal_mail_watch.toml`](../../orders/signal_mail_watch.toml):

```toml
# orders/signal_mail_watch.toml
[order]
description = "Signal Mail Watchdog: 5-minute automated mail scan, graduated priority nudge, and failure escalation"
exec = "commands/signal_mail_watch.sh"
trigger = "cron"
schedule = "*/5 * * * *"
```

### Activation Procedure

1. S-6 verifies script execution and passes all unit tests in `tests/test_signal_mail_watch.py`.
2. S-6 tasks the Mayor via `gc mail send mayor --notify` to validate and register the order.
3. The Mayor registers the order and reloads the municipal runtime (`gc reload`).

---

## 8. Verification & Empirical Receipts

The watchdog script is verified using the automated test suite [`tests/test_signal_mail_watch.py`](../../tests/test_signal_mail_watch.py), asserting:

- Single-script integrity (no duplicate `.py` file).
- Graduated P0-P4 threshold gating.
- Idempotency and 15-minute cooldown suppression.
- Offline recipient safety.
- P1 failure classification and error trap execution.

```text
Ran 5 tests in 0.501s
OK
```
