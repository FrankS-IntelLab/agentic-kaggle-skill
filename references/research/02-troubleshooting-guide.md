# Troubleshooting Guide

Common issues and their solutions from real competition experience.

---

## Submission Errors

### 400 Bad Request

**Symptoms**: `kaggle competitions submit` returns 400 error

**Possible causes**:
1. Wrong submission format
2. Missing header or wrong column names
3. IDs don't match test set
4. Competition requires .zip format
5. Competition requires model submission, not answer submission

**Solutions**:

```bash
# 1. Check submission format
head submission.csv

# 2. Try .zip format
zip submission.zip submission.csv
kaggle competitions submit <name> -f submission.zip -m "Solution"

# 3. Check top notebooks for expected format
kaggle kernels list --competition <name> --sort-by hotness
```

**Detection**: Look at top notebooks. If they train models and submit .zip with model files, it's a model submission competition.

---

### Zip Submission Mistake

**Common error**: Zipping everything in the folder

```bash
# ❌ WRONG - includes notebook file
zip -m submission.zip *

# ✅ CORRECT - only the CSV
zip submission.zip submission.csv
```

**Python version**:
```python
import zipfile
with zipfile.ZipFile('submission.zip', 'w', zipfile.ZIP_DEFLATED) as zf:
    zf.write('submission.csv', 'submission.csv')  # Only the CSV!
```

---

### Submission Shows ERROR Status

**Possible causes**:
1. Runtime exceeded
2. Memory limit exceeded
3. Missing dependencies
4. Code error

**Debug**:
```bash
# Get kernel output
kaggle kernels output <username>/<kernel-name> -p ./output/

# Check log file
cat ./output/*.log
```

---

## Kernel Issues

### Data Not Attached

**Symptoms**: `FileNotFoundError` for competition data, `/kaggle/input/` is empty

**Causes**:
- Kernel pushed via CLI may not auto-attach data
- `competition_sources` in metadata not working

**Solutions**:

1. **Fork a working kernel first** (recommended):
   - Go to Kaggle website
   - Fork an existing competition notebook
   - Push updates to your fork

2. **Add dataset explicitly**:
   ```json
   {
     "dataset_sources": ["competitions/<competition-name>"],
     "competition_sources": ["<competition-name>"]
   }
   ```

3. **Manual upload fallback**:
   - Generate submission locally
   - Upload via Kaggle website

---

### Kernel Runs But No Submission Generated

**Causes**:
- Ran in Run mode, not Commit mode
- Test set not mounted
- Fallback logic activated

**Solutions**:

1. Use "Save & Run All" on Kaggle website
2. Wait for commit to complete
3. Check if submission auto-generated in `kaggle competitions submissions`

---

### Wrong Predictions (Training IDs in Submission)

**Symptoms**: Submission contains training file IDs

**Cause**: Debug fallback to training data

**Fix**:
```python
# ❌ WRONG
test_files = sorted(TEST_DIR.glob("*.ogg"))
if not test_files:
    test_files = sorted(TRAIN_DIR.glob("*.ogg"))[:10]  # Bad fallback!

# ✅ CORRECT
test_files = sorted(TEST_DIR.glob("*.ogg"))
if not test_files:
    submission = pd.read_csv("sample_submission.csv")
    submission.iloc[:, 1:] = 0.0
    submission.to_csv("submission.csv", index=False)
```

---

## Score Issues

### Score Lower Than Expected

**Checklist**:
- [ ] Waited 4+ hours for stabilization?
- [ ] Using correct metric?
- [ ] Predictions in correct format?
- [ ] Post-processing applied correctly?

### Score Higher Than Expected (Suspicious)

**Don't celebrate yet!**
- Early scores are inflated
- Wait 4+ hours
- Compare against baseline

### Score Dropped After Code Change

**Debug**:
1. Check if change was actually deployed
2. Verify kernel completed successfully
3. Look at kernel output for errors
4. Compare submission files

---

## GPU Issues

### CUDA Error: No Kernel Image

**Error**: `CUDA error: no kernel image is available for execution on the device`

**Cause**: PyTorch compiled for different GPU architecture

**Fix**:
```python
# Use 4-bit quantization instead of FP16
model = AutoModelForCausalLM.from_pretrained(
    model_name,
    load_in_4bit=True,
    device_map="auto",
)
```

### Out of Memory

**Solutions**:
1. Reduce batch size
2. Use gradient checkpointing
3. Use quantization (4-bit, 8-bit)
4. Process in chunks

---

## File Path Issues

### Double Extension

**Symptoms**: `FileNotFoundError` for `file.ogg.ogg`

**Cause**: Filename column already includes extension

**Fix**:
```python
# Check if extension already present
print(train_df['filename'].head())

# If it shows "path/file.ogg", use as-is
audio_file = train_audio_path / filename  # Not filename + ".ogg"
```

---

## Dependency Issues

### Internet Disabled, Can't Install Package

**Solution**: Find Kaggle dataset with wheel file

```python
import subprocess
import sys
from pathlib import Path

wheel_files = list(Path('/kaggle/input/dataset-name').glob('*.whl'))
if wheel_files:
    subprocess.check_call([sys.executable, '-m', 'pip', 'install', str(wheel_files[0]), '--quiet'])
```

---

## Silent Failures

### Feature Extraction Returns 0 Samples

**Cause**: Silent exception catching

**Fix**:
```python
# ❌ BAD - hides errors
try:
    audio, sr = load_audio(file)
    emb = extract_embedding(audio, sr)
except Exception:
    continue

# ✅ BETTER - logs failures
try:
    audio, sr = load_audio(file)
    emb = extract_embedding(audio, sr)
except Exception as e:
    print(f"Failed on {file}: {type(e).__name__}: {e}")
    continue
```

---

## API Rate Limits

### Rate Limit Pattern

**Pattern**:
- Some APIs hit limits during peak hours
- Limits clear in off-peak hours

**Solution**: Switch to backup provider during limit window

---

## Quick Diagnostic Commands

```bash
# Check submission status
kaggle competitions submissions <name>

# Check kernel status
kaggle kernels status <username>/<kernel-name>

# Get kernel output
kaggle kernels output <username>/<kernel-name> -p ./output/

# List competition files
kaggle competitions files <name>

# View leaderboard
kaggle competitions leaderboard <name> -s
```
