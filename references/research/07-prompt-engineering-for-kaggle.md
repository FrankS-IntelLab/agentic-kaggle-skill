# Prompt Engineering for Kaggle Delegation

> *How to write effective prompts when delegating work to AI agents during a competition.*

---

## When to Delegate vs Do It Yourself

| Task | Delegate? | Why |
|------|-----------|-----|
| Download data / check files | No | Faster to do directly |
| Metadata edits (kernel-metadata.json) | No | Mechanical, low risk |
| Fork & push existing kernel | No | One-line CLI command |
| Feature engineering | **Yes** | Agent can explore many transforms quickly |
| Model training & tuning | **Yes** | Agent can iterate on architecture/hyperparams |
| Ensemble construction | **Yes** | Agent can test many blend combinations |
| Debugging failing kernels | **Yes** | Agent can read logs and diagnose |
| Final submission decisions | **No** | Human judgment on risk/reward |

---

## The Delegation Prompt Framework

### Structure

Every delegation prompt should contain these elements:

1. **Context** — What competition are we in? What is the task?
2. **Data** — Where is the data? What does it look like?
3. **Goal** — What exactly should the agent produce?
4. **Constraints** — Time, memory, internet, GPU limits
5. **Success Criteria** — How will we know it worked?
6. **Forbidden Actions** — What must the agent NOT do?

### Template

```markdown
## Competition Context
- Task: [e.g., binary classification / object detection / forecasting]
- Metric: [e.g., AUC-ROC / F1 / RMSE]
- Submission format: [CSV / model weights / ONNX]

## Data
- Training data: /kaggle/input/<dataset>/train/ (describe structure)
- Test data: /kaggle/input/<dataset>/test/
- Sample submission: /kaggle/input/<dataset>/sample_submission.csv
- Key columns: [list]
- Size: [rows × columns]

## Goal
[Specific deliverable, e.g.: "Train a model and generate submission.csv"]

## Constraints
- Internet: [enabled / disabled]
- GPU: [available / not available]
- Time limit: [per-sample or per-batch]
- Memory: [if known]

## Success Criteria
- Must beat current score of [X]
- Must use [specific technique or constraint]

## Forbidden
- Do not use training data as fallback when test data is missing
- Do not hardcode file paths outside /kaggle/input/
- Do not assume test data has the same distribution as train
```

---

## Effective Prompt Patterns

### ✅ Good Prompt — Feature Engineering
```
I'm working on a tabular classification competition.
Training data: /kaggle/input/comp-data/train.csv (5000 rows, 30 features, target column "label")
Test data: /kaggle/input/comp-data/test.csv (2000 rows, same features, no label)

Create 10 new features using interactions, aggregations, and transforms.
- Check each new feature's correlation with the target
- Drop any feature with >0.95 correlation to an existing feature
- Save the enhanced dataset to /kaggle/working/enhanced_train.csv and /kaggle/working/enhanced_test.csv

Report: list each new feature and its correlation with target.
```

### ✅ Good Prompt — Model Training
```
Train a LightGBM classifier on /kaggle/working/enhanced_train.csv
Target: "label", features: all other columns
Constraints:
- 5-fold stratified CV
- Optimize for AUC-ROC
- Early stopping on fold 0
- Max 500 boosting rounds, patience 50
Report: mean CV score + std, feature importance top 10
Save model to /kaggle/working/model.txt
```

### ✅ Good Prompt — Debugging
```
My kernel at https://www.kaggle.com/code/username/kernel-name failed with error:
[paste error message]

The kernel:
1. Reads train/test data from /kaggle/input/...
2. Trains a model
3. Generates submission.csv

Diagnose the error and fix the code.
Do NOT change the model architecture — only fix the bug.
Push the updated kernel to the same path.
```

### ❌ Bad Prompt — Too Vague
```
Build a good model for my competition
```

### ❌ Bad Prompt — No Constraints
```
Train the best possible classifier. Use whatever you need.
```

### ❌ Bad Prompt — Missing Data Context
```
Make a submission for the competition. It should score well.
```

---

## Common Delegation Failures

| Failure | Cause | Fix |
|---------|-------|-----|
| Agent goes off-track | Prompt too vague | Add specific goal + success criteria |
| Agent uses forbidden internet | No constraint stated | Always state "internet disabled if applicable" |
| Agent ignores time limit | No time constraint | Specify "max X seconds per prediction" |
| Agent produces wrong format | No format example | Provide sample of expected output |
| Agent trains on test data | No data separation instruction | Explicitly say "train ONLY on train data" |
| Agent doesn't report results | No reporting instruction | Always ask "report: [specific metrics]" |

---

## Iteration Pattern

Delegation is rarely one-shot. Use this loop:

```
1. Delegate with clear prompt
2. Agent returns result + report
3. You verify locally (if possible)
4. If result is good → proceed
5. If result needs improvement → refine prompt with:
   - What was wrong
   - What to try differently
   - Updated constraints
6. Repeat until satisfied
```

---

## Multi-Agent Delegation

When using multiple subagents in parallel:

- **Give each agent a different angle** — e.g., one does feature engineering, another does hyperparameter tuning
- **Define clear interfaces** — each agent should output to a known file path
- **Assign a validator** — one agent checks the others' outputs before merging
- **Avoid conflicting writes** — never have two agents write to the same file

---

## Key Takeaways

1. **Context is king** — The more relevant context you give, the better the result
2. **Constraints prevent disasters** — Always state what the agent must NOT do
3. **Show, don't just tell** — Include examples of expected input/output format
4. **Iterate** — First attempt rarely perfect; refine based on results
5. **Verify before trusting** — Always check agent output before submitting