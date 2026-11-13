# Results: 102 ROCm System Validation (clocks, temp, etc.)

**Main goal:** Analyze GPU ROCm runtime environment health  
**Tools:** Run dmesg, amd-smi metric, rocminfo, and sysfs PCIe link files

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_139 · recorded 2026-11-13). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× AMD Instinct MI300X VF (196288 MB) · INTEL(R) XEON(R) PLATINUM 8568Y+ · Ubuntu 24.04.5 LTS · kernel 6.8.0-147-generic · driver 7.1.9.31600000
- **Run:** 2026-11-13T13:43:55Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 00:11 · benchmark 00:06 · whole run 00:28
- **Source:** run 102_20261113_134355
- **Command:** `bash run_benchmark.sh --smoke --monitor-duration-sec 5 --expected-gpu-count 1 --min-pcie-link-width 16`

| # | Metric | Result |
|---|---|---|
| 1 | dmesg GPU fault count (dmesg_gpu_fault_count) | 0.0 |
| 2 | GPU temperature (gpu_temperature_c) | 38.0 |
| 3 | PCIe link width (pcie_link_width_lanes) | 16.0 |
| 4 | PCIe link speed (pcie_link_speed_gt_s) | 32.0 |
| 5 | HSA agents visible (hsa_agents_visible_count) | 1.0 |
<!-- /profile:smoke -->
