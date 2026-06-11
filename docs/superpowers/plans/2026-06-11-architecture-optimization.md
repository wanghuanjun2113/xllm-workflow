# Architecture Optimization Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Restructure xllm-workflow to align with six core architectural principles (deterministic execution, audience isolation, knowledge flywheel, SSOT, skill encapsulation, simplicity).

**Architecture:** Incremental restructuring — create target directories, move/split files, merge documents, update all path references, then delete old locations. Each task produces a verifiable structural change. Final verification via existing tests.

**Tech Stack:** Git (mv + rm), Python (pytest for verification), Markdown (document merging)

---

## File Structure

### New Files Created

| File | Responsibility |
|------|---------------|
| `Agent.md` | Core Agent system prompt (merged from AGENTS.md + INSTRUCTIONS.md + CLAUDE.md) |
| `config.json` | Unified configuration (active, full_test, static with npu_specs) |
| `reference/knowledge/README.md` | Knowledge directory purpose description |
| `reference/code-style/custom-code-style.md` | Code style conventions (moved from references/) |
| `reference/io_specs/accuracy-artifact-schema.md` | Accuracy artifact contract (moved) |
| `reference/io_specs/perf-artifact-schema.md` | Perf artifact contract (moved) |
| `reference/io_specs/profiling-artifact-schema.md` | Profiling artifact contract (moved) |
| `reference/io_specs/run-manifest-template.md` | Run manifest template (moved) |
| `reference/pr_history/SKILL.md` | PR history query workflow (moved) |
| `reference/pr_history/card-schema.md` | Dossier card schema (moved) |
| `reference/pr_history/deepseek-v3.md` | DeepSeek-V3 dossier (moved) |
| `reference/pr_history/glm-5.md` | GLM-5 dossier (moved) |
| `reference/pr_history/qwen35-mtp.md` | Qwen3.5 MTP dossier (moved) |
| `baseline/README.md` | Baseline directory purpose description |
| `baseline/xllm-benchmark.md` | Qwen35-27B benchmark data (moved) |
| `scripts/query.py` | PR history query CLI (moved) |
| `scripts/collect_evalscope_results.py` | Evalscope result collector (moved) |
| `scripts/compare_npu_benchmark.py` | Cross-framework comparison (moved) |
| `scripts/validate_framework_cli.py` | Framework CLI validator (moved) |
| `scripts/README.md` | Scripts directory purpose description |

### Files Modified

| File | Change |
|------|--------|
| `Claude.md` | Reduced to thin redirect to Agent.md |
| `.gitignore` | Add `code/` and `runs/` |
| `humanize/README.md` | Add usage description and write-back rules |
| `skills/xllm-npu-accuracy-debug/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-benchmark/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-capacity-planner/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-code-review/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-compute-simulation/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-eval-runner/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-incident-triage/SKILL.md` | Update cross-skill reference |
| `skills/xllm-npu-op-migration/SKILL.md` | Update `kernel-pilot` references |
| `skills/xllm-npu-pipeline-analysis/SKILL.md` | Update `../../references/` + cross-skill paths |
| `skills/xllm-npu-profiler/SKILL.md` | Update `../../references/` paths |
| `skills/xllm-npu-sota-loop/SKILL.md` | Update `../../references/`, `../../model-pr-optimization-history/`, `../../kernel-pilot/` |
| `tests/test_repository_hygiene.py` | Remove kernel-pilot/model-pr-history skill references, update paths |
| `tests/test_model_pr_history.py` | Update query.py path and dossier paths |
| `README.md` | Update all 50 path references for new structure |
| `README_zh.md` | Update all 50 path references for new structure |

### Files/Directories Deleted

| Item | Reason |
|------|--------|
| `INSTRUCTIONS.md` | Merged into Agent.md |
| `AGENTS.md` | Merged into Agent.md |
| `references/` (entire directory) | Split into reference/ subdirectories |
| `model-pr-optimization-history/` | Content migrated to reference/pr_history/ and scripts/ |
| `kernel-pilot/` | Deleted (workflow covered by op-migration skill) |
| `patches/` | Empty, no target architecture equivalent |
| `model-config-index.json` (was in references/) | Placeholder, no actual content |

---

### Task 1: Create Target Directory Structure + Move Reference Files

**Files:**
- Create: `reference/knowledge/README.md`
- Create: `reference/code-style/`, `reference/io_specs/`, `reference/pr_history/`
- Move: 7 files from `references/` to `reference/` subdirectories
- Delete: `references/` directory (after move)

- [ ] **Step 1: Create all target directories**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow
mkdir -p reference/knowledge reference/code-style reference/io_specs reference/pr_history
mkdir -p baseline scripts
```

- [ ] **Step 2: Move reference files to new locations**

```bash
# knowledge
git mv references/npu-specs.json reference/knowledge/npu-specs.json

# code-style
git mv references/custom-code-style.md reference/code-style/custom-code-style.md

# io_specs
git mv references/accuracy-artifact-schema.md reference/io_specs/accuracy-artifact-schema.md
git mv references/perf-artifact-schema.md reference/io_specs/perf-artifact-schema.md
git mv references/profiling-artifact-schema.md reference/io_specs/profiling-artifact-schema.md
git mv references/run-manifest-template.md reference/io_specs/run-manifest-template.md

# baseline (xllm-benchmark from references)
git mv references/xllm-benchmark.md baseline/xllm-benchmark.md
```

- [ ] **Step 3: Delete model-config-index.json (placeholder, no content)**

```bash
git rm references/model-config-index.json
```

- [ ] **Step 4: Create reference/knowledge/README.md**

Write the following to `reference/knowledge/README.md`:

```markdown
# reference/knowledge/ — Domain Knowledge

