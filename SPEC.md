# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Samples ROCm GPU health once per second from amd-smi, rocminfo, and sysfs, and counts dmesg GPU fault lines since boot. monitor_duration_sec: seconds to keep sampling; temperature is the peak over the window (smoke 5 / baseline 200 / extended 580). expected_gpu_count: fewest HSA GPU agents that must stay visible; fewer fails the run (1). min_pcie_link_width: narrowest acceptable PCIe link, in lanes, on every AMD GPU; narrower fails the run (16) Sweep dimensions: monitor_duration_sec, expected_gpu_count, min_pcie_link_width.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| monitor_duration_sec | `--monitor-duration-sec` | smoke=5, baseline=200, extended=580 | 200 | From Parameter list; see Execution Description With Parameters. |
| expected_gpu_count | `--expected-gpu-count` | smoke=1, baseline=1, extended=1 | 1 | From Parameter list; see Execution Description With Parameters. |
| min_pcie_link_width | `--min-pcie-link-width` | smoke=16, baseline=16, extended=16 | 16 | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Run dmesg, amd-smi metric, rocminfo, and sysfs PCIe link files
```

## Raw Output Format

CSV with one row per one-second sample, plus dmesg.txt

sample_index,status,dmesg_gpu_fault_count,gpu_temperature_c,hsa_agents_visible_count,pcie_link_width_lanes,pcie_link_speed_gt_s,queues_available_count,window_new_dmesg_fault_count,sample_elapsed_s,sample_gpu_temperature_c,sample_hsa_agents_visible_count,sample_pcie_link_width_lanes,sample_pcie_link_speed_gt_s,error_message
0,ok,0,41,1,16,32,1,0,0.0,40,1,16,32,

## Metrics

- **#1: dmesg GPU fault count** — stored as `dmesg_gpu_fault_count`.
- **#2: GPU temperature** — stored as `gpu_temperature_c`.
- **#3: PCIe link width** — stored as `pcie_link_width_lanes`.
- **#4: PCIe link speed** — stored as `pcie_link_speed_gt_s`.
- **#5: HSA agents visible** — stored as `hsa_agents_visible_count`.

## Framework

Samples ROCm GPU health once per second from amd-smi, rocminfo, and sysfs, and counts dmesg GPU fault lines since boot. monitor_duration_sec: seconds to keep sampling; temperature is the peak over the window (smoke 5 / baseline 200 / extended 580). expected_gpu_count: fewest HSA GPU agents that must stay visible; fewer fails the run (1).

## Installation and Execution Summary

Run amd-smi metric, rocminfo, and sysfs PCIe link files once per second for monitor_duration_sec, and dmesg at the start and end, then report peak junction temperature, the narrowest PCIe link, the fewest HSA GPU agents seen, and fault-keyword counts, to measure ROCm GPU health. DCGM is not used

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

CSV with one row per one-second sample, plus dmesg.txt

sample_index,status,dmesg_gpu_fault_count,gpu_temperature_c,hsa_agents_visible_count,pcie_link_width_lanes,pcie_link_speed_gt_s,queues_available_count,window_new_dmesg_fault_count,sample_elapsed_s,sample_gpu_temperature_c,sample_hsa_agents_visible_count,sample_pcie_link_width_lanes,sample_pcie_link_speed_gt_s,error_message
0,ok,0,41,1,16,32,1,0,0.0,40,1,16,32,

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
5. All required aggregate metrics are physically sensible (positive values). Samples ROCm GPU health once per second from amd-smi, rocminfo, and sysfs, and counts dmesg GPU fault lines since boot. monitor_duration_sec: seconds to keep sampling; temperature is the peak over the window (smoke 5 / baseline 200 / extended 580). expected_gpu_count: fewest HSA GPU agents that must stay visible; fewer fails the run (1).
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Samples ROCm GPU health once per second from amd-smi, rocminfo, and sysfs, and counts dmesg GPU fault lines since boot. monitor_duration_sec: seconds to keep sampling; temperature is the peak over the window (smoke 5 / baseline 200 / extended 580). expected_gpu_count: fewest HSA GPU agents that must stay visible; fewer fails the run (1).

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
