# reference/knowledge/ — Domain Knowledge

Immutable domain rules and hardware specifications that inform Agent decisions.

## Contents

- `npu-specs.json` — NPU hardware peak FLOPs, bandwidth, and HBM specs for Atlas A3/A2 devices

## Principles

- Files here are **read-only references** — never modify based on a single run
- New domain knowledge (operator limits, memory allocation strategies) is added here, not in skill-local references
- For code style conventions, see `reference/code-style/`
- For interface contracts, see `reference/io_specs/`