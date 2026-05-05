# Advanced Competition Patterns

> *Patterns distilled from autonomous agent experimentation on Kaggle competitions.*

---

## 1. State Machine Execution Loop

A state-driven pattern for autonomous competition agents using cron-based execution.

### State Machine Architecture

```
┌─────────────────────────────────────────────────┐
│              Cron Trigger (every N min)          │
└─────────────────────────────────────────────────┘
                        │
                        ▼
┌─────────────────────────────────────────────────┐
│              Read /workspace/state.json          │
└─────────────────────────────────────────────────┘
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼
       ┌──────────┐         ┌──────────┐
       │ SUPERVISOR│         │  WORKER  │
       └──────────┘         └──────────┘
              │                   │
              │  instruction      │  execution
              ▼                   ▼
       ┌─────────────────────────────────┐
       │     Update /workspace/state.json │
       └─────────────────────────────────┘
```

### Stage Definitions

| Stage | Description | Next Action |
|-------|-------------|-------------|
| `idle` | No active work | Upload notebook |
| `uploaded` | Notebook pushed to platform | Run notebook |
| `running` | Notebook executing | Check status |
| `completed` | Notebook finished | Collect results |
| `evaluated` | Results collected | Submit |
| `submitted` | Submission done | Reset or continue |
| `failed` | Error occurred | Recovery mode |

### State File Format

```json
{
  "mode": "supervisor|worker",
  "stage": "idle|uploaded|running|completed|evaluated|submitted|failed",
  "instruction": "current_task",
  "instruction_id": 0,
  "last_completed_instruction_id": 0,
  "status": "idle|pending|done|failed",
  "run_id": "run_20260505_120000",
  "last_update": "2026-05-05T12:00:00Z",
  "best_score": 0.0,
  "target_score": 0.0
}
```

### One Step Per Run Rule

**Critical**: Execute ONLY ONE step per cron trigger.

- DO NOT wait or block
- DO NOT simulate results  
- Each run reads state, acts once, updates state, stops
- This prevents timeout issues and enables fault tolerance

---

## 2. Config-Driven Cronjob Design

**Never hardcode resource identifiers in cronjob prompts.**

### The Problem

| Hardcoded ❌ | Config-Driven ✅ |
|-------------|------------------|
| Kernel name embedded in prompt | Kernel name in config file |
| Must recreate cronjob to switch | Update config → cronjob adapts |
| Multiple cronjobs for experiments | Single cronjob handles all |

### Solution Pattern

```
┌─────────────────────────────────────┐
│  current_experiment.json (CONFIG)   │  ← Single source of truth
│  - current_kernel: "user/kernel-v2" │
│  - expected_score: 0.95             │
│  - phase: "monitoring"              │
└─────────────────────────────────────┘
                │
                ▼
┌─────────────────────────────────────┐
│  Cronjob (every N minutes)          │
│  1. READ config file                │
│  2. Monitor <current_kernel>        │
│  3. Take action based on phase      │
│  4. UPDATE config file              │
└─────────────────────────────────────┘
```

### Implementation

**Config file:**
```json
{
  "current_kernel": "username/kernel-name",
  "experiment_name": "v2-weight-test",
  "expected_score": 0.95,
  "phase": "monitoring"
}
```

**Helper script:**
```python
# switch_experiment.py
import json

def switch_experiment(kernel, name, expected_score):
    config = {
        "current_kernel": kernel,
        "experiment_name": name,
        "expected_score": expected_score,
        "phase": "monitoring"
    }
    with open("current_experiment.json", "w") as f:
        json.dump(config, f, indent=2)

# Switch experiments without touching cronjob
switch_experiment("username/v2", "v2-test", 0.95)
```

---

## 3. Supervisor Validation Duties

The Supervisor MUST validate Worker outputs before accepting.

### Validation Checklist

| Check | When | Why |
|-------|------|-----|
| `run_id` exists | Before submit | Ensures kernel actually ran |
| Output timestamp recent | After collect | Ensures new generation |
| Output logs exist | After collect | Ensures expected code ran |
| File size reasonable | After collect | Detects empty/invalid outputs |

### Rejection Rules

**IF validation fails:**
- Reject the output
- Reset to idle  
- Report failure to user

**Why this matters:** Workers can shortcut by:
- Downloading old output files
- Skipping validation logic
- Copying from other sources

The Supervisor's job is to CATCH these violations.

---

## 4. Rank Averaging for AUC Competitions

When leaderboard metric is AUC (ranking-based), use rank averaging instead of raw probability averaging.

### The Pattern

```python
import pandas as pd

# ❌ WRONG: Raw probability averaging
blended = 0.6 * prob_a + 0.4 * prob_b

# ✅ CORRECT: Rank averaging (for AUC metrics)
ra = pd.DataFrame(prob_a).rank(axis=0, pct=True)
rb = pd.DataFrame(prob_b).rank(axis=0, pct=True)
blended = 0.6 * ra + 0.4 * rb
```

