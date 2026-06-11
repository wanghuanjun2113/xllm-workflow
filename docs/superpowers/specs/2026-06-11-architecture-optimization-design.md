# xllm-workflow Architecture Optimization Design

Date: 2026-06-11

## Motivation

Reorganize the xllm-workflow project to align with six core architectural principles:

1. **Deterministic Execution** — Lock deterministic logic into scripts, prohibit LLM intervention, ensure stable reproduction and minimize token cost.
2. **Audience-Specific Docs** — Human docs are macro and minimal; Agent docs must prescribe detailed execution flows and boundary constraints.
3. **Knowledge Flywheel** — Mandatory retrospective and structured write-back mechanism; every troubleshooting and tuning session feeds durable experience.
4. **Single Source of Truth** — Eliminate duplicated config, specs, or prompts across multiple locations.
5. **Skill Encapsulation** — Semi-structured standard workflows (PR submission, log analysis) are encapsulated as Skill interfaces for Agent dispatch.
6. **Keep It Simple** — Minimize directory depth and coordination links; resist over-engineering while satisfying Agent context supply and anti-hallucination needs.

## Approach

**Incremental restructuring (Approach A)**: Restructure directory layout and merge/split documents step by step. Preserve all existing content semantics; only move, split, or merge structural locations. Each step is independently verifiable via existing tests.

## Target Directory Structure

```
xllm-npu-agent-workspace/
├── .gitignore              # Block code/, runs/, __pycache__/, *.py[cod], .pytest_cache/
├── README.md               # [Human] English overview (updated paths)
├── README_zh.md            # [Human] Chinese overview (updated paths)
├── docs/                   # [Human] Long-form docs (unchanged)
│   ├── assets/
│   └── npu-ai-coding-standard-workflow.md
├── Agent.md                # [Agent] Core system prompt, constraints, skill routing, directory guide
├── Claude.md               # [Agent] Thin redirect to Agent.md; only Claude-specific supplements if any
├── config.json             # [Config] active + full_test + static (SSOT for all configuration)
├── prompts/                # [Human-Agent bridge] Copy-ready task prompt templates (unchanged)
├── reference/              # [Knowledge] Static reference library
│   ├── knowledge/          #   Domain knowledge (npu-specs.json, immutable rules)
│   ├── code-style/         #   Code style conventions (custom-code-style.md)
│   ├── pr_history/         #   Evolution history (model dossiers, PR change logs)
│   └── io_specs/           #   Interface contracts (artifact schemas, manifest template)
├── baseline/               # [Acceptance] Performance baseline data (xllm-benchmark.md + README)
├── humanize/               # [Flywheel] Dynamic experience pool (updated README with usage guide)
├── scripts/                # [Deterministic] Automation scripts (cross-skill shared scripts)
├── skills/                 # [Skill toolbox] 11 Agent skills
│   ├── xllm-npu-accuracy-debug/
│   ├── xllm-npu-benchmark/
│   ├── xllm-npu-capacity-planner/
│   ├── xllm-npu-code-review/
│   ├── xllm-npu-compute-simulation/
│   ├── xllm-npu-eval-runner/
│   ├── xllm-npu-incident-triage/
│   ├── xllm-npu-op-migration/
│   ├── xllm-npu-pipeline-analysis/
│   ├── xllm-npu-profiler/
│   └── xllm-npu-sota-loop/
├── tests/                  # [Guard] Repository hygiene and spec validators (updated paths)
├── code/                   # [External/Not committed] Source code mount, .gitignore blocked
└── runs/                   # [Execution/Not committed] Last 5 run logs, .gitignore blocked
```

## Detailed Changes

### 1. Agent.md Merge (SSOT + Audience Isolation)

**Merge**: AGENTS.md + INSTRUCTIONS.md + CLAUDE.md → Agent.md

**Delete**: INSTRUCTIONS.md

**Claude.md**: Reduced to thin redirect:
```
核心规则、Skill路由、目录说明请读取 Agent.md。
本文件仅记录 Claude Code 工具专有的补充约定（如有）。
```

**Agent.md structure**:
```
## 1. 仓库目的
  ← From AGENTS.md (3 bullet points)

## 2. 核心工程约束（8条）
  ← Deduplicated merge of AGENTS.md 7 rules + CLAUDE.md 6 rules:

  1. Think Before Editing          (CLAUDE only)
  2. Simplicity First              (CLAUDE only)
  3. Evidence Before Patch         (AGENTS#1 + CLAUDE#3 merged)
  4. Keep Changes Surgical         (CLAUDE#4 + AGENTS#6 no-private-data merged)
  5. Fair Comparisons Only         (AGENTS#2)
  6. Profiling Is Diagnostic       (AGENTS#3)
  7. Review-Gated Evidence Loop    (AGENTS#4)
  8. Validate and Record           (AGENTS#7 + CLAUDE#6 merged)

## 3. Task → Skill 路由表
  ← Only skill routing, no prompt references
  ← Prompts are human-facing; stay in README only
  ← 11 rows (kernel-pilot and model-pr-history entries removed)

## 4. 级联配置与目录说明
  ← Brief description of each core directory and read priority:
  - config.json: active/full_test/static config SSOT
  - reference/: static knowledge base
    - knowledge/  → domain knowledge (NPU specs, immutable rules)
    - code-style/ → code style conventions (C++/Python/NPU coding)
    - pr_history/ → evolution history (model dossiers, PR change logs)
    - io_specs/   → interface contracts (skill-Agent interaction schemas)
  - baseline/    → performance acceptance criteria
  - humanize/    → experience flywheel, dynamic experience pool
  - scripts/     → deterministic engine, automation script library
  - code/        → external mount, target framework source, not committed
  - runs/        → execution现场, last 5 run logs, not committed

## 5. 文档卫生规则
  ← From AGENTS.md (updated path references)
```

