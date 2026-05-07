# Environment Setup Patterns

> *Patterns for setting up competition environments and dependencies.*

---

## 1. Kaggle Environments — Source Install

Newer competition environments are NOT in the PyPI `kaggle-environments` package (PyPI lags behind). Install from GitHub source when you see environment errors.

### When to Use

- You get `InvalidArgument: Unknown Environment Specification` error
- Local testing fails because environment isn't recognized
- Competition is new and PyPI package is outdated

### Problem

```bash
pip install kaggle-environments  # Gets v1.25.x from PyPI

python3 -c "from kaggle_environments import make; make('new-competition-env')"
# ERROR: InvalidArgument: Unknown Environment Specification
```

### Solution

```bash
# Clone the latest source from GitHub
git clone --depth 1 https://github.com/Kaggle/kaggle-environments.git /tmp/kaggle-environments
cd /tmp/kaggle-environments

# Relax Python version constraint if needed
sed -i 's/requires-python = ">=3.11"/requires-python = ">=3.10"/' pyproject.toml

# Install in editable mode, skip optional dependencies
pip install -e . --no-deps

# Verify it works
python3 -c "
from kaggle_environments import make
env = make('your-competition-env', debug=False)
env.run(['random', 'random'])
print('✓ Environment works!')
"
```

### ⚠️ Important Notes

1. **Editable install** — Don't delete `/tmp/kaggle-environments/` after installing
2. **Overlay behavior** — This installs on top of the old package, adding new environments
3. **Version check** — GitHub source contains envs not yet released to PyPI

### Alternative: Test Directly on Kaggle

If local setup is too complex, create a test notebook on Kaggle:

```python
# Install latest in Kaggle notebook (has internet during editing)
!pip install --upgrade kaggle-environments

from kaggle_environments import make
env = make('your-competition-env')
env.run(['your_agent', 'random'])
```

---

## Summary

| Pattern | Problem | Solution |
|---------|---------|----------|
| Source Install | PyPI outdated for new envs | Install from GitHub source |
| Kaggle Notebook Testing | Local env unavailable | Test directly on platform |
