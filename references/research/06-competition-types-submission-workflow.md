# Competition Types & Submission Workflow

> *Understanding competition types and proper submission workflow prevents common failures.*

---

## Competition Type Taxonomy

### By Submission Format

| Type | What You Submit | Examples |
|------|-----------------|----------|
| **Prediction-Based** | CSV with predictions | Most ML competitions |
| **Adapter-Based** | LoRA adapter weights | LLM fine-tuning challenges |
| **ONNX-Based** | ONNX model files | Neural network optimization |

### By Submission Behavior

| Type | Direct File Upload | Kernel Execution Required |
|------|-------------------|--------------------------|
| **Pure Code Competition** | ❌ ERROR | ✅ Required |
| **Mixed Competition** | ✅ Works | ✅ Also works |
| **Standard Competition** | ✅ Works | ✅ Optional |

### Discovery Method

**Step 1: Try direct submission first (fastest if it works)**
```bash
# Download kernel output
kaggle kernels output user/kernel -p ./output/

# Try direct submission
kaggle competitions submit -c competition-name -f ./output/submission.zip -m "test"
```

**Step 2: Interpret results**
- ✅ **COMPLETE + Score** → Mixed/Standard competition
- ❌ **ERROR** → Pure code competition → Use kernel workflow

**Time savings:** Direct submission takes seconds. Kernel execution takes 10-30+ minutes.

---

## The `-k -v` Flag Requirement

### Why It Matters

Submissions MUST use `-k <kernel> -v <version>` flags for:
- **Notebook linkage** - Clicking submission shows the code
- **Code traceability** - Others can reproduce results
- **Competition rules** - Most competitions require code attribution

### The Common Mistake

```bash
# ❌ WRONG: Just uploads file, no attribution
kaggle competitions submit -c competition -f submission.zip -m "message"

# ✅ CORRECT: Links submission to notebook
kaggle competitions submit -c competition -f submission.zip \
  -k user/notebook -v VERSION -m "message"
```

**Result of wrong approach:** Submission shows score but has **no notebook linkage** - clicking it shows nothing.

### Implementation-Level Enforcement

Enforce in code to prevent accidents:

```python
def submit_to_competition(competition, submission_path, kernel_slug=None, version=None, message="Auto-submitted"):
    """Submit to competition with kernel linkage.
    
    CRITICAL: kernel_slug and version are REQUIRED.
    Without -k -v flags, submissions are orphan file uploads.
    """
    if not kernel_slug or not version:
        return {
            "success": False,
            "error": "kernel_slug and version are REQUIRED",
            "hint": "Use: kaggle competitions submit -c COMP -f file.zip -k user/kernel -v VERSION"
        }
    
    cmd = f"kaggle competitions submit -c {competition} -f {submission_path} -k {kernel_slug} -v {version} -m \"{message}\""
    # Execute command...
```

---

## Key Takeaways

| Pattern | Action |
|---------|--------|
| Try direct upload first | Faster if it works |
| Use `-k -v` flags | Always for code competitions |
| Enforce in code | Prevent accidental orphan submissions |