### 2. reference/ Split (SSOT)

**Split** current `references/` (7 files) into `reference/` (4 subdirectories):

| Destination | Files |
|-------------|-------|
| `reference/knowledge/` | `npu-specs.json` |
| `reference/code-style/` | `custom-code-style.md` |
| `reference/io_specs/` | `accuracy-artifact-schema.md`, `perf-artifact-schema.md`, `profiling-artifact-schema.md`, `run-manifest-template.md` |
| `reference/pr_history/` | `deepseek-v3.md`, `glm-5.md`, `qwen35-mtp.md`, `card-schema.md`, `SKILL.md` (query workflow entry) |

**Delete** after migration: entire `references/` directory

**Delete**: `model-config-index.json` (placeholder, no actual content)

**Move to baseline/**: `xllm-benchmark.md` (from references)

Skill-local `references/` subdirectories remain unchanged (e.g., profiler's `npu-fuse-catalog.md`, benchmark's `npu-fairness-rules.md`).

### 3. config.json Creation (SSOT)

Create unified config with three blocks:

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
    "npu_specs": { ... }
  }
}
```

`static.npu_specs` absorbs `references/npu-specs.json` content. After migration, `reference/knowledge/npu-specs.json` becomes a redirect pointing to `config.json` (or is removed — see implementation plan for decision).

### 4. Top-level Skill Relocation

- **model-pr-optimization-history/** → Dossiers migrate to `reference/pr_history/`, `query.py` migrates to `scripts/`, then directory deleted.
- **kernel-pilot/** → Deleted entirely (content not needed; op-migration skill covers the workflow).

### 5. scripts/ Consolidation

Cross-skill shared scripts move to top-level `scripts/`:

| Script | Source | Reason |
|--------|--------|--------|
| `query.py` | model-pr-optimization-history | PR history query, cross-skill |
| `collect_evalscope_results.py` | xllm-npu-benchmark | evalscope collection, used by eval-runner too |
| `compare_npu_benchmark.py` | xllm-npu-benchmark | cross-framework comparison |
| `validate_framework_cli.py` | xllm-npu-benchmark | CLI validation, multi-skill |

Skill-specific scripts stay in their skill's `scripts/` subdirectory:
- profiler: analyze_xllm_npu_profile.py, render_triage_npu.py, run_profiling.sh, multibatch_test.py
- pipeline-analysis: create_pipeline_artifacts.py
- eval-runner: run.sh, eval_perf.sh, eval_acc.sh

### 6. .gitignore Update

Add `code/` and `runs/` to block source code mount and run logs from entering the repo.

### 7. humanize/ README Update

Add usage description, write-back rules, and note that directory starts empty and fills organically.

### 8. patches/ Deletion

Empty directory with only a README. Target architecture has no equivalent. Delete entirely.

### 9. baseline/ Creation

New directory with:
- `xllm-benchmark.md` (migrated from references/)
- `README.md` (purpose description)

### 10. tests/ Path Updates

- `test_repository_hygiene.py` — Update path references: `references/` → `reference/`
- `test_model_pr_history.py` — Update dossier path: `model-pr-optimization-history/xllm/` → `reference/pr_history/`

### 11. Skill SKILL.md Path Updates

All 11 skills need global reference path updates:

| Old path | New path |
|----------|----------|
| `../../references/npu-specs.json` | `../../config.json` (static.npu_specs) or `../../reference/knowledge/npu-specs.json` |
| `../../references/accuracy-artifact-schema.md` | `../../reference/io_specs/accuracy-artifact-schema.md` |
| `../../references/perf-artifact-schema.md` | `../../reference/io_specs/perf-artifact-schema.md` |
| `../../references/profiling-artifact-schema.md` | `../../reference/io_specs/profiling-artifact-schema.md` |
| `../../references/run-manifest-template.md` | `../../reference/io_specs/run-manifest-template.md` |
| `../../references/custom-code-style.md` | `../../reference/code-style/custom-code-style.md` |
| `../../references/xllm-benchmark.md` | `../../baseline/xllm-benchmark.md` |
| `../../model-pr-optimization-history/SKILL.md` | `../../reference/pr_history/SKILL.md` |

Skill-local `references/` paths stay unchanged.

### 12. README.md / README_zh.md Updates

- Repository Layout section rewritten to match target structure
- All path references updated (references/ → reference/knowledge/, etc.)
- New directories added (config.json, baseline/, scripts/)
- Removed directories deleted (patches/, kernel-pilot/, model-pr-optimization-history/)
- Prompt routing stays in README (human-facing), not in Agent.md

## Items Deleted

| Item | Reason |
|------|--------|
| `INSTRUCTIONS.md` | Merged into Agent.md |
| `references/` (entire directory) | Split into reference/ subdirectories |
| `model-pr-optimization-history/` | Dossiers → reference/pr_history/, query.py → scripts/ |
| `kernel-pilot/` | Deleted (workflow covered by op-migration skill) |
| `patches/` | Empty, no target architecture equivalent |
| `model-config-index.json` | Placeholder with no content |

## Verification

- Run `tests/test_repository_hygiene.py` after each structural change
- Run `tests/test_model_pr_history.py` after dossier migration
- Verify all skill SKILL.md references resolve to correct new paths
- Verify Agent.md contains all content from AGENTS.md + INSTRUCTIONS.md + CLAUDE.md with no loss
- Verify config.json static.npu_specs matches original npu-specs.json content