Immutable domain rules and hardware specifications that inform Agent decisions.

## Contents

- `npu-specs.json` — NPU hardware peak FLOPs, bandwidth, and HBM specs for Atlas A3/A2 devices

## Principles

- Files here are **read-only references** — never modify based on a single run
- New domain knowledge (operator limits, memory allocation strategies) is added here, not in skill-local references
- For code style conventions, see `reference/code-style/`
- For interface contracts, see `reference/io_specs/`
```

- [ ] **Step 5: Create baseline/README.md**

Write the following to `baseline/README.md`:

```markdown
# baseline/ — Performance Acceptance Criteria

Performance baseline data for each model on different NPU hardware. Used as the
reference point for optimization validation (Phase 1 of the evidence loop).

## Contents

- `xllm-benchmark.md` — Qwen35-27B baseline performance and accuracy data

## Usage

- Before claiming an optimization gain, compare against the relevant baseline entry
- Baseline data must be collected under warmup conditions with documented workload
- New baseline entries are added when a new model/NPU combo is formally validated
```

- [ ] **Step 6: Delete old references/ directory**

After all files have been moved, the `references/` directory should be empty. Verify and remove:

```bash
ls references/   # Should show no remaining files
git rm -r references/
```

- [ ] **Step 7: Commit structural moves**

```bash
git add -A
git commit -m "refactor: move references/ to reference/ subdirectories (knowledge, code-style, io_specs) and create baseline/

- references/npu-specs.json → reference/knowledge/npu-specs.json
- references/custom-code-style.md → reference/code-style/custom-code-style.md
- references/{accuracy,perf,profiling}-artifact-schema.md → reference/io_specs/
- references/run-manifest-template.md → reference/io_specs/
- references/xllm-benchmark.md → baseline/xllm-benchmark.md
- references/model-config-index.json → deleted (placeholder)
- references/ → deleted (moved to reference/)
"
```

---

### Task 2: Create config.json (SSOT Configuration)

**Files:**
- Create: `config.json`

- [ ] **Step 1: Create config.json with npu_specs in static block**

Write the following to `config.json`:

```json
{
  "active": {
    "model": "qwen35-27b",
    "framework": "xllm",
    "npu": "Atlas 800I A3",
    "tp": 4,
    "mtp": 3,
    "dtype": "bfloat16",
    "batch_size": 1,
    "max_model_len": 8192
  },
  "full_test": {
    "models": ["qwen35-27b", "deepseek-v3", "glm-5"],
    "frameworks": ["xllm", "vllm-ascend", "sglang-npu"],
    "npus": ["Atlas 800I A3", "Atlas 300I A2 64GB"]
  },
  "static": {
    "npu_specs": {
      "atlas_800i_a3_per_npu_estimate": {
        "name": "Huawei Atlas 800I A3 per-NPU estimate",
        "source": "https://e.huawei.com/cn/products/computing/ascend/atlas-800i-a3",
        "notes": "Derived from the official 8-NPU server specification: 4.48 PFLOPS FP16, 8.96 POPS INT8, 8 * 128GB on-chip memory, 3.2TB/s memory bandwidth. Use npu-smi, CANN PlatformAscendC, and profiling artifacts for formal conclusions.",
        "peak_flops": {
          "bf16": null,
          "fp16": 560000000000000,
          "int8": 1120000000000000
        },
        "memory_bandwidth_bytes_per_sec": 3200000000000,
        "hbm_bytes": 137438953472
      },
      "atlas_800t_a3_per_npu_estimate": {
        "name": "Huawei Atlas 800T A3 per-NPU estimate",
        "source": "https://e.huawei.com/cn/products/computing/ascend/atlas-800t-a3",
        "notes": "Derived from the official 8-NPU server specification: 6.0 PFLOPS FP16, 12.0 POPS INT8, 8 * 128GB on-chip memory, 3.2TB/s memory bandwidth.",
        "peak_flops": {
          "bf16": null,
          "fp16": 750000000000000,
          "int8": 1500000000000000
        },
        "memory_bandwidth_bytes_per_sec": 3200000000000,
        "hbm_bytes": 137438953472
      },
      "atlas_300i_a2_64gb": {
        "name": "Huawei Atlas 300I A2 64GB",
        "source": "https://e.huawei.com/cn/products/computing/ascend/atlas-300i-a2",
        "notes": "Official product page lists 280 TFLOPS FP16, 560 TOPS INT8, 64GB on-chip memory, and 1.6TB/s memory bandwidth.",
        "peak_flops": {
          "bf16": null,
          "fp16": 280000000000000,
          "int8": 560000000000000
        },
        "memory_bandwidth_bytes_per_sec": 1600000000000,
        "hbm_bytes": 68719476736
      },
      "atlas_300i_a2_32gb": {
        "name": "Huawei Atlas 300I A2 32GB",
        "source": "https://e.huawei.com/cn/products/computing/ascend/atlas-300i-a2",
        "notes": "Official product page lists 280 TFLOPS FP16, 560 TOPS INT8, 32GB on-chip memory, and 0.8TB/s memory bandwidth.",
        "peak_flops": {
          "bf16": null,
          "fp16": 280000000000000,
          "int8": 560000000000000
        },
        "memory_bandwidth_bytes_per_sec": 800000000000,
        "hbm_bytes": 34359738368
      }
    }
  }
}
```

- [ ] **Step 2: Delete standalone npu-specs.json (SSOT — config.json is now the source)**

```bash
git rm reference/knowledge/npu-specs.json
```

- [ ] **Step 3: Update reference/knowledge/README.md to remove npu-specs.json reference**

Edit `reference/knowledge/README.md`: remove the `npu-specs.json` entry from Contents section and add note about config.json:

```markdown
# reference/knowledge/ — Domain Knowledge

