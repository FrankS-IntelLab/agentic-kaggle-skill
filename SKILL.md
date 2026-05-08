---
name: agentic-data-science-competition
description: |
  AI Agent-driven Kaggle competition workflow. Learn from real competition experience:
  score stabilization patterns, submission troubleshooting, kernel workflows, GPU task delegation,
  and the spec-driven development approach that achieved top leaderboard positions.
  Use when: working on any Kaggle competition, analyzing submission failures, setting up
  automated pipelines, or replicating top notebook solutions.
version: 1.0.0
author: Frank S (IntelLab)
tags: [kaggle, competition, ml, agents, automation, data-science, leaderboard]
---

# Agentic Data Science Competition

> *「The agent doesn't just submit — it learns from failures, adapts strategies, and iterates autonomously.」*

## Core Philosophy

This skill distills practical patterns from real competition experience:

1. **Agents as teammates** — Not just tools, but collaborators that can research, debug, and iterate
2. **Spec-driven development** — Document before coding, delegate with clear constraints
3. **Fail fast, learn faster** — Early scores are misleading; systematic debugging wins
4. **Automation where it matters** — Cronjobs for monitoring, delegation for complex work

---

## Quick Reference

### When to Use This Skill

| Trigger | Action |
|---------|--------|
| Starting a new competition | → Read "Project Setup" section |
| Submission returns 400 error | → Check "Troubleshooting" section |
| Score dropped unexpectedly | → Read "Score Stabilization" section |
| Need to replicate top notebook | → Use "Replication Workflow" |
| Kernel push fails | → Check "Kernel Workflow" section |
| GPU required but unavailable | → Use "Delegation Strategy" |

### Key Commands

```bash
# Submit to competition
kaggle competitions submit <name> -f <file> -m "<message>"

# Check submission status
kaggle competitions submissions <name>

# Pull top notebook WITH metadata
kaggle kernels pull <owner>/<kernel> -p ./path/ -m

# Push kernel to Kaggle
kaggle kernels push -p ./path/

# Monitor kernel status
kaggle kernels status <username>/<kernel-name>
```

---

## Score Stabilization Pattern

**Critical insight**: Kaggle scores take time to stabilize after submission.

|| Time | Score Behavior | What To Do ||
|------|----------------|------------||
| Start | Baseline | Submit early to start evaluation ||
| +2 hours | Peak (inflated) | **Don't trust!** Often artificially high ||
| +4 hours | Stabilized | **True score** — make decisions now ||

**Lesson**: Never celebrate early highs. Wait 4+ hours before judging performance.

---

## Submission Troubleshooting

### 400 Bad Request Error

1. Check submission format (with/without header, quotes)
2. Verify IDs match test set exactly
3. **Try .zip format** — some competitions require zipping the CSV
4. Check if competition requires **model submission** vs answer submission

### Zip Submission Format (Critical!)

**Common mistake**: Zipping everything in the folder (including `__notebook__.ipynb`)

**Correct way**:
```python
import zipfile
with zipfile.ZipFile('submission.zip', 'w', zipfile.ZIP_DEFLATED) as zf:
    zf.write('submission.csv', 'submission.csv')  # Only the CSV!
```

### Competition Types

| Type | What You Submit | Examples |
|------|-----------------|----------|
| **Answer Submission** | CSV with predictions | Most competitions |
| **Model Submission** | Trained model weights (LoRA, checkpoints) | Some LLM competitions |

**Detection**: Look at top notebooks — do they train models or just generate predictions?

---

## Kernel Workflow

### Run Mode vs Commit Mode

| Mode | Test Set | Use Case |
|------|----------|----------|
| **Run** | Hidden | Development, debugging |
| **Commit** ("Save & Run All") | Mounted | Production, final submission |

**Why this matters**: `kaggle kernels push` runs in Run mode. Test set is NOT mounted. Use sample_submission.csv for placeholder.

### Data Path Pattern

```
/kaggle/input/competitions/<competition-name>/   ← Correct!
NOT /kaggle/input/<competition-name>/            ← Wrong!
```

### Kernel Metadata Best Practices

```json
{
  "id": "username/kernel-name",
  "is_private": true,          // ← Always true by default!
  "enable_internet": false,    // ← Check competition rules
  "competition_sources": ["competition-name"],
  "dataset_sources": ["dataset-with-dependencies"]
}
```

---

## Replicating Top Notebooks

### Workflow

```bash
# 1. Pull with metadata (-m flag is critical!)
kaggle kernels pull <owner>/<kernel> -p ./solution/ -m

# 2. Edit kernel-metadata.json
#    - Change "id" to your username/new-name
#    - KEEP all dataset_sources, model_sources, kernel_sources

# 3. Push
kaggle kernels push -p ./solution/

# 4. Monitor
kaggle kernels status <your-username>/<new-kernel-name>
```

### Critical Points

1. **Always use `-m` flag** — gets kernel-metadata.json with dependencies
2. **Preserve ALL dependencies** — dataset_sources, model_sources, kernel_sources
3. **Only change id and title** — everything else should match original
4. **Check enable_internet** — if original has false, keep it false

### When to Delegate

**Delegate to OpenCode/Claude Code** when:
- Adapting solutions to different contexts
- Combining multiple techniques
- Integrating with existing codebase
- Complex transformation or refactoring

