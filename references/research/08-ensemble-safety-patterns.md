# Ensemble Safety Patterns

> *When combining multiple solutions helps — and when it actively hurts your score.*

---## Compliance Note

Use these patterns only with competition-permitted data, code, models, notebooks, and external resources. Do not privately share competition code, data, predictions, prompts, logs, or results outside your Kaggle team. Do not tune directly against the public leaderboard through repeated submissions; use cross-validation and local holdout validation instead. Always check the specific competition rules before applying these workflows.



## The Ensemble Trap

Combining multiple solutions (blending, stacking, averaging) is standard advice in competitions. But ensembles can **degrade** your score under specific conditions. Understanding when to blend — and when not to — is a critical competitive skill.

---

## When Ensemble Helps

Ensemble works best when the **errors of your sources are uncorrelated**. If Source A is wrong on different samples than Source B, combining them cancels out individual errors.

### Conditions for Effective Ensemble

| Condition | Why It Helps |
|-----------|-------------|
| Sources use different algorithms | Different models make different types of errors |
| Sources use different feature sets | Captures different signals in the data |
| Sources trained on different data splits | Reduces overfitting to the same noise |
| Sources use different preprocessing | Handles data quirks differently |

**Rule of thumb**: The more *diverse* your sources, the more value from ensembling.

---

## When Ensemble Hurts

### 1. Correlated Sources

If all your sources make errors on the **same samples**, blending provides no benefit — and the averaging process can amplify shared biases.

```
Source A accuracy: 85%
Source B accuracy: 85%
Both wrong on same 15% → Blended accuracy: still 85%
```

**Detection**: Run your sources on a validation set. If their predictions are >0.90 correlated (Pearson or Spearman), they are too similar to blend effectively.

```python
import numpy as np

corr = np.corrcoef(source_a_preds, source_b_preds)[0, 1]
if abs(corr) > 0.90:
    print("⚠️ Sources are highly correlated — blending unlikely to help")
```

### 2. Dominant Source Masking

When one source is significantly better than others, a simple average **pulls down** the best performer.

```
Source A score: 0.92  (best)
Source B score: 0.80
Simple average:  ~0.86  ← Worse than Source A alone!
```

**Fix**: Use weighted averaging, or simply use the best single source as your baseline.

### 3. Negative Contribution Sources

Some sources have **correctly calibrated confidence** but **wrong predictions**. When averaged with good sources, they dilute confidence without correcting errors.

**Detection Method**:
```python
# Test each source's marginal contribution to the best pair
best_sources = [source_1, source_2]  # Your top two

for candidate in remaining_sources:
    test_blend = weighted_average(best_sources + [candidate], weights)
    if score(test_blend) < score(best_blend):
        print(f"⚠️ {candidate.name} degrades the blend — exclude")
```

**Rule**: If a source hurts every combination it's added to, remove it regardless of its standalone score.

### 4. Overfitting to Public Leaderboard

Optimizing blend weights on the public LB can overfit to the evaluation subset. The private LB may have a different distribution.

**Mitigation**:
- Use cross-validation to tune blend weights, not public LB scores
- Limit the number of blend weight combinations you test
- Reserve a local validation/holdout split from training data for final weight selection

---

## Safe Ensemble Workflow

Follow this step-by-step process before blending:

```
Step 1: Evaluate each source independently
   ├─ Cross-validation score
   ├─ Public LB score (if available)
   └─ Error analysis (where does each source fail?)

Step 2: Check pairwise correlation
   ├─ If correlation > 0.90 between any pair → exclude the weaker one
   └─ Keep only sources with meaningfully different error patterns

Step 3: Start with the best single source
   └─ This is your baseline — blending must beat this

Step 4: Greedy addition
   ├─ Add the next best source
   ├─ Test blend on validation set
   ├─ If score improves → keep it
   └─ If score drops → discard the new source permanently

Step 5: Weight optimization
   ├─ Grid search or Bayesian optimization on blend weights
   └─ Validate on held-out set, NOT public LB

Step 6: Final evaluation
   └─ Only submit if blend beats best single source on validation
```

---

## Correlation Check Toolkit

### Pairwise Correlation Matrix

```python
import pandas as pd
import numpy as np

# predictions: dict of {source_name: array_of_predictions}
pred_df = pd.DataFrame(predictions)
corr_matrix = pred_df.corr()

print("Pairwise prediction correlation:")
print(corr_matrix)

# Flag highly correlated pairs
threshold = 0.90
for i in range(len(corr_matrix.columns)):
    for j in range(i+1, len(corr_matrix.columns)):
        if abs(corr_matrix.iloc[i, j]) > threshold:
            print(f"⚠️  {corr_matrix.columns[i]} ↔ {corr_matrix.columns[j]}: "
                  f"r={corr_matrix.iloc[i, j]:.3f}")
```

### Error Overlap Analysis

```python
# Identify where each source is wrong
errors = {}
for name, preds in predictions.items():
# ground_truth refers to validation labels ONLY - never use hidden test labels
    errors[name] = set(np.where(preds != ground_truth)[0])

# Find overlapping errors
for name_a, name_b in combinations(predictions.keys(), 2):
    overlap = errors[name_a] & errors[name_b]
    overlap_pct = len(overlap) / len(errors[name_a] | errors[name_b])
    print(f"{name_a} ∩ {name_b}: {overlap_pct:.1%} error overlap")
```

**Interpretation**: High overlap means blending won't help. Low overlap means the sources complement each other.

---

## Blending Methods

| Method | When to Use | Complexity |
|--------|-------------|------------|
| Simple average | Sources are diverse, similar quality | Very Low |
| Weighted average | Sources have different quality levels | Low |
| Rank averaging | Metric is rank-based (e.g., AUC) | Low |
| Stacking (meta-learner) | Many sources, enough validation data | Medium |
| Geometric mean | Predictions are probabilities, need calibration | Low |

### When in Doubt, Start Simple

Simple averaging with 2-3 diverse sources often beats complex stacking with 10 correlated sources.

---

## Key Takeaways

1. **Diversity > Count** — Two diverse sources beat five similar ones
2. **Always check correlation first** — High correlation = wasted effort
3. **Test marginal contribution** — Every source must earn its place in the blend
4. **Optimize on validation, not public LB** — Avoid overfitting your blend weights
5. **The best ensemble can't exceed the coverage of its sources** — If no source solves a task subset, blending won't fix that
6. **When blending hurts, simplify** — Sometimes the best strategy is a single strong source