Immutable domain rules that inform Agent decisions.

## Contents

(Directory currently stores domain knowledge rules. NPU hardware specs are now in `config.json` under `static.npu_specs`.)

## Principles

- Files here are **read-only references** — never modify based on a single run
- New domain knowledge (operator limits, memory allocation strategies) is added here, not in skill-local references
- NPU hardware specs: read from `config.json` → `static.npu_specs`
- For code style conventions, see `reference/code-style/`
- For interface contracts, see `reference/io_specs/`
```

- [ ] **Step 4: Commit config.json creation**

```bash
git add -A
git commit -m "feat: add config.json as unified configuration SSOT

- active: qwen35-27b on Atlas 800I A3, TP=4, MTP=3
- full_test: cross-model and cross-framework test targets
- static.npu_specs: NPU hardware specs (absorbs npu-specs.json)
- Remove standalone npu-specs.json (SSOT: config.json only)
"
```

---

### Task 3: Migrate model-pr-optimization-history to reference/pr_history/ and scripts/

**Files:**
- Move: dossiers + SKILL.md + card-schema.md → `reference/pr_history/`
- Move: `scripts/query.py` → `scripts/`
- Delete: `model-pr-optimization-history/` directory

- [ ] **Step 1: Move dossiers and supporting files**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow

git mv model-pr-optimization-history/xllm/deepseek-v3.md reference/pr_history/deepseek-v3.md
git mv model-pr-optimization-history/xllm/glm-5.md reference/pr_history/glm-5.md
git mv model-pr-optimization-history/xllm/qwen35-mtp.md reference/pr_history/qwen35-mtp.md
git mv model-pr-optimization-history/references/card-schema.md reference/pr_history/card-schema.md
git mv model-pr-optimization-history/SKILL.md reference/pr_history/SKILL.md
```

