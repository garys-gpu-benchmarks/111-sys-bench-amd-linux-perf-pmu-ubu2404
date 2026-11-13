# Results: 111 Linux perf PMU Analysis Harness

**Main goal:** Analyze CPU IPC, branch, cache, and stall behavior under load  
**Tools:** Run perf stat -x, -e <events> -- <stress-ng command> for each yaml workload command

Reference results measured by the maintainer on the system shown in each section (generator TEMPLATE_00_138 · recorded 2026-11-12). Your own runs write `results/RESULTS_ACTUAL.md` in the same format.

> Smoke runs use deliberately small sizes. They check that the workload installs and runs end to end; their numbers are not performance results. Baseline and extended are.

<!-- profile:smoke -->
## smoke

- **System:** 1× AMD Instinct MI300X VF (196288 MB) · INTEL(R) XEON(R) PLATINUM 8568Y+ · Ubuntu 24.04.5 LTS · kernel 6.8.0-147-generic · driver 7.1.9.31600000
- **Run:** 2026-11-12T22:55:24Z · status **PASS** (exit 0)
- **Time (mm:ss):** setup 00:07 · benchmark 00:05 · whole run 00:23
- **Source:** run 111_20261112_225524
- **Command:** `bash run_benchmark.sh --smoke --num-cpus 1 --workload-commands "stress-ng --cpu {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --cache {num_cpus} --timeout {duration}s --metrics-brief" --event-list cycles,cpu-cycles,instructions,branches,branch-misses,cache-references,cache-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-load-misses,iTLB-load-misses,stalled-cycles-frontend,stalled-cycles-backend,context-switches --duration 2`

| # | Metric | Result |
|---|---|---|
| 1 | Instructions per cycle, all commands combined (instructions_per_cycle_ipc) | 2.7787249584731484 |
| 2 | Branch misprediction rate, all commands combined, pct (branch_misprediction_rate) | 0.4142831090340228 |
| 3 | Cache misses per 1,000 instructions, all commands combined (cache_misses_per_1000_instructions) | 0.01602460549211702 |
| 4 | Pipeline stalls, all commands combined, pct (pipeline_stall_pct) | 24.736880230819335 |
| 5 | Context switches per second, all commands combined (context_switches_during_workload_switches_s) | 62.1581148904841 |
<!-- /profile:smoke -->
