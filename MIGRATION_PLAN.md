# Migration Plan: Python 3.10 → 3.11

## Executive Summary
- **Timeline**: 3 weeks (conservative) or 1-2 weeks (aggressive)
- **Risk Level**: MEDIUM
- **Critical Changes**: gym → gymnasium migration

---

## Critical Issues to Fix

### 1. Gym → Gymnasium Migration (HIGH PRIORITY)

Replace all gym imports with gymnasium and update API calls.

**Import Changes:**
```python
# BEFORE
import gym
from gym import spaces
from gym import Wrapper

# AFTER
import gymnasium as gym
from gymnasium import spaces
from gymnasium import Wrapper
```

**reset() Method Update:**
```python
# BEFORE
state = env.reset()

# AFTER
state, info = env.reset(seed=42)
```

**step() Method Update:**
```python
# BEFORE
state, reward, done, info = env.step(action)

# AFTER
state, reward, terminated, truncated, info = env.step(action)
done = terminated or truncated
```

**Affected Files** (5 files with direct imports):
- `train/config.py` (lines 172, 201, 205, 258, 261, 262)
- `train/evaluator.py` (lines 271, 279, 312, 320)
- `train/demo.py` (lines 2, 10, 14, 15, 28, 33, 39, 41, 42, 55, 62)

**Files with env.reset() calls** (25 instances):
- `drl_agents/agents/AgentBase.py:113, 143`
- `drl_agents/agents/AgentPPO.py:79`
- `drl_agents/agents/AgentA2C.py:139, 174`
- `drl_agents/elegantrl_models.py:109`
- `train/evaluator.py:160`
- `train/run.py:26, 75, 80`
- `train/utils.py:34, 39`
- `train/demo.py:28, 55`
- `environment_CCXT.py:129`
- `environment_Alpaca.py:91`

**Files with env.step() calls** (18 instances):
- `train/evaluator.py:173`
- `drl_agents/agents/AgentBase.py:108, 136`
- `drl_agents/agents/AgentPPO.py:74, 102`
- `drl_agents/agents/AgentA2C.py:135, 174`
- `drl_agents/elegantrl_models.py:121`
- `train/demo.py:33, 62`
- `environment_CCXT.py:228`
- `environment_Alpaca.py:172`

### 2. pandas .append() Replacement (MEDIUM PRIORITY)

**3 instances** to fix:
- `processor_Yahoo.py:160`
- `processor_Base.py:114-116`
- `processor_Base.py:132`

**Change Pattern:**
```python
# BEFORE
new_df = new_df.append(tmp_df)

# AFTER
new_df = pd.concat([new_df, tmp_df], ignore_index=True)
```

### 3. Remove Outdated Import (LOW PRIORITY)

**File**: `function_PBO.py:1`
```python
# Remove this line:
# from __future__ import print_function
```

---

## Dependency Updates

### requirements-python311.txt
```txt
# Core ML libraries
torch>=2.9.0,<3.0.0
numpy>=1.24.4,<2.0.0
pandas>=2.2.0,<3.0.0

# Binance API
python-binance>=1.0.32,<2.0.0

# Reinforcement Learning
elegantrl>=0.3.6,<1.0.0
gymnasium>=1.2.0,<2.0.0

# Optimization
optuna>=4.0.0,<5.0.0

# Technical Analysis
TA-Lib>=0.6.0,<1.0.0
stockstats>=0.6.0,<1.0.0

# Visualization
matplotlib>=3.8.0,<4.0.0
seaborn>=0.12.2,<1.0.0

# Network dependencies
requests>=2.31.0
urllib3>=2.0.0

# Utilities
tqdm>=4.66.0
psutil>=5.9.0
joblib>=1.3.0

# Finance data
yfinance>=0.2.50,<1.0.0

# Statistics
scipy>=1.14.0,<2.0.0
scikit-learn>=1.5.0,<2.0.0

# Timezone
pytz>=2024.1

# Additional required packages
aiohttp>=3.9.0
certifi>=2023.11.0
```

---

## Testing Checklist

### Phase 1: Smoke Tests
- [ ] Import verification (all imports load correctly)
- [ ] Data pipeline test (data download works)
- [ ] Environment creation test (gymnasium envs initialize)

### Phase 2: Integration Tests
- [ ] Basic agent training (demo script runs)
- [ ] Optimization setup (Optuna initializes)
- [ ] Validation on existing models (load & test)

### Phase 3: End-to-End Tests
- [ ] Full optimization run (multiple trials)
- [ ] Backtesting (performance metrics computed)
- [ ] PBO analysis (overfitting check)

---

## Timeline

| Phase | Duration | Description |
|-------|----------|-------------|
| Preparation | 1 day | Environment setup, backups |
| Code Fixes | 3 days | pandas .append(), __future__ |
| Gym Migration | 5 days | All gym imports, return values |
| Dependencies | 3 days | Update requirements, install |
| Testing | 5 days | All test scripts passing |
| Documentation | 1 day | README, migration guide |
| Validation | 1 day | Pre-commit checklist |
| Deployment | 2 days | Staging + production |
| **Total** | **21 days** | **3 weeks** |

**Aggressive Option**: 10-14 days (parallelize phases, reduce testing depth)

---

## Risk Assessment

| Risk Category | Risk Level | Mitigation Strategy |
|--------------|------------|---------------------|
| Gymnasium API changes | **HIGH** | Thorough testing, systematic replacement |
| pandas .append() deprecation | **MEDIUM** | Simple find-replace operation |
| Elegantrl compatibility | **MEDIUM** | Already working with 3.11, monitor for issues |
| Performance regression | **LOW** | Benchmark before/after, < 5% tolerance |
| Training result differences | **MEDIUM** | Compare metrics, investigate if drift > 5% |

---

## Rollback Plan

### Immediate Rollback (< 1 hour after deployment)
```bash
git checkout <previous-stable-commit>
pyenv local 3.10.12
pip install -r requirements-python310.txt
```

### Full Rollback (< 24 hours)
```bash
# Restore backups
cp -r train_results_backup/ train_results/
cp -r data_backup/ data/

# Revert to Python 3.10 environment
pyenv uninstall 3.11.12
pyenv local 3.10.12
pip install -r requirements-python310.txt
```

---

## Success Criteria

Migration is successful when:
- ✅ All test scripts run without errors
- ✅ Training results within 5% of baseline
- ✅ No regression in Sharpe ratio
- ✅ Performance benchmarks stable or improved
- ✅ Documentation updated
- ✅ Staging deployment successful
- ✅ Production monitoring stable for 48 hours

---

## Next Steps

1. Review and approve this migration plan
2. Create development branch for migration work
3. Execute Phase 1: Preparation
4. Proceed through all phases sequentially
5. Deploy and monitor

For questions or concerns, refer to detailed analysis in project repository.