- [ ] **Step 2: Move query.py to top-level scripts/**

```bash
git mv model-pr-optimization-history/scripts/query.py scripts/query.py
```

- [ ] **Step 3: Delete remaining model-pr-optimization-history directory**

```bash
# Verify remaining files
ls model-pr-optimization-history/  # xllm/ dir should be empty after dossier moves
ls model-pr-optimization-history/xllm/  # should be empty

git rm -r model-pr-optimization-history/
```

- [ ] **Step 4: Create scripts/README.md**

Write the following to `scripts/README.md`:

```markdown
# scripts/ — Deterministic Engine

Automation scripts for compilation, service launch, performance testing, and accuracy testing.
These scripts are **deterministic** — LLM must not modify their logic. Changes require human review.

## Contents

- `query.py` — Query model dossiers in `reference/pr_history/` by model, keyword, or path
- `collect_evalscope_results.py` — Collect and normalize evalscope benchmark results
- `compare_npu_benchmark.py` — Cross-framework NPU benchmark comparison
- `validate_framework_cli.py` — Validate framework CLI parameters

## Principles

- Scripts here are cross-skill shared utilities; skill-specific scripts stay in their skill's `scripts/` subdirectory
- All scripts must be runnable from the repository root
- Parameter changes go into `config.json`, not hardcoded in scripts
```

- [ ] **Step 5: Commit migration**

```bash
git add -A
git commit -m "refactor: migrate model-pr-optimization-history to reference/pr_history/ and scripts/

- Dossiers (deepseek-v3, glm-5, qwen35-mtp) → reference/pr_history/
- SKILL.md + card-schema.md → reference/pr_history/
- query.py → scripts/
- model-pr-optimization-history/ → deleted
"
```

---

### Task 4: Move Cross-Skill Shared Scripts to scripts/

**Files:**
- Move: 3 scripts from `skills/xllm-npu-benchmark/scripts/` → `scripts/`

- [ ] **Step 1: Move benchmark shared scripts**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow

git mv skills/xllm-npu-benchmark/scripts/collect_evalscope_results.py scripts/collect_evalscope_results.py
git mv skills/xllm-npu-benchmark/scripts/compare_npu_benchmark.py scripts/compare_npu_benchmark.py
git mv skills/xllm-npu-benchmark/scripts/validate_framework_cli.py scripts/validate_framework_cli.py
```

- [ ] **Step 2: Verify benchmark skill scripts directory**

```bash
ls skills/xllm-npu-benchmark/scripts/  # Should be empty now
# If empty, remove the directory
rm skills/xllm-npu-benchmark/scripts/
```

Note: If `git rm` of an empty directory is needed, use `git rm -r` approach or simply leave the empty dir for git to ignore.

- [ ] **Step 3: Commit script consolidation**

```bash
git add -A
git commit -m "refactor: move cross-skill shared scripts to top-level scripts/

- collect_evalscope_results.py, compare_npu_benchmark.py, validate_framework_cli.py
  moved from skills/xllm-npu-benchmark/scripts/ to scripts/
- Skill-local scripts (profiler, eval-runner, pipeline-analysis) remain in their skill dirs
"
```

---

### Task 5: Delete kernel-pilot/ and patches/

**Files:**
- Delete: `kernel-pilot/` (entire directory)
- Delete: `patches/` (entire directory)

- [ ] **Step 1: Delete kernel-pilot/**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow
git rm -r kernel-pilot/
```

- [ ] **Step 2: Delete patches/**

```bash
git rm -r patches/
```

- [ ] **Step 3: Commit deletions**

```bash
git add -A
git commit -m "refactor: remove kernel-pilot/ and patches/

- kernel-pilot/ deleted (workflow covered by op-migration skill)
- patches/ deleted (empty, no target architecture equivalent)
"
```

---

### Task 6: Update .gitignore

**Files:**
- Modify: `.gitignore`

- [ ] **Step 1: Add code/ and runs/ to .gitignore**

Edit `.gitignore` to add two new entries. The current content is:

```
__pycache__/
*.py[cod]
.pytest_cache/
```

Change to:

```
__pycache__/
*.py[cod]
.pytest_cache/
code/
runs/
```

- [ ] **Step 2: Commit .gitignore update**

```bash
git add .gitignore
git commit -m "chore: add code/ and runs/ to .gitignore

Block source code mount and run logs from entering the repo.
"
```

---

### Task 7: Update humanize/README.md

**Files:**
- Modify: `humanize/README.md`

- [ ] **Step 1: Rewrite humanize/README.md with usage description**

The current content defines only the ledger contract. Replace with a fuller description that includes usage, write-back rules, and directory purpose:

```markdown
# humanize/ — Experience Flywheel

This directory is the dynamic experience pool. Agents accumulate troubleshooting
and tuning lessons here, making the workspace "smarter with every use."

## Purpose

- Preserve validated optimization conclusions, troubleshooting experience, and recurring pitfalls
- Provide reference for future Agents executing similar tasks
- Complement static knowledge in `reference/` with run-proven lessons

## Write-back Rules

- Only write **validated lessons** (not guesses, not hypotheses)
- Each lesson must include: scenario, root cause, solution, verification method
- Concrete ledger files (attempt-ledger, optimization-ledger, source-idea-ledger, lineage.jsonl)
  are generated under each run root (`runs/`), not here
- Only lessons with durable value are promoted back into this directory

## Ledger Contract

Concrete ledgers live under each run root:

```text
<run-root>/humanize/
  attempt-ledger.md
  optimization-ledger.md
  source-idea-ledger.md
  lineage.jsonl
```

Promote durable lessons into the repository:

- model-specific risks and wins → `reference/pr_history/`
- profiling lessons → skill references or `reference/pr_history/`
- reusable artifact fields → `reference/io_specs/`
- repeatable workflows → `skills/`

This prevents stale local experiment paths from becoming global guidance while
still preserving the evidence-loop record for each run.

## Current State

(Directory starts empty. Content fills organically as Agents accumulate real
troubleshooting and tuning experience.)
```

- [ ] **Step 2: Commit humanize update**

```bash
git add humanize/README.md
git commit -m "docs: update humanize/README.md with usage description and write-back rules

- Add purpose, write-back rules, and directory state description
- Keep existing ledger contract content
- Align with knowledge flywheel architectural principle
"
```

---

### Task 8: Write Agent.md (Merged Document)

**Files:**
- Create: `Agent.md`

- [ ] **Step 1: Create Agent.md with merged content**

Write the following to `Agent.md`. This merges AGENTS.md (7 rules), INSTRUCTIONS.md (common tasks, start-here), and CLAUDE.md (6 rules) into a deduplicated 8-rule document with skill routing, directory guide, and hygiene rules:

```markdown
# Agent.md — NPU Agent Workspace System Prompt

Guidelines for Codex, opencode, Claude Code, and other coding agents working in
this repository.

This repository contains reusable AI coding workflows for NPU large-model serving
optimization. The first supported landing target is xLLM, but the evidence
model should stay portable to other OpenAI-compatible NPU serving frameworks
when their adapters and runbooks are added.

## 1. Repository Purpose

- Preserve repeatable NPU optimization workflows as skills, schemas, scripts,
  prompts, and model history.
- Keep benchmark, profiling, accuracy, capacity, and review evidence explicit.
- Separate generic workflow rules from model-specific or framework-specific
  lessons.
- Promote durable lessons into `reference/pr_history/`, `reference/`,
  `skills/*/references/`, or run-root `humanize/` ledgers.

## 2. Core Engineering Constraints

1. **Think Before Editing**
   - State assumptions when the task is ambiguous.
   - If multiple interpretations change the implementation, ask before editing.
   - If a simpler approach exists, say so before choosing a heavier path.
   - Prefer the existing skill structure, artifact schema, and naming conventions.
   - For trivial documentation fixes, make the small obvious edit and verify it.

2. **Simplicity First**
   - Implement only what the user asked for and what the validation goal requires.
   - Do not add abstractions, configuration, compatibility layers, or speculative
     features for a single-use need.
   - If a change starts to sprawl, shrink it back to the smallest verifiable diff.

3. **Evidence Before Patch**
   - Performance optimization requires a warmed-up baseline and profiling evidence
     before code changes.
   - Accuracy fixes require a stable reproducer before broader evaluation.
   - Profiling captures explain bottlenecks; they are not formal before/after
     performance results.
   - Do not claim a gain without raw artifacts, metrics, and the exact workload.

4. **Keep Changes Surgical**
   - Touch only files needed for the request.
   - Do not rewrite skill bodies into long essays. Keep `SKILL.md` procedural and
     move detailed material into `references/` when needed.
   - Do not delete failed attempts or historical lessons; convert them into
     reusable notes.
   - Do not add local paths, private host names, internal IPs, private datasets, or
     secrets to committed files.

5. **Fair Comparisons Only**
   - Compare frameworks under the same model, tokenizer, dtype, hardware, workload,
     sampling parameters, and SLA. Tune each framework independently.

6. **Profiling Is Diagnostic**
   - Profiling captures explain bottlenecks but do not replace non-profiling
     before/after performance runs.

7. **Review-Gated Evidence Loop**
   - Use Research → Learn → Code → Review → Validate → Record.
   - This is inspired by PolyARCH/humanize's RLCR review discipline, but this
     repository does not implement Humanize RLCR itself.

8. **Validate and Record**
   - Run repository tests after changing schemas, scripts, or skill structure.
   - For documentation-only edits, at least run markdown-sensitive hygiene checks
     when available.
   - Update README / README_zh / Agent.md together when changing public workflow
     concepts.
   - End every optimization or bug-fix loop by recording reusable lessons in a
     ledger, reference, or model PR history.

## 3. Task → Skill Routing

| Task | Start With |
|---|---|
| End-to-end optimization goal | `skills/xllm-npu-sota-loop/SKILL.md` |
| Launch service or collect evalscope artifacts | `skills/xllm-npu-eval-runner/SKILL.md` |
| Fair benchmark comparison | `skills/xllm-npu-benchmark/SKILL.md` |
| msprof / MindStudio profiling analysis | `skills/xllm-npu-profiler/SKILL.md` |
| Prefill/decode boundary, layer timing, or rank skew | `skills/xllm-npu-pipeline-analysis/SKILL.md` |
| Capacity, HBM, KV cache, concurrency, or OOM risk | `skills/xllm-npu-capacity-planner/SKILL.md` |
| FLOPs, MFU, or hardware lower-bound estimates | `skills/xllm-npu-compute-simulation/SKILL.md` |
| Garbled output, CEval drop, or GPU/NPU mismatch | `skills/xllm-npu-accuracy-debug/SKILL.md` |
| Crash, hang, HCCL, graph, or PagedAttention incident | `skills/xllm-npu-incident-triage/SKILL.md` |
| NPU code review before PR | `skills/xllm-npu-code-review/SKILL.md` |
| Operator migration | `skills/xllm-npu-op-migration/SKILL.md` |

## 4. Configuration and Directory Guide

### Cascade Priority

1. `config.json` — active configuration for current work
2. `reference/` — static domain knowledge and interface contracts
3. `skills/*/references/` — skill-specific detailed references
4. `skills/*/SKILL.md` — procedural execution workflow

### Directory Descriptions

- **`config.json`** — Unified configuration entry: `active` (current model/NPU/parameters), `full_test` (cross-model/framework test targets), `static` (NPU hardware specs). Single source of truth for all configuration.
- **`reference/`** — Static knowledge base, immutable domain rules:
  - `knowledge/` — Domain knowledge (immutable rules; NPU specs are in config.json `static.npu_specs`)
  - `code-style/` — Code style conventions (C++/Python/NPU coding standards)
  - `pr_history/` — Evolution history (model dossiers, PR change logs, queryable via `scripts/query.py`)
  - `io_specs/` — Interface contracts (artifact schemas, manifest templates defining skill-Agent interaction)
- **`baseline/`** — Performance acceptance criteria. Baseline data for each model on different NPU hardware. Compare against these before claiming an optimization gain.
- **`humanize/`** — Experience flywheel. Dynamic experience pool for validated troubleshooting and tuning lessons. Concrete ledgers live under run roots; only durable lessons are promoted here.
- **`scripts/`** — Deterministic engine. Cross-skill shared automation scripts (compilation, testing, profiling). LLM must not modify script logic; changes require human review.
- **`code/`** — External mount area. Target framework source code (one framework per directory). Not committed; `.gitignore` blocks it. Agent should read source here but never modify repository content based on code/ contents.
- **`runs/`** — Execution现场. Preserves the last 5 runs of compilation, testing, and profiling logs. Not committed; `.gitignore` blocks it. Agent reads logs here for evidence but does not commit run artifacts.

## 5. Documentation Hygiene

- Keep public entry documents stable and generic.
- Put model-specific lessons under `reference/pr_history/` or skill
  `references/`, not in the main README or generic workflow document.
- Remove temporary blog drafts, local environment notes, and stale roadmap text
  before opening a PR.
- Update README links when adding or removing documentation entry points.
- Keep `Agent.md` and `Claude.md` conceptually aligned.
```

