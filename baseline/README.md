# baseline/ — Performance Acceptance Criteria

Performance baseline data for each model on different NPU hardware. Used as the
reference point for optimization validation (Phase 1 of the evidence loop).

## Contents

- `xllm-benchmark.md` — Qwen35-27B baseline performance and accuracy data

## Usage

- Before claiming an optimization gain, compare against the relevant baseline entry
- Baseline data must be collected under warmup conditions with documented workload
- New baseline entries are added when a new model/NPU combo is formally validated