# PRD.md:  "The Why"; Product requirements, benchmark metadata table, high-level requirements, etc.

Product Requirements Document

"The Why"; Product requirements, benchmark metadata table, high-level requirements, etc. Defines the benchmark goal, validation objective, test name, benchmark number, category, and high-level success criteria.

## Benchmark Matrix Document Metadata (via benchmark_specification.json)

This PRD.md section is populated from benchmark_specification.json, which is the structured source of benchmark-specific product requirements.

## Workload Number
102

## Workload Name
ROCm System Validation (clocks, temp, etc.)

## Execution Summary (Run and Measure)
Run dmesg, rocm-smi temperature/bus/clock/power queries, amd-smi metric, and rocminfo, then parse junction temp, PCIe width/speed, HSA agents, and fault-keyword counts, to measure ROCm GPU health. DCGM is not used

## Main Goal
Analyze GPU ROCm runtime environment health

## Validation Objective
Validates a single telemetry snapshot. ECC/RAS/clock yaml flags are not implemented as separate probes

## Workload Category
System Validation & Reliability

## Validation Requirement

The benchmark must include an automated SQLite-integrated validation layer that verifies persisted results from `results/benchmark.db`. Validation must confirm:

1. The benchmark run completed successfully with no tool errors.
2. Required samples and aggregate metrics were persisted for every swept shape.
3. Metrics are finite and physically sensible (positive, within plausible bounds).
4. Measured values satisfy configured thresholds when the workload defines pass/fail gates.
5. The benchmark fails validation when required data is missing, invalid, or outside bounds.

## Non-Functional Requirements

| Requirement | Target |
|---|---|
| Automation | Runs to completion without manual intervention after `bash run_benchmark.sh` |
| Idempotency | Re-running `run_benchmark.sh` appends a new run; never corrupts existing rows |
| Persistence | All metrics survive script exit; `results/benchmark.db` is the durable record |
