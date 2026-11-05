# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Takes one ROCm health snapshot from dmesg, amd-smi, rocminfo, and sysfs. Every run records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count Sweep dimensions: no swept parameters.

## Parameters

No workbook parameters are populated for this workload.

## Invocation

```bash
Run dmesg, amd-smi metric, rocminfo, and sysfs PCIe link files
```

## Raw Output Format

CSV aggregate plus dmesg.txt

sample_index,status,dmesg_gpu_fault_count,gpu_temperature_c,hsa_agents_visible_count,pcie_link_width_lanes,pcie_link_speed_gt_s,queues_available_count,error_message
0,ok,0,40,1,16,32,1,

## Metrics

- **#1: dmesg GPU fault count** — stored as `dmesg_gpu_fault_count`.
- **#2: GPU temperature** — stored as `gpu_temperature_c`.
- **#3: PCIe link width** — stored as `pcie_link_width_lanes`.
- **#4: PCIe link speed** — stored as `pcie_link_speed_gt_s`.
- **#5: HSA agents visible** — stored as `hsa_agents_visible_count`.

## Framework

Takes one ROCm health snapshot from dmesg, amd-smi, rocminfo, and sysfs. Every run records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count

## Installation and Execution Summary

Run dmesg, amd-smi metric (temperature, clocks, power), rocminfo, and sysfs PCIe link files, then parse junction temp, PCIe width/speed, HSA agents, and fault-keyword counts, to measure ROCm GPU health. DCGM is not used

## Platform Portability

- **AMD (primary):** ```bash
Run dmesg, amd-smi metric, rocminfo, and sysfs PCIe link files
```
- **NVIDIA:** Primary target is AMD ROCm. NVIDIA notes in this section are reference only and are not the execution path.

## Model Context Protocols

- **Active:** None

## Execution-Loop Validation Contract

EXECUTION CHAIN: `run_benchmark.sh` ➔ raw output ➔ `scripts/parse_results.py` ➔ `results/benchmark.db` ➔ `scripts/validate_results.py`

This benchmark uses a lightweight, SQLite-integrated execution loop for result validation. All validation is performed by `scripts/validate_results.py`.

### Validation script usage

```bash
export BENCHMARK_PYTHON=/usr/bin/python3.13  # optional; select the installed interpreter

# After a live run:
".venv/bin/python" scripts/validate_results.py --db results/benchmark.db

# CI / no-GPU path (seeds fixture and validates it):
".venv/bin/python" scripts/validate_results.py --seed-fixture --quiet

# Override DB path via environment variable:
BENCHMARK_DB=tests/fixtures/benchmark.db \
  ".venv/bin/python" scripts/validate_results.py
```

### Run artifact contract

CSV aggregate plus dmesg.txt

sample_index,status,dmesg_gpu_fault_count,gpu_temperature_c,hsa_agents_visible_count,pcie_link_width_lanes,pcie_link_speed_gt_s,queues_available_count,error_message
0,ok,0,40,1,16,32,1,

```bash
bash run_benchmark.sh --help
bash run_benchmark.sh --smoke
bash run_benchmark.sh --baseline
bash run_benchmark.sh --extended
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Takes one ROCm health snapshot from dmesg, amd-smi, rocminfo, and sysfs. Every run records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Takes one ROCm health snapshot from dmesg, amd-smi, rocminfo, and sysfs. Every run records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