### Why This Works

- AUC measures ranking quality, not probability calibration
- Rank averaging is robust to scale differences between models
- Expected gain: +0.001–0.005 over raw averaging

### Weight Optimization Strategy

After finding best blend ratio, test variations:
- If 60/40 is best, test: [58/42, 60/40, 62/38]
- Expected gain: +0.001–0.002
- Use cronjob to automate weight sweep experiments

---

## 5. ONNX Model Validation

### Forbidden Operations Check

```python
FORBIDDEN_OPS = {"LOOP", "SCAN", "NONZERO", "UNIQUE", "SCRIPT", "FUNCTION", "COMPRESS"}

for node in model.graph.node:
    if node.op_type.upper() in FORBIDDEN_OPS:
        return -1  # Reject model
```

### Pre-Submission Validation

Before any submission, validate ALL models:

```python
import onnx
from pathlib import Path

errors = []
for i in range(TASK_COUNT):
    try:
        m = onnx.load(f"task{i:03d}.onnx")
        onnx.checker.check_model(m)
    except Exception as e:
        errors.append((i, str(e)[:100]))

if errors:
    print(f"Found {len(errors)} invalid models:")
    for task_id, error in errors:
        print(f"  task{task_id:03d}: {error}")
else:
    print("✓ All models valid!")
```

### Task-Specific Error Debugging

When you see: **"Error processing onnx networks for tasks: [X]"**

| Probability | Cause | Description |
|-------------|-------|-------------|
| 80% | Model export diverged | Different preprocessing, failed retrain |
| 15% | Corrupted/truncated file | File exists but incomplete |
| 5% | Input shape mismatch | Task-specific input differs |

### Debug Steps

1. Verify model exists in ZIP: `unzip -l submission.zip | grep task_`
2. Validate ONNX structure locally
3. Compare with working tasks (opset version, input names/shapes, model size)
4. Replace corrupted models with backups

---

## 6. Provenance Verification

A submission is ONLY valid if it has verifiable provenance.

### Required Evidence

| Evidence | How to Verify |
|----------|---------------|
| `run_id` | Check state file for non-null value |
| Kernel status | API must return COMPLETE |
| Output files | Output download must succeed |
| Submission linkage | Description must reference valid run |

### Fraud Detection Rules

**IF submission claims "generated by notebook" but:**

| Condition | Verdict | Action |
|-----------|---------|--------|
| No `run_id` | 🚫 FRAUDULENT | Reject immediately |
| Kernel status = ERROR | 🚫 INVALID | Reject, require fix |
| No output files | 🚫 INVALID | Reject, require re-run |
| State says "completed" but kernel shows ERROR | 🚫 FRAUD | Block Worker |

### Verification Commands

```bash
# Check kernel status
kaggle kernels status <username>/<kernel-name>

# Download outputs (verifies files exist)
kaggle kernels output <username>/<kernel-name> -p /tmp/output/

# Check submissions
kaggle competitions submissions -c <competition> | head -5
```

---

## 7. Dependency Vendoring

When a required package cannot be installed via internet on the competition platform.

### Pattern

1. **Prepare dependency locally:**
   ```bash
   pip download <package> -d ./packages/
   ```

2. **Create a platform dataset:**
   - Upload the .whl or package files

3. **Attach dataset to notebook:**
   - Add as input in kernel-metadata.json

4. **Install from dataset path:**
   ```python
   !pip install /kaggle/input/<dataset-name>/<package>-*.whl
   ```

5. **Verify import works:**
   ```python
   import <package>
   print(<package>.__version__)
   ```

### When to Use

- Competition disables internet access
- Required package not in default environment
- Custom package version needed

---

## 8. Remote Compute Only Rule

### The Principle

All experiments MUST be executed on the competition platform (Kaggle), not locally.

**Local environment is STRICTLY LIMITED to:**
- Code writing
- Notebook preparation  
- Lightweight validation (optional, not authoritative)

**Local execution is NOT valid for:**
- Scoring
- Evaluation
- Submission decisions

### Mandatory Execution Flow

1. Worker writes code/notebook locally
2. Worker uploads notebook to platform
3. Worker triggers platform run
4. Worker WAITS for completion
5. Worker retrieves outputs from platform
6. Worker extracts metrics from platform results

**ONLY platform results are considered valid.**

---

## Summary: Key Takeaways

| Pattern | Value | When to Use |
|---------|-------|-------------|
| State Machine | Fault-tolerant autonomous loops | Cron-based automation |
| Config-Driven Cronjob | Swappable experiments | Multiple experiments, one cronjob |
| Supervisor Validation | Prevent fake results | Worker/Supervisor pattern |
| Rank Averaging | +0.001–0.005 improvement | AUC competitions |
| ONNX Validation | Prevent runtime errors | Model submission competitions |
| Provenance Verification | Trust but verify | Any submission workflow |
| Dependency Vendoring | Offline installation | No-internet competitions |
| Remote Compute Only | Authoritative results | All competitions |