**Don't delegate** when:
- Simple file downloads
- Metadata-only changes
- Direct forks without modifications

---

## Spec-Driven Development

### Process

1. **Document SPEC.md first** before coding:
   - Competition overview (task, data, evaluation)
   - Top approaches from notebooks
   - Strategy and implementation plan
   - Technical constraints
   - Success metrics and timeline

2. **Delegate to AI coding agents** with the spec:
   - Provide SPEC.md as context
   - Specify constraints (time limit, GPU availability)
   - Set clear success criteria

3. **Iterate based on results**

### SPEC.md Template

```markdown
# SPEC.md - Competition Name

## 1. Competition Overview
- Task, deadline, prize, current top score

## 2. Task Definition
- Input/Output format, data summary

## 3. Top Approaches (from Kaggle Notebooks)
- Approach A: [votes/score], strategy
- Approach B: [votes/score], strategy

## 4. Strategy & Implementation Plan
- Phase 1: Quick baseline
- Phase 2: High-ROI improvements
- Phase 3: Fine-tuning

## 5. Technical Constraints
- Submission format, time limits, GPU requirements

## 6. Success Metrics
| Milestone | Target | Timeline |
|-----------|--------|----------|
| Baseline | 0.xx | Day 1 |
| Improved | 0.xx | Day 2 |
```

---

## Delegation Strategy

### Local vs Kaggle

| Task | Where | Why |
|------|-------|-----|
| Data analysis | Local | Fast iteration, no GPU needed |
| Prompt engineering | Local | Quick testing |
| Model training | Kaggle | Free GPU |
| Large-scale inference | Kaggle | GPU + internet |

### When to Use Which Agent

| Agent | Best For |
|-------|----------|
| OpenCode | Code development, feature implementation |
| Claude Code | PR review, complex debugging |
| Subagents (delegate_task) | Parallel research, isolated workstreams |

---

## Common Pitfalls

### 1. Debug Fallback to Training Data

**Wrong**:
```python
test_files = sorted(TEST_DIR.glob("*.ogg"))
if not test_files:
    test_files = sorted((COMP_DIR / "train_soundscapes").glob("*.ogg"))[:10]
```

**Correct**:
```python
if not test_files:
    submission = pd.read_csv("sample_submission.csv")
    submission.iloc[:, 1:] = 0.0
    submission.to_csv("submission.csv", index=False)
```

### 2. Game AI Competitions Require Comprehensive Features

**Pattern observed**: Top-performing game agents often require comprehensive feature implementations.

**Takeaway**: Don't oversimplify game AI agents. Study top solutions to understand the full feature set needed to compete.

### 3. Double File Extension

**Check if filename column already includes extension**:
```python
# Check this:
print(train_df['filename'].head())

# If it shows "1161364/iNat1216197.ogg", don't add .ogg again!
```

### 4. Silent Exception Catching

**Bad** (hides errors):
```python
try:
    audio, sr = load_audio(file)
except Exception:
    continue  # No idea why it failed!
```

**Better**:
```python
try:
    audio, sr = load_audio(file)
except Exception as e:
    print(f"Failed on {file}: {type(e).__name__}: {e}")
    continue
```

---

## API Usage Patterns

### Rate Limit Pattern

- Some AI APIs hit limits during peak hours (e.g., 10:00-17:00 Beijing time)
- **Solution**: Switch to backup model/provider during limit window
- **Recovery**: Limits clear in evening hours

**Plan around this**: Check time before delegating to AI coding agents.

---

## Verification Checklist

After submitting:
- [ ] Check submission shows in `kaggle competitions submissions`
- [ ] Monitor status: PENDING → RUNNING → COMPLETE
- [ ] Wait 4+ hours before evaluating final score
- [ ] Compare against baseline and target

---

## Advanced Patterns

See [references/research/04-advanced-patterns.md](references/research/04-advanced-patterns.md) for:

- **State Machine Execution Loop** — Fault-tolerant autonomous agent pattern
- **Config-Driven Cronjob Design** — Swappable experiments without recreating jobs
- **Supervisor Validation Duties** — Fraud detection and provenance verification
- **Rank Averaging for AUC** — +0.001–0.005 improvement technique
- **ONNX Model Validation** — Forbidden ops and error debugging
- **Dependency Vendoring** — Offline package installation

See [references/research/05-environment-setup-patterns.md](references/research/05-environment-setup-patterns.md) for:

- **Kaggle Environments Source Install** — Install latest envs from GitHub when PyPI is outdated

See [references/research/06-competition-types-submission-workflow.md](references/research/06-competition-types-submission-workflow.md) for:

- **Competition Type Taxonomy** — Prediction/Adapter/ONNX and Pure Code/Mixed/Standard
- **The `-k -v` Flag Requirement** — Prevent orphan submissions with notebook linkage

---

## Related Skills

| Skill | Purpose |
|-------|---------|
| `agentic-competition-workflow` | Git-first project management, validation pipelines, full competition lifecycle |
| `kaggle-auto-submit` | End-to-end automation with cronjob monitoring |
| `autonomous-iteration` | ANALYSIS → BUILD → EXPERIMENT → REVIEW loops |
| `opencode` | Delegate coding to OpenCode CLI |
| `claude-code` | Delegate coding to Claude Code CLI |
