# Automation Patterns

Patterns for automating Kaggle competition workflows.

---

## Cronjob-Based Monitoring

### Purpose

Monitor long-running kernels and auto-submit when complete.

### Pattern

```yaml
# Cronjob configuration
schedule: "every 5m"
prompt: |
  Check kernel status at kaggle kernels status <username>/<kernel-name>
  If status is COMPLETE, submit to competition
  If status is ERROR, report the error
  Otherwise, continue monitoring
```

### Benefits

- No manual polling
- Timely submission
- Error notification

### Implementation

Use Hermes `cronjob` tool:

```bash
# Create monitoring cronjob
cronjob action=create \
  name="kaggle-monitor" \
  schedule="every 5m" \
  prompt="Check kernel status and auto-submit when complete" \
  enabled_toolsets=["terminal"]
```

---

## Automated Notebook Push

### Workflow

1. Local development
2. Push to Kaggle
3. Monitor execution
4. Fetch results
5. Analyze and iterate

### Script Pattern

```bash
#!/bin/bash
# push-and-monitor.sh

KERNEL_NAME="username/kernel-name"
COMPETITION="competition-name"

# Push kernel
echo "Pushing kernel..."
kaggle kernels push -p ./notebook/

# Wait for completion
echo "Monitoring execution..."
while true; do
    STATUS=$(kaggle kernels status $KERNEL_NAME | grep -oP '(?<=status": ")[^"]+')
    echo "Status: $STATUS"
    if [ "$STATUS" = "COMPLETE" ]; then
        echo "Kernel complete!"
        break
    elif [ "$STATUS" = "ERROR" ]; then
        echo "Kernel failed!"
        kaggle kernels output $KERNEL_NAME -p ./output/
        exit 1
    fi
    sleep 300  # Wait 5 minutes
done

# Fetch output
kaggle kernels output $KERNEL_NAME -p ./output/
echo "Output saved to ./output/"
```

---

## Spec-Driven Development

### Process

1. **Document before coding**
2. **Delegate with clear constraints**
3. **Iterate based on results**

### SPEC.md Template

```markdown
# SPEC.md - [Competition Name]

## 1. Competition Overview
- Task: [What needs to be done]
- Deadline: [Date]
- Prize: [Amount]
- Current top score: [Score]
- My target: [Score]

## 2. Task Definition
- Input: [Data format]
- Output: [Prediction format]
- Evaluation metric: [Metric name]
- Data size: [Rows, columns, files]

## 3. Top Approaches (from Kaggle Notebooks)
### Approach A: [Name]
- Votes: [Number]
- LB Score: [Score]
- Key techniques: [List]
- Reference: [Kernel URL]

### Approach B: [Name]
- [Same structure]

## 4. Strategy & Implementation Plan
### Phase 1: Quick Baseline
- Goal: [What]
- Timeline: [Days]
- Approach: [How]

### Phase 2: High-ROI Improvements
- Goal: [What]
- Timeline: [Days]
- Approach: [How]

### Phase 3: Fine-tuning (if needed)
- Goal: [What]
- Timeline: [Days]
- Approach: [How]

## 5. Technical Constraints
- Submission format: [CSV/zip/model]
- Time limit: [Seconds]
- GPU required: [Yes/No]
- Internet: [Enabled/Disabled]
- Dependencies: [List]

## 6. Execution Strategy
- Local development: [What can be done locally]
- Kaggle: [What needs GPU/internet]

## 7. Success Metrics
| Milestone | Target | Timeline |
|-----------|--------|----------|
| Baseline | 0.xx | Day X |
| Improved | 0.xx | Day Y |
| Final | 0.xx | Day Z |

## 8. Lessons Learned
- [Update as you go]
```

### Delegation Command

```bash
opencode run "Implement the approach documented in SPEC.md. Follow Phase 1 first, then iterate."
```

---

## Delegation Strategy

### When to Delegate

| Task | Delegate? | To Whom |
|------|-----------|---------|
| Simple file download | No | — |
| Metadata changes | No | — |
| Direct fork | No | — |
| Code adaptation | Yes | OpenCode |
| Feature implementation | Yes | OpenCode |
| PR review | Yes | Claude Code |
| Complex debugging | Yes | Claude Code |
| Parallel research | Yes | Subagents |

### Delegation Pattern

```bash
# Clear spec → Delegate → Verify → Iterate

# 1. Create spec
cat > SPEC.md << EOF
[Detailed spec]
EOF

# 2. Delegate
opencode run "Follow SPEC.md to implement the solution"

# 3. Verify
# - Check code changes
# - Test locally (if possible)
# - Push to Kaggle
# - Monitor results

# 4. Iterate
# - Update SPEC.md with lessons learned
# - Re-delegate if needed
```

---

## Hybrid Technique Combining

### Pattern: Extract + Combine

1. Pull multiple top kernels
2. Extract key techniques from each
3. Combine into new solution
4. Create new kernel with combined approach

### Example

```bash
# Pull kernel A (good embeddings)
kaggle kernels pull user/kernel-a -p ./analysis/a/ -m

# Pull kernel B (good ensemble)
kaggle kernels pull user/kernel-b -p ./analysis/b/ -m

# Extract techniques
# - From A: Embedding pipeline
# - From B: Ensemble method

# Create SPEC.md for combined approach
cat > SPEC.md << EOF
## Combined Approach
- Embeddings: [From kernel-a]
- Ensemble: [From kernel-b]
EOF

# Delegate implementation
opencode run "Combine techniques per SPEC.md"
```

---

## Automated Result Collection

### Pattern

```python
# In Kaggle notebook, print structured results
import json

results = {
    "model": "ModelName",
    "cv_score": 0.923,
    "submission_rows": len(submission),
    "timestamp": datetime.now().isoformat()
}

print("=== RESULTS ===")
print(json.dumps(results, indent=2))
```

### Fetch and Parse

```python
# Locally, after kernel completes
import json

with open('./output/kernel.log', 'r') as f:
    log_entries = json.load(f)

for entry in log_entries:
    if entry.get('stream_name') == 'stdout':
        if '=== RESULTS ===' in entry['data']:
            # Parse the JSON results
            ...
```

---

## Error Recovery Pattern

### Auto-Retry on Transient Failures

```python
from hermes_tools import terminal, retry

@retry(max_attempts=3, delay=60)
def push_kernel(path):
    result = terminal(f"kaggle kernels push -p {path}")
    if result['exit_code'] != 0:
        raise Exception(f"Push failed: {result['output']}")
    return result
```

### Fallback Strategies

| Failure | Fallback |
|---------|----------|
| CLI submit fails | Manual upload via web |
| Kernel data not attached | Fork working kernel |
| GPU unavailable | Use CPU with quantization |
| API rate limit | Switch model/provider |

---

## Monitoring Dashboard

### Key Metrics

- Active kernels
- Pending submissions
- Recent scores
- Error rate

### Simple Dashboard Script

```bash
#!/bin/bash
# dashboard.sh

echo "=== Kaggle Competition Dashboard ==="
echo ""

echo "Recent Submissions:"
kaggle competitions submissions <name> | head -10

echo ""
echo "Kernel Status:"
kaggle kernels status <username>/<kernel-name>

echo ""
echo "Leaderboard (top 10):"
kaggle competitions leaderboard <name> -s | head -15
```

---

## Best Practices

1. **Always set `is_private: true`** for competition notebooks
2. **Use cronjobs** for long-running monitoring tasks
3. **Log everything** in kernels for debugging
4. **Verify submissions** before declaring success
5. **Document lessons** in SPEC.md for future reference