- [ ] **Step 2: Commit Agent.md**

```bash
git add Agent.md
git commit -m "feat: create Agent.md as unified Agent system prompt

Merged AGENTS.md + INSTRUCTIONS.md + CLAUDE.md into single Agent entry:
- 8 deduplicated core engineering constraints
- Task → Skill routing table (11 skills, no prompt references)
- Configuration and directory guide (cascade priority + descriptions)
- Documentation hygiene rules
"
```

---

### Task 9: Rewrite Claude.md as Thin Redirect

**Files:**
- Modify: `Claude.md`

- [ ] **Step 1: Replace Claude.md content with thin redirect**

Replace the entire content of `Claude.md` with:

```markdown
# Claude.md

核心规则、Skill路由、目录说明请读取 [Agent.md](Agent.md)。

本文件仅记录 Claude Code 工具专有的补充约定（如有）。
```

- [ ] **Step 2: Commit Claude.md update**

```bash
git add Claude.md
git commit -m "refactor: reduce Claude.md to thin redirect to Agent.md

SSOT: all Agent rules now live in Agent.md. Claude.md only supplements
Claude Code-specific conventions if needed.
"
```

---

### Task 10: Delete INSTRUCTIONS.md and AGENTS.md

**Files:**
- Delete: `INSTRUCTIONS.md`
- Delete: `AGENTS.md`

- [ ] **Step 1: Delete both files**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow
git rm INSTRUCTIONS.md
git rm AGENTS.md
```

- [ ] **Step 2: Commit deletions**

```bash
git commit -m "refactor: remove INSTRUCTIONS.md and AGENTS.md

