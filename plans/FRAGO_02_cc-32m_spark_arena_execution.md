# Fragmentary Order (FRAGO) 02 to OPORD cc-32m: Spark Arena Benchmark Execution (llama-benchy)

```
Document ID:     FRAGO-COLT-2026-09-24-02-SPARK-ARENA
Base Order:      OPLAN-COLT-2026-09-01-LOCAL-MODELS (Epic cc-32m)
Classification:  CAMP COLT SOVEREIGN APPLIANCE OPERATIONAL DIRECTIVE
Author:          Chief of Staff (cos), Camp Colt
Coordinating:    S-3 Operations (s3-operations), Platform Chief (colt-sysadmin), S-6 Signal (s6-signal)
Target Enclave:  ASUS Ascent GX10 (NVIDIA Grace Blackwell GB10 @ 10.82.0.3 / colt-gpu-01)
Date:            2026-09-24
Authority:       Commander's Directive (MCTP 3-30A / MCWP 5-10)
```

---

## 1. Situation & Purpose

1. **Strategic Intent**: Execute standardized hardware inference benchmarking for the **Spark Arena Leaderboard** ([spark-arena.com](https://spark-arena.com/)) on our sovereign ASUS Ascent GX10 (NVIDIA Grace Blackwell GB10) accelerator (`colt-gpu-01`).
2. **Clarification of Intent**: Previous testing under FRAGO 01 addressed internal agent behavioral suites. This order refocuses operational execution specifically on the official community standard: **`llama-benchy`** raw inference metrics (TTFT, TPS, and concurrency latency) adhering to the Spark Arena recipe specification.
3. **Execution Vehicle**: The in-cluster Kubernetes benchmark job located at [`rigs/suitcase-ai/provision/k8s/apps/inference/benchmarks/llama-benchy-job.yaml`](../../suitcase-ai/provision/k8s/apps/inference/benchmarks/llama-benchy-job.yaml).

---

## 2. Mandatory Command Review & Publication Gate (STRICT HOLD)

> [!IMPORTANT]
> **LOCAL REVIEW GATE ONLY — ZERO PREMATURE EXTERNAL EGRESS**
> 1. All benchmark receipts (`results.csv` and compiled `recipe.yaml`) MUST be harvested and compiled into a local review dossier.
> 2. The dossier must be submitted to the Commander via the Chief of Staff for **personal review and sign-off locally**.
> 3. **NO EXTERNAL PUBLICATION** to `spark-arena.com`, public registries, or external communications channels is authorized until the Commander personally approves the results post-review.

---

## 3. Tasks to Staff Sections

### A. S-3 Operations (`s3-operations`) — Tactical Lead
1. Coordinate with Platform Chief to verify `colt-gpu-01` inference pod readiness (`nemotron-49b` in `nim-system`).
2. Deploy and monitor the in-cluster `llama-benchy` Job:
   ```bash
   kubectl apply -f rigs/suitcase-ai/provision/k8s/apps/inference/benchmarks/llama-benchy-job.yaml
   ```
3. Harvest benchmark output:
   - Extract `/tmp/results.csv` from the benchmark pod.
   - Pair with authoritative recipe ([`rigs/suitcase-ai/recipes/nemotron-49b-fp8.yaml`](../../suitcase-ai/recipes/nemotron-49b-fp8.yaml)).
4. Compile the **Spark Arena Local Review Dossier** and submit to Command via `gc mail send cos`.

### B. Platform Chief (`colt-sysadmin`) — Platform SRE
1. Pre-flight hardware audit on `colt-gpu-01`:
   - Confirm GPU temperature $< 50^\circ\text{C}$ idle.
   - Confirm vLLM/NIM inference engine health on `:8002/v1/health/ready`.
   - Verify UMA memory clamping constraint ($\le 0.75$ / 96 GB max) to safeguard 32 GB system buffer.
2. Monitor host thermal, power, and memory behavior during benchmark execution.

### C. S-6 Signal (`s6-signal`) — Quality & Stan/Eval
1. Audit benchmark logs for latency jitter and anomalous connection drops.
2. Validate zero secret leakage in exported CSV and recipe metadata.

---

## 4. Coordinating Instructions & Artifacts

- **Primary Output Artifacts**:
  1. `results.csv`: Standardized Spark Arena CSV (`model`, `latency_sec`, `prompt_tokens`, `completion_tokens`, `tps`).
  2. `recipe.yaml`: Complete declarative configuration matching the physical test.
  3. `local_review_dossier.md`: Staff evaluation comparing local GB10 numbers against published spark-arena.com standings.
- **Reporting**: S-3 submits Completed Staff Work upon job completion.

Regards,  
Chief of Staff, Camp Colt
