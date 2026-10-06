# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Takes one ROCm health snapshot from dmesg, rocm-smi, amd-smi, and rocminfo. Yaml pcie_check, clock_check, temperature_check, power_check, ecc_check, ras_check, firmware_version_check, hsa_agent_check, queue_check, and dmesg_check are documented; the collector always records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count. output_format: csv Sweep dimensions: pcie_check, clock_check, temperature_check, power_check, ecc_check, ras_check, firmware_version_check, hsa_agent_check.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| pcie_check | `--pcie-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| clock_check | `--clock-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| temperature_check | `--temperature-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| power_check | `--power-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| ecc_check | `--ecc-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| ras_check | `--ras-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| firmware_version_check | `--firmware-version-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| hsa_agent_check | `--hsa-agent-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| queue_check | `--queue-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| dmesg_check | `--dmesg-check` | smoke=true, baseline=true, extended=true | true | From Parameter list; see Execution Description With Parameters. |
| output_format | `--output-format` | smoke=csv, baseline=csv, extended=csv | csv | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Run dmesg, rocm-smi --showtemp --showbus --showclocks --showpower, amd-smi metric, and rocminfo
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

Takes one ROCm health snapshot from dmesg, rocm-smi, amd-smi, and rocminfo. Yaml pcie_check, clock_check, temperature_check, power_check, ecc_check, ras_check, firmware_version_check, hsa_agent_check, queue_check, and dmesg_check are documented; the collector always records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count. output_format: csv

## Installation and Execution Summary

Run dmesg, rocm-smi temperature/bus/clock/power queries, amd-smi metric, and rocminfo, then parse junction temp, PCIe width/speed, HSA agents, and fault-keyword counts, to measure ROCm GPU health. DCGM is not used

## Platform Portability

- **AMD (primary):** ```bash
Run dmesg, rocm-smi --showtemp --showbus --showclocks --showpower, amd-smi metric, and rocminfo
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
bash run_benchmark.sh --profile smoke --validate
bash run_benchmark.sh --profile baseline --validate
bash run_benchmark.sh --profile extended --validate
```
`run_benchmark.sh --help` prints usage and exits. The harness calls `scripts/ensure_setup.sh` when `.setup_state` is absent.

### Required integrity checks (built into `validate_results.py`)

1. Latest run exists and `runs.status = 'ok'`.
2. `run.error_message` is NULL.
3. `started_at` and `finished_at` are valid ISO-8601 UTC strings.
4. All required aggregate metrics in `runs` are non-NULL and finite.
5. All required aggregate metrics are physically sensible (positive values). Takes one ROCm health snapshot from dmesg, rocm-smi, amd-smi, and rocminfo. Yaml pcie_check, clock_check, temperature_check, power_check, ecc_check, ras_check, firmware_version_check, hsa_agent_check, queue_check, and dmesg_check are documented; the collector always records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count. output_format: csv
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Takes one ROCm health snapshot from dmesg, rocm-smi, amd-smi, and rocminfo. Yaml pcie_check, clock_check, temperature_check, power_check, ecc_check, ras_check, firmware_version_check, hsa_agent_check, queue_check, and dmesg_check are documented; the collector always records dmesg faults, junction temp, HSA agent count, PCIe width/speed, and queue count. output_format: csv

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