Content merged into Agent.md. No information loss.
"
```

---

### Task 11: Update Skill SKILL.md Path References

**Files:**
- Modify: 11 skill SKILL.md files

This task updates all `../../references/` → `../../reference/` subdirectory paths,
removes `../../kernel-pilot/` references, and changes `../../model-pr-optimization-history/` to `../../reference/pr_history/`.

Each skill is updated independently. For efficiency, use `sed` for path pattern replacements where the substitution is mechanical, and manual Edit for context-dependent changes.

- [ ] **Step 1: Update xllm-npu-accuracy-debug/SKILL.md**

Edit `skills/xllm-npu-accuracy-debug/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 33 | `../../references/accuracy-artifact-schema.md` | `../../reference/io_specs/accuracy-artifact-schema.md` |
| 35 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |

- [ ] **Step 2: Update xllm-npu-benchmark/SKILL.md**

Edit `skills/xllm-npu-benchmark/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 32 | `../../references/perf-artifact-schema.md` | `../../reference/io_specs/perf-artifact-schema.md` |
| 33 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |

- [ ] **Step 3: Update xllm-npu-capacity-planner/SKILL.md**

Edit `skills/xllm-npu-capacity-planner/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 27 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |
| 68 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |
| 69 | `../../references/perf-artifact-schema.md` | `../../reference/io_specs/perf-artifact-schema.md` |

- [ ] **Step 4: Update xllm-npu-code-review/SKILL.md**

Edit `skills/xllm-npu-code-review/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 37 | `../../references/custom-code-style.md` | `../../reference/code-style/custom-code-style.md` |

Also update the reference to `AGENTS.md` if present (should reference `Agent.md` instead).

- [ ] **Step 5: Update xllm-npu-compute-simulation/SKILL.md**

Edit `skills/xllm-npu-compute-simulation/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 16 | `../../references/npu-specs.json` | `../../config.json` (refer to `static.npu_specs`) |
| 51 | `../../references/npu-specs.json` | `../../config.json` (refer to `static.npu_specs`) |
| 52 | `../../references/model-config-index.json` | Remove this reference (file deleted) |

For line 16 and 51: Change the reference text to indicate reading from `config.json → static.npu_specs` instead of a standalone json file. Example: "NPU 规格来自 `config.json` 的 `static.npu_specs` 字段。"

For line 52: Remove the model-config-index reference entirely (file was deleted as placeholder).

- [ ] **Step 6: Update xllm-npu-eval-runner/SKILL.md**

Edit `skills/xllm-npu-eval-runner/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 141 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |
| 255 | `../../references/accuracy-artifact-schema.md` | `../../reference/io_specs/accuracy-artifact-schema.md` |

Note: Line 141 has display text `references/run-manifest-template.md` and link `../../references/run-manifest-template.md`. Update both to `reference/io_specs/run-manifest-template.md` and `../../reference/io_specs/run-manifest-template.md`. Same for line 255.

- [ ] **Step 7: Update xllm-npu-incident-triage/SKILL.md**

Edit `skills/xllm-npu-incident-triage/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 268 | `../xllm-npu-profiler/references/ascend-profiling-formats.md` | No change needed (cross-skill reference, skill-local references stay unchanged) |

No `../../references/` references found. No changes needed for this skill.

- [ ] **Step 8: Update xllm-npu-op-migration/SKILL.md**

Edit `skills/xllm-npu-op-migration/SKILL.md`:

Check for any `kernel-pilot` references. If found (e.g., "use kernel-pilot only when profiling justifies"), update to remove or replace with a note that kernel-level work follows profiling evidence. No `../../references/` paths found — skill uses only local references.

- [ ] **Step 9: Update xllm-npu-pipeline-analysis/SKILL.md**

Edit `skills/xllm-npu-pipeline-analysis/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 14 | `../../references/profiling-artifact-schema.md` | `../../reference/io_specs/profiling-artifact-schema.md` |
| 97 | `../../references/profiling-artifact-schema.md` | `../../reference/io_specs/profiling-artifact-schema.md` |

Line 96 (`../xllm-npu-profiler/references/source-map.md`) stays unchanged (skill-local reference).

- [ ] **Step 10: Update xllm-npu-profiler/SKILL.md**

Edit `skills/xllm-npu-profiler/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 19 | `../../references/profiling-artifact-schema.md` | `../../reference/io_specs/profiling-artifact-schema.md` |
| 21 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |

- [ ] **Step 11: Update xllm-npu-sota-loop/SKILL.md**

Edit `skills/xllm-npu-sota-loop/SKILL.md`:

| Line | Old | New |
|------|-----|-----|
| 38 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |
| 77 | `../../references/perf-artifact-schema.md` | `../../reference/io_specs/perf-artifact-schema.md` |
| 78 | `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |
| 124 | `../../kernel-pilot/SKILL.md` | Remove this reference (kernel-pilot deleted) |
| 202 | `../../model-pr-optimization-history/xllm/qwen35-mtp.md` | `../../reference/pr_history/qwen35-mtp.md` |

Also update line 44 skill invocation reference: "Use model-pr-optimization-history" → "Use `reference/pr_history/` SKILL.md".

Lines 200-201 (cross-skill references to benchmark and profiler) stay unchanged (skill-local references).

- [ ] **Step 12: Commit all skill path updates**

```bash
git add skills/
git commit -m "refactor: update all skill SKILL.md path references for new structure

- ../../references/ → ../../reference/{knowledge,code-style,io_specs}/
- ../../references/npu-specs.json → ../../config.json (static.npu_specs)
- ../../model-pr-optimization-history/ → ../../reference/pr_history/
- ../../kernel-pilot/ → removed (directory deleted)
- ../../references/model-config-index.json → removed (placeholder deleted)
- Skill-local references/ unchanged
"
```

---

### Task 12: Update tests/

**Files:**
- Modify: `tests/test_repository_hygiene.py`
- Modify: `tests/test_model_pr_history.py`

- [ ] **Step 1: Update test_repository_hygiene.py**

Edit `tests/test_repository_hygiene.py`:

Changes needed:
1. Remove `ROOT / "kernel-pilot/SKILL.md"` and `ROOT / "model-pr-optimization-history/SKILL.md"` from `test_skill_frontmatter_has_name_and_description` (both directories deleted)
2. Remove same references from `test_skills_do_not_hardcode_single_agent_install_paths`
3. Update the skill file list to only use `ROOT.glob("skills/*/SKILL.md")` (no extra top-level skills)
4. Verify path references work with `reference/` instead of `references/`

Updated test file:

```python
from pathlib import Path
import re


