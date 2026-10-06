# SPEC.md: "The Human How"; Exact technical requirements, environment setup, implementation details, etc.

## Execution Description

Runs each yaml workload_commands entry under perf stat -x, and derives IPC, branch-mispredict percent, cache misses per 1,000 instructions, stall percent, and context switches/s. num_cpus substitutes into stress-ng. event_list adds perf events; unknown names are dropped. duration is seconds per command. Sweep dimensions: num_cpus, workload_commands, event_list, duration, output_format.

## Parameters

| Parameter | CLI Flag | Tested Values | Default | Description |
| --- | --- | --- | --- | --- |
| num_cpus | `--num-cpus` | smoke=1, baseline=2, extended=4 | 2 | From Parameter list; see Execution Description With Parameters. |
| workload_commands | `--workload-commands` | smoke=stress-ng --cpu {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --cache {num_cpus} --timeout {duration}s --metrics-brief, baseline=stress-ng --cpu {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --cache {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --branch {num_cpus} --timeout {duration}s --metrics-brief, extended=stress-ng --cpu {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --cache {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --branch {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --vm {num_cpus} --vm-bytes 64M --timeout {duration}s --metrics-brief | stress-ng --cpu {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --cache {num_cpus} --timeout {duration}s --metrics-brief,stress-ng --branch {num_cpus} --timeout {duration}s --metrics-brief | From Parameter list; see Execution Description With Parameters. |
| event_list | `--event-list` | smoke=cycles,cpu-cycles,instructions,branches,branch-misses,cache-references,cache-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-load-misses,iTLB-load-misses,stalled-cycles-frontend,stalled-cycles-backend,context-switches, baseline=cycles,cpu-cycles,instructions,branches,branch-misses,cache-references,cache-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-load-misses,iTLB-load-misses,stalled-cycles-frontend,stalled-cycles-backend,context-switches, extended=cycles,cpu-cycles,instructions,branches,branch-misses,cache-references,cache-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-load-misses,iTLB-load-misses,stalled-cycles-frontend,stalled-cycles-backend,context-switches | cycles,cpu-cycles,instructions,branches,branch-misses,cache-references,cache-misses,L1-dcache-loads,L1-dcache-load-misses,LLC-loads,LLC-load-misses,dTLB-load-misses,iTLB-load-misses,stalled-cycles-frontend,stalled-cycles-backend,context-switches | From Parameter list; see Execution Description With Parameters. |
| duration | `--duration` | smoke=2, baseline=70, extended=150 | 70 | From Parameter list; see Execution Description With Parameters. |
| output_format | `--output-format` | smoke=csv, baseline=csv, extended=csv | csv | From Parameter list; see Execution Description With Parameters. |

## Invocation

```bash
Run apt installed Linux perf (linux-tools-$(uname -r)) as perf stat -x, -e <events> -- <stress-ng command> for each yaml workload command. stress-ng is the workload generator
```

## Raw Output Format

CSV, one row per workload command (duplicated when only one command), plus perf_N.csv, perf_N.txt, and pmu_events.txt

sample_index,status,workload_command,duration_sec,elapsed_sec,instructions_per_cycle_ipc,branch_misprediction_rate,cache_misses_per_1000_instructions,pipeline_stall_pct,context_switches_during_workload_switches_s,cycles,instructions,branches,branch_misses,cache_misses,stall_cycles,pipeline_stall_event,pmu_counter_running_pct,context_switches,cpu_time_sec,cpu_utilization_percent,stress_ng_bogo_ops_s,pmu_source,error_message
0,ok,stress-ng,5,5.1,1.2,2.0,8.0,15,100,1e9,1.2e9,1e8,2e6,1e7,1.5e8,stalled-cycles-backend,100,500,4.8,96,1000,perf,

## Metrics

- **#1: Instructions per cycle** — stored as `instructions_per_cycle_ipc`.
- **#2: Branch misprediction rate, pct** — stored as `branch_misprediction_rate`.
- **#3: Cache misses per 1,000 instructions** — stored as `cache_misses_per_1000_instructions`.
- **#4: Pipeline stalls, pct** — stored as `pipeline_stall_pct`.
- **#5: Ctx switches/s** — stored as `context_switches_during_workload_switches_s`.

## Framework

Runs each yaml workload_commands entry under perf stat -x, and derives IPC, branch-mispredict percent, cache misses per 1,000 instructions, stall percent, and context switches/s. num_cpus substitutes into stress-ng. event_list adds perf events; unknown names are dropped.

## Installation and Execution Summary

perf stat -x, -o <run>/perf_N.csv -e cycles,instructions,branches,branch-misses,cache-misses,context-switches,<stall event>,<event_list> -- <workload command>. Stall event: cycle_activity.stalls_total on Intel, stalled-cycles-backend on AMD (BENCHMARK_PMU_STALL_EVENT overrides). If perf cannot open its events, the command runs without perf, the four PMU metrics read na (<reason>), and context switches come from getrusage

## Platform Portability

- **AMD (primary):** ```bash
Run apt installed Linux perf (linux-tools-$(uname -r)) as perf stat -x, -e <events> -- <stress-ng command> for each yaml workload command. stress-ng is the workload generator
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

CSV, one row per workload command (duplicated when only one command), plus perf_N.csv, perf_N.txt, and pmu_events.txt

sample_index,status,workload_command,duration_sec,elapsed_sec,instructions_per_cycle_ipc,branch_misprediction_rate,cache_misses_per_1000_instructions,pipeline_stall_pct,context_switches_during_workload_switches_s,cycles,instructions,branches,branch_misses,cache_misses,stall_cycles,pipeline_stall_event,pmu_counter_running_pct,context_switches,cpu_time_sec,cpu_utilization_percent,stress_ng_bogo_ops_s,pmu_source,error_message
0,ok,stress-ng,5,5.1,1.2,2.0,8.0,15,100,1e9,1.2e9,1e8,2e6,1e7,1.5e8,stalled-cycles-backend,100,500,4.8,96,1000,perf,

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
5. All required aggregate metrics are physically sensible (positive values). Runs each yaml workload_commands entry under perf stat -x, and derives IPC, branch-mispredict percent, cache misses per 1,000 instructions, stall percent, and context switches/s. num_cpus substitutes into stress-ng. event_list adds perf events; unknown names are dropped.
6. At least 2 sample rows exist for the latest `run_id` (sweep coverage).
7. No sample has `status = 'error'`.
8. Runs each yaml workload_commands entry under perf stat -x, and derives IPC, branch-mispredict percent, cache misses per 1,000 instructions, stall percent, and context switches/s. num_cpus substitutes into stress-ng. event_list adds perf events; unknown names are dropped.

### Baseline / Threshold configuration (`config/benchmark_config.yaml`)

Expected ranges and gates live in `config/benchmark_config.yaml` under `baselines:` or `thresholds:`. To update them, edit that file — never edit validation code directly.

Threshold key suffixes encode comparison direction when `thresholds:` is present: `_min` → observed value must be ≥ threshold. `_max` → observed value must be ≤ threshold. Informational `baselines:` ranges are not pass/fail gates.
