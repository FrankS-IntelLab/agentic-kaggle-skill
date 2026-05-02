# Competition Patterns

Research notes on patterns observed across multiple Kaggle competitions.

---

## Score Stabilization Pattern

### The Problem

Kaggle scores are not instant — they take time to stabilize due to:
- Parallel evaluation infrastructure
- Running battles against other agents (game competitions)
- Averaging over multiple episodes

### Observed Pattern

| Time Since Submission | Score | Interpretation |
|-----------------------|-------|----------------|
| 0-2 hours | Peak (inflated) | **Don't trust!** Favorable conditions |
| 2-4 hours | Regressing | Moving toward true value |
| 4+ hours | Stabilized | **True competitive score** |

### Why Early Scores Are Inflated

1. **Lucky matchups**: Initial evaluations may be against weaker opponents
2. **Small sample size**: Few episodes = high variance
3. **Rating system inertia**: ELO-like systems need time to converge

### Practical Implication

- Submit early to start evaluation
- Don't make decisions based on < 4 hour scores
- If score drops from peak, that's normal — not a bug

---

## Competition Type Taxonomy

### Answer Submission Competitions

**What you submit**: CSV with predictions

**Typical workflow**:
1. Download data
2. Build model locally or on Kaggle
3. Generate predictions
4. Submit CSV

**Examples**: Most tabular, vision, NLP competitions

### Model Submission Competitions

**What you submit**: Trained model weights

**Typical workflow**:
1. Download training data
2. Fine-tune a base model
3. Save LoRA adapter or checkpoint
4. Submit model weights (often as .zip)

**Examples**: Some LLM reasoning challenges

**Detection**: Look at top notebooks — do they train models?

### Key Difference

| Aspect | Answer Submission | Model Submission |
|--------|-------------------|------------------|
| CLI command | `kaggle competitions submit -f submission.csv` | Submit .zip with model files |
| 400 error | Check CSV format | Check if model files required |
| GPU needed | Maybe for training | Usually yes for fine-tuning |

---

## Kernel Execution Modes

### Run Mode (Default)

- Triggered by: `kaggle kernels push`
- Test set: **Hidden**
- Use case: Development, debugging
- Submission validity: ❌ Invalid (uses fallback logic)

### Commit Mode ("Save & Run All")

- Triggered by: Kaggle website "Save & Run All" button
- Test set: **Mounted**
- Use case: Production, final submission
- Submission validity: ✅ Valid (processes actual test set)

### Critical Insight

When you `kaggle kernels push`, the kernel runs in Run mode. The hidden test set is NOT mounted at `/kaggle/input/competitions/<name>/test/`.

**Any fallback logic that uses training data will activate, creating invalid submissions.**

**Correct pattern**:
```python
test_files = sorted(TEST_DIR.glob("*.ogg"))
if not test_files:
    # Use sample_submission.csv for placeholder
    submission = pd.read_csv("sample_submission.csv")
    submission.iloc[:, 1:] = 0.0
    submission.to_csv("submission.csv", index=False)
else:
    # Normal inference
    ...
```

---

## Data Path Patterns

### Kaggle Notebook Paths

```
/kaggle/input/competitions/<competition-name>/     ← Correct!
/kaggle/input/<competition-name>/                  ← Wrong!
```

The `competitions/` subdirectory is easy to miss.

### Debug Technique

Always add a cell to check data availability:
```python
import os
for root, dirs, files in os.walk('/kaggle/input'):
    print(root, files[:5])
```

If empty, the kernel has no data attached regardless of metadata settings.

---

## Top Notebook Analysis Patterns

### What to Look For

1. **Submission format**: CSV? .zip? Model weights?
2. **Dependencies**: What datasets/models are attached?
3. **Internet setting**: enabled or disabled?
4. **GPU usage**: Required? Optional?
5. **Key techniques**: What's the secret sauce?

### Extracting Techniques

When analyzing top notebooks:
1. Pull with `-m` flag to get metadata
2. Convert to Python: `jupytext --to py:percent notebook.ipynb`
3. Identify key functions and patterns
4. Document in SPEC.md for replication

---

## Submission Quota Management

### Daily Limits

Some competitions have daily submission limits.

### Strategy

1. Use test notebooks for validation (doesn't count against quota)
2. Submit only when confident
3. Monitor submission status before resubmitting

### Automated Monitoring

Use cronjobs to:
- Check kernel completion
- Auto-submit when ready
- Report results

---

## Competition Lifecycle

### Phase 1: Exploration (Day 1-2)

- Understand competition mechanics
- Download and analyze data
- Study top notebooks
- Create SPEC.md

### Phase 2: Baseline (Day 2-3)

- Implement quick baseline
- Submit to get on leaderboard
- Identify gaps

### Phase 3: Iteration (Day 3+)

- Improve incrementally
- Test changes systematically
- Monitor score stabilization

### Phase 4: Final Push (Last Week)

- Ensemble approaches
- Fine-tune hyperparameters
- Final submissions

---

## Key Metrics to Track

| Metric | Why It Matters |
|--------|----------------|
| Best public score | Target to beat |
| Submission count | Quota management |
| Kernel runtime | Time budget |
| GPU hours | Resource planning |
| Dependencies | Reproducibility |