ROOT = Path(__file__).resolve().parents[1]


def text_files():
    suffixes = {".md", ".py", ".sh", ".json", ".jsonl", ".yaml", ".yml"}
    for path in ROOT.rglob("*"):
        if ".git" in path.parts:
            continue
        if path.is_file() and path.suffix in suffixes:
            yield path


def test_skill_frontmatter_has_name_and_description():
    skill_files = list(ROOT.glob("skills/*/SKILL.md")) + [
        ROOT / "reference/pr_history/SKILL.md",
    ]
    for skill in skill_files:
        text = skill.read_text(encoding="utf-8")
        assert text.startswith("---\n"), skill
        header = text.split("---", 2)[1]
        assert re.search(r"^name:\s*\S+", header, re.M), skill
        assert re.search(r"^description:\s+.+", header, re.M), skill


def test_skills_do_not_hardcode_single_agent_install_paths():
    forbidden = [
        ".opencode/skills/",
        "$CODEX_HOME/skills/",
        "~/.claude/skills/",
    ]
    skill_files = list(ROOT.glob("skills/*/SKILL.md")) + [
        ROOT / "reference/pr_history/SKILL.md",
    ]
    for skill in skill_files:
        text = skill.read_text(encoding="utf-8")
        for item in forbidden:
            assert item not in text, f"{item} found in {skill}"


def test_no_public_readme_forbidden_source_reference():
    forbidden = [
        "B" + "Buf",
        "AI-Coding-" + "Auto-" + "Driven-SKILLS",
        "Auto-" + "Driven",
    ]
    for path in [ROOT / "README.md", ROOT / "README_zh.md"]:
        text = path.read_text(encoding="utf-8")
        for item in forbidden:
            assert item not in text, f"{item} found in {path}"


def test_no_obvious_local_or_credential_strings():
    patterns = [
        r"/home/[A-Za-z][A-Za-z0-9._-]*",
        r"\b[a-z][0-9]{8}\b",
        r"\b[a-z]+pengju\b",
        r"\b[a-z]+pengju[0-9]*\b",
        r"SSH config:\s*`?\d+`?",
        r"\bhost\s*[:=]\s*\d+\b",
        r"192\.168\.",
        "xllm" + "-gpj",
        "jd" + "_openai_20k",
        "BEGIN " + "RSA",
        "PRIVATE " + "KEY",
    ]
    combined = "\n".join(p.read_text(encoding="utf-8", errors="ignore") for p in text_files())
    for pattern in patterns:
        assert not re.search(pattern, combined), pattern

    allowed_ips = {"0.0.0.0", "127.0.0.1"}
    for match in re.finditer(r"\b\d{1,3}(?:\.\d{1,3}){3}\b", combined):
        assert match.group(0) in allowed_ips, match.group(0)
```

- [ ] **Step 2: Update test_model_pr_history.py**

Edit `tests/test_model_pr_history.py`:

Changes needed:
1. Update `QUERY` path: `ROOT / "model-pr-optimization-history" / "scripts" / "query.py"` → `ROOT / "scripts" / "query.py"`
2. Update assertion path: `"xllm/qwen35-mtp"` → `"pr_history/qwen35-mtp"` or `"qwen35-mtp"` (depends on query.py output format after migration)
3. Update assertion: `"qwen35-mtp.md"` stays (filename unchanged)

Updated test file:

```python
import subprocess
import sys
from pathlib import Path


ROOT = Path(__file__).resolve().parents[1]
QUERY = ROOT / "scripts" / "query.py"


def run_query(*args):
    return subprocess.run(
        [sys.executable, str(QUERY), *args],
        cwd=ROOT,
        check=True,
        text=True,
        capture_output=True,
    ).stdout


def test_query_supports_model_and_keyword_filters():
    out = run_query("--model", "Qwen3.5", "--keyword", "MTP")
    assert "qwen35-mtp" in out
    assert "Acceptance Rate" in out or "MTP" in out


def test_query_supports_path_filter():
    out = run_query("--framework", "xllm", "--path", "MTPWorkerImpl::run_validate")
    assert "qwen35-mtp.md" in out
    assert "MTPWorkerImpl::run_validate" in out
```

Note: The query.py script needs its internal path references updated too (it currently references `model-pr-optimization-history/xllm/`). This must be done in the same commit.

- [ ] **Step 3: Update scripts/query.py internal path references**

Read `scripts/query.py` and update any internal path references from `model-pr-optimization-history/` to `reference/pr_history/`. The script currently references `model-pr-optimization-history/xllm/` as the dossier directory. Change to `reference/pr_history/`.

- [ ] **Step 4: Run tests to verify**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow
python -m pytest tests/ -v
```

Expected: All tests PASS.

- [ ] **Step 5: Commit test and query.py updates**

```bash
git add tests/ scripts/query.py
git commit -m "refactor: update tests and query.py for new directory structure

- test_repository_hygiene: remove kernel-pilot/model-pr-history, add reference/pr_history
- test_model_pr_history: update paths to scripts/query.py and reference/pr_history/
- query.py: update internal paths from model-pr-optimization-history/ to reference/pr_history/
"
```

---

