# PRD.md:  "The Why"; Product requirements, benchmark metadata table, high-level requirements, etc.

Product Requirements Document

"The Why"; Product requirements, benchmark metadata table, high-level requirements, etc. Defines the benchmark goal, validation objective, test name, benchmark number, category, and high-level success criteria.

## Benchmark Matrix Document Metadata (via benchmark_specification.json)

This PRD.md section is populated from benchmark_specification.json, which is the structured source of benchmark-specific product requirements.

## Workload Number
111

## Workload Name
Linux perf PMU Analysis Harness

## Execution Summary (Run and Measure)
perf stat -x, -o <run>/perf_N.csv -e cycles,instructions,branches,branch-misses,cache-misses,context-switches,<stall event>,<event_list> -- <workload command>. Stall event: cycle_activity.stalls_total on Intel, stalled-cycles-backend on AMD (BENCHMARK_PMU_STALL_EVENT overrides). If perf cannot open its events, the command runs without perf, the four PMU metrics read na (<reason>), and context switches come from getrusage

## Main Goal
Analyze low-level CPU microarchitecture behavior (IPC, branch prediction, cache misses, pipeline stalls) under yaml stress-ng loads

## Validation Objective
Validates that each yaml stress-ng command completes under perf stat and that IPC, branch misprediction, cache-miss, pipeline-stall, and context-switch rates are parsed, or reported as na (<reason>) when the host does not expose a PMU counter

## Workload Category
System Profiling & Performance Analysis

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
