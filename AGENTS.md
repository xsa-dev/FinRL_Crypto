# Agent Development Guidelines

## Commands
- **Setup**: `pip install -r requirements.txt` (Python 3.10+)
- **Run single test**: No formal test framework - use individual scripts
- **Lint/Typecheck**: Not configured - add py lint and mypy if needed
- **Validation**: `python 2_validate.py` - validates trained agents

## Code Style
- **Imports**: Standard library first, then third-party, then local modules
- **Types**: Use type hints for function signatures (follow AgentBase.py:10 pattern)
- **Naming**: 
  - Classes: PascalCase (AgentBase, AgentPPO)
  - Functions/variables: snake_case (calculate_start_end_dates)
  - Constants: UPPER_CASE (TIMEFRAME, TICKER_LIST)
- **Error Handling**: Basic try/except blocks, see config_main.py for patterns
- **Documentation**: Docstrings for functions and classes (follow AgentBase.py:12-26)
- **Formatting**: Follow existing indentation and spacing patterns

## Architecture Notes
- Agent inheritance from AgentBase in drl_agents/agents/
- Configuration in config_main.py with datetime calculations
- No formal testing - validation through 2_validate.py script
 
## Core Architecture
 
### Base Agent
 
**[AgentBase](drl_agents/agents/AgentBase.py)** - The foundational class that provides:
- Environment exploration (single and vectorized)
- Trajectory conversion and replay buffer management
- Network optimization utilities
- Model persistence (save/load)
- Prioritized Experience Replay (PER) support
 
## Supported Algorithms
 
### Policy Gradient Methods
 
#### PPO (Proximal Policy Optimization)
**[AgentPPO](drl_agents/agents/AgentPPO.py)** - State-of-the-art policy gradient algorithm
- **AgentDiscretePPO**: For discrete action spaces (buy/sell/hold)
- **AgentSharePPO**: Shared parameter version for pixel-level states
- Features: GAE (Generalized Advantage Estimation), KL divergence clipping
 
#### A2C (Advantage Actor-Critic)
**[AgentA2C](drl_agents/agents/AgentA2C.py)** - Synchronous advantage actor-critic
- **AgentDiscreteA2C**: Discrete action variant
- **AgentShareA2C**: Shared parameter implementation
- Built on PPO foundation with simplified updates
 
### Value-Based Methods
 
#### DQN Variants
Available in [net.py](drl_agents/agents/net.py):
- **QNet**: Standard Deep Q-Network
- **QNetDuel**: Dueling DQN (state/value separation)
- **QNetTwin**: Double DQN (reduces overestimation)
- **QNetTwinDuel**: Dueling Double DQN (D3QN)
 
### Actor-Critic Methods
 
#### DDPG (Deep Deterministic Policy Gradient)
**[AgentDDPG](drl_agents/agents/AgentDDPG.py)** - Continuous control with deterministic policies
- Suitable for continuous trading actions (position sizing)
 
#### TD3 (Twin Delayed DDPG)
**[AgentTD3](drl_agents/agents/AgentTD3.py)** - Improved DDPG with:
- Twin critics for stability
- Delayed policy updates
- Target policy smoothing
 
#### SAC (Soft Actor-Critic)
**[AgentSAC](drl_agents/agents/AgentSAC.py)** - State-of-the-art off-policy algorithm
- **AgentModSAC**: Modified version with TTUR (Two Time-scale Update Rule)
- **AgentShareSAC**: Shared parameter implementation
- Features: Entropy regularization, automatic temperature tuning
 
## Neural Network Architectures
 
### Actor Networks
- **Actor**: Standard deterministic policy network
- **ActorSAC**: Stochastic policy with reparameterization
- **ActorPPO**: Policy network with log-probability computation
- **ActorDiscretePPO**: Discrete action policy for crypto trading
 
### Critic Networks
- **Critic**: Standard Q-value network
- **CriticPPO**: Value function for PPO
- **CriticTwin**: Double Q-networks for stability
- **CriticREDQ**: Randomized Ensemble Double Q-learning
 
### Shared Networks
- **SharePPO**: Combined actor-critic for image-based states
- **ShareSPG**: Stochastic policy gradient with shared parameters
 
## Key Features
 
### Action Spaces
- **Continuous**: Precise position sizing (0-100% portfolio allocation)
- **Discrete**: Buy/Sell/Hold decisions
- **Multi-action**: Simultaneous trading across multiple cryptocurrencies
 
### Training Optimizations
- Prioritized Experience Replay (PER)
- Generalized Advantage Estimation (GAE)
- Vectorized environment support
- GPU acceleration
- Gradient clipping and normalization
 
### Financial Applications
- Portfolio optimization
- Risk management through entropy regularization
- Multi-asset trading strategies
- High-frequency trading support
 
## Usage Patterns
 
1. **Continuous Trading**: Use SAC, TD3, or DDPG for position sizing
2. **Discrete Decisions**: Use PPO or A2C for buy/sell/hold signals
3. **Multi-asset**: Use vectorized environments with shared networks
4. **Research**: Experiment with different network architectures in net.py
 
## Configuration
 
All agents support standard hyperparameters:
- `net_dim`: Network hidden layer dimensions
- `state_dim`: Environment state space dimensionality
- `action_dim`: Action space dimensionality
- `learning_rate`: Optimizer learning rate
- `gamma`: Discount factor
- `gpu_id`: CUDA device selection
 
## Integration
 
The agents integrate seamlessly with:
- Crypto data processors ([Binance](processor_Binance.py), [Yahoo](processor_Yahoo.py))
- Trading environments ([Alpaca](environment_Alpaca.py))
- Validation frameworks ([CPCV](function_CPCV.py), [KCV](1_optimize_kcv.py))
- Backtesting system ([4_backtest.py](4_backtest.py))