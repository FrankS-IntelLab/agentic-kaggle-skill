# Audio Classification Competition Case Study

Audio classification competition for species identification from sound recordings.

---

## Competition Overview

- **Type**: Audio classification / Bioacoustics
- **Task**: Identify species from audio recordings
- **Evaluation**: Macro-averaged AUC
- **Data**: Training audio, test soundscapes
- **Location**: Specific geographic region

---

## Key Lessons

### 1. Data Path Matters

**Wrong path**:
```
/kaggle/input/competition-name/
```

**Correct path**:
```
/kaggle/input/competitions/competition-name/
```

The `competitions/` subdirectory is easy to miss.

### 2. Prior-Based Prediction Limitation

**Problem**: Using location/time priors didn't improve score

**Cause**: All test samples from same location/time → identical predictions → no ranking differentiation

**Lesson**: Priors help only when there's variation between test samples. Use actual audio features (embeddings) for differentiation.

### 3. Top Notebook Replication

**Workflow**:
```bash
# Pull with metadata
kaggle kernels pull user/top-notebook -p ./solution/ -m

# Edit metadata
{
  "id": "username/kernel-name",
  "is_private": true,
  "enable_internet": false,
  "dataset_sources": [
    "dataset/embeddings-model"
  ],
  "model_sources": ["pretrained-model"]
}

# Push
kaggle kernels push -p ./solution/
```

### 4. Hybrid Approach: Temporal Model + SED

**Best approach**: Ensemble of two techniques
- **Temporal Model**: Temporal modeling with embeddings
- **SED**: Sound Event Detection with CNN

**Ensemble weights**: 60% Temporal + 40% SED (rank-averaging)

**Result**: Top leaderboard position

---

## Troubleshooting Journey

### Issue 1: Silent Feature Extraction Failures

**Problem**: 10,000 samples yielded 0 embeddings

**Cause**: Exception silently caught without logging

**Fix**:
```python
try:
    audio, sr = load_audio(file)
    emb = extract_embedding(audio, sr)
except Exception as e:
    print(f"Failed on {file}: {type(e).__name__}: {e}")
    continue
```

### Issue 2: Double File Extension

**Problem**: `FileNotFoundError` for `file.ogg.ogg`

**Cause**: Filename column already included `.ogg`

**Fix**:
```python
# Check format first
print(train_df['filename'].head())

# Use filename as-is, don't add extension
audio_file = train_audio_path / filename
```

### Issue 3: Training IDs in Submission

**Problem**: Submission contained training file IDs instead of test IDs

**Cause**: Debug fallback to training data when test set was hidden

**Fix**:
```python
test_files = sorted(TEST_DIR.glob("*.ogg"))
if not test_files:
    # Use sample_submission.csv, NOT training data
    submission = pd.read_csv("sample_submission.csv")
    submission.iloc[:, 1:] = 0.0
    submission.to_csv("submission.csv", index=False)
```

---

## Technical Approach

### Embedding Extraction

```python
# Load pretrained model
model = load_pretrained_model('/kaggle/input/embeddings-model/')

# Extract embeddings
def get_embedding(audio_path):
    audio, sr = load_audio(audio_path)
    embedding = model.embed(audio)
    return embedding
```

### SED Ensemble

```python
# SED inference with TTA
def sed_inference(audio_path, model, tta_shifts=3):
    predictions = []
    for shift in range(tta_shifts):
        pred = model.predict(audio_path, time_shift=shift)
        predictions.append(pred)
    return np.mean(predictions, axis=0)
```

### Rank Averaging

```python
# Ensemble with rank averaging
from scipy.stats import rankdata

def rank_average(preds1, preds2, weights=[0.6, 0.4]):
    rank1 = rankdata(preds1) / len(preds1)
    rank2 = rankdata(preds2) / len(preds2)
    return weights[0] * rank1 + weights[1] * rank2
```

---

## Results

| Version | Approach | CV Score | LB Score |
|---------|----------|----------|----------|
| V1 | Uniform baseline | — | 0.500 |
| V5 | Location priors | — | 0.500 (no improvement) |
| V10 | Embeddings | 0.910 | 0.910 |
| V15 | Temporal model | 0.920 | 0.923 |
| V19 | Temporal + tuning | 0.925 | 0.927 |
| V21 | Temporal + SED ensemble | 0.930 | 0.935 |

---

## Key Takeaways

1. **Use actual features, not just priors** — priors don't help when all samples are similar
2. **Check file path format** — filename column may already include extensions
3. **Never use training data as fallback** — use sample_submission.csv instead
4. **Ensemble complementary approaches** — Temporal + SED worked better than either alone
5. **Replicate top notebooks** — pull with `-m` flag to get all dependencies