### Task 13: Update README.md

**Files:**
- Modify: `README.md`

- [ ] **Step 1: Read README.md and identify all path references requiring update**

Based on the exploration, ~50 lines reference restructuring-affected paths. Key categories:
- `references/` → split into `reference/knowledge/`, `reference/code-style/`, `reference/io_specs/`
- `model-pr-optimization-history/` → `reference/pr_history/`
- `kernel-pilot/` → removed
- `patches/` → removed
- `AGENTS.md` → `Agent.md`
- `INSTRUCTIONS.md` → removed
- Repository Layout section → rewrite for new structure

- [ ] **Step 2: Update all path references in README.md**

Use Edit tool to make precise replacements for each affected section. Key changes:

1. **Entry Points table**: Update `references/` → `reference/knowledge/`, `reference/io_specs/`, etc.; remove `kernel-pilot/`, `model-pr-optimization-history/`, `patches/` rows; add `config.json`, `baseline/`, `scripts/` rows
2. **Skill Routing table**: Remove kernel-pilot and model-pr-optimization-history rows
3. **Artifact Schema references**: `references/xxx.md` → `reference/io_specs/xxx.md`
4. **Repository Layout section**: Rewrite to match target structure
5. **Symlink installation**: Remove kernel-pilot and model-pr-optimization-history symlinks
6. **Evidence loop reference**: Update `model-pr-optimization-history` → `reference/pr_history/`
7. **Hygiene rules**: Update `model-pr-optimization-history/` → `reference/pr_history/`, `references/` → `reference/`

Exact line-by-line edits must be determined by reading the current README.md content. This step requires careful manual editing.

- [ ] **Step 3: Commit README.md update**

```bash
git add README.md
git commit -m "docs: update README.md for new directory structure

- references/ → reference/ (knowledge, code-style, io_specs)
- model-pr-optimization-history/ → reference/pr_history/
- kernel-pilot/ → removed
- patches/ → removed
- AGENTS.md/INSTRUCTIONS.md → Agent.md
- Add config.json, baseline/, scripts/ descriptions
"
```

---

### Task 14: Update README_zh.md

**Files:**
- Modify: `README_zh.md`

- [ ] **Step 1: Apply the same path reference updates as README.md**

Mirror all changes from Task 13 in the Chinese version. Same categories of updates:
- `references/` → split references
- `model-pr-optimization-history/` → `reference/pr_history/`
- `kernel-pilot/` → removed
- `patches/` → removed
- `AGENTS.md` → `Agent.md`
- `INSTRUCTIONS.md` → removed
- Repository Layout → rewritten

- [ ] **Step 2: Commit README_zh.md update**

```bash
git add README_zh.md
git commit -m "docs: update README_zh.md for new directory structure (Chinese)

Mirror changes from README.md English version.
"
```

---

### Task 15: Update docs/ Workflow Document

**Files:**
- Modify: `docs/npu-ai-coding-standard-workflow.md` (if it references old paths)

- [ ] **Step 1: Check docs/npu-ai-coding-standard-workflow.md for old path references**

Read the file and search for `references/`, `model-pr-optimization-history/`, `kernel-pilot/`, `patches/`, `AGENTS.md`, `INSTRUCTIONS.md`.

If found, update them to match the new structure using Edit tool.

- [ ] **Step 2: Commit if changes needed (skip if no references found)**

```bash
git add docs/
git commit -m "docs: update workflow doc path references for new structure"
```

---

### Task 16: Final Verification

- [ ] **Step 1: Run all tests**

```bash
cd /home/g00510989/xllm_whj/xllm-workflow
python -m pytest tests/ -v
```

Expected: All tests PASS (test_repository_hygiene + test_model_pr_history).

- [ ] **Step 2: Verify directory structure matches target**

```bash
find . -not -path './.git/*' -not -path './.pytest_cache/*' -not -path './__pycache__/*' -not -name '*.pyc' | sort
```

Verify the output matches the target directory structure from the design spec.

- [ ] **Step 3: Verify all skill SKILL.md references resolve**

```bash
# Check that no broken ../../references/ paths remain
grep -r '../../references/' skills/ || echo "No old references found (GOOD)"

# Check that no kernel-pilot references remain
grep -r 'kernel-pilot' skills/ || echo "No kernel-pilot references found (GOOD)"

# Check that no model-pr-optimization-history references remain
grep -r 'model-pr-optimization-history' skills/ || echo "No model-pr-history references found (GOOD)"
```

Expected: All grep commands return empty (no old path references).

- [ ] **Step 4: Verify Agent.md content completeness**

```bash
# Check Agent.md contains all 8 core constraints
grep -c '^\d\. \*\*' Agent.md
```

Expected: 8 (matches the 8 deduplicated constraints).

- [ ] **Step 5: Verify Claude.md is thin redirect**

```bash
wc -l Claude.md
```

Expected: ~3-4 lines (thin redirect only).

- [ ] **Step 6: Verify config.json exists and is valid JSON**

```bash
python -c "import json; json.load(open('config.json')); print('Valid JSON')"
```

Expected: "Valid JSON"

- [ ] **Step 7: Verify old directories/files are gone**

```bash
ls references/ 2>&1       # Expected: error (directory deleted)
ls model-pr-optimization-history/ 2>&1  # Expected: error
ls kernel-pilot/ 2>&1     # Expected: error
ls patches/ 2>&1          # Expected: error
ls INSTRUCTIONS.md 2>&1   # Expected: error
ls AGENTS.md 2>&1         # Expected: error
```

Expected: All commands return "No such file or directory" errors.

- [ ] **Step 8: Final commit if any remaining fixes needed**

```bash
git add -A
git commit -m "chore: final verification fixes for architecture optimization"
```