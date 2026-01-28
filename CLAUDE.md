# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Bittensor subnet template** for building incentive mechanisms on the Bittensor network. The template provides boilerplate for miners (produce value) and validators (produce consensus) that interact via a custom protocol to determine TAO distribution.

## Common Commands

### Installation
```bash
pip install -e .
```

### Running Neurons
```bash
# Run miner (requires registered hotkey on subnet)
python neurons/miner.py --netuid <NETUID> --wallet.name <WALLET> --wallet.hotkey <HOTKEY>

# Run validator
python neurons/validator.py --netuid <NETUID> --wallet.name <WALLET> --wallet.hotkey <HOTKEY>

# Run with mock mode for local testing (no network required)
python neurons/miner.py --mock
python neurons/validator.py --mock
```

### Code Quality
```bash
# Format code (line length 79)
black --line-length 79 --exclude '(env|venv|.eggs|.git)' .

# Lint
flake8
pylint ./

# Type check
mypy .

# Run tests
pytest tests/
```

## Architecture

### Class Hierarchy
```
BaseNeuron (template/base/neuron.py)
├── BaseMinerNeuron (template/base/miner.py)
│   └── Miner (neurons/miner.py)
└── BaseValidatorNeuron (template/base/validator.py)
    └── Validator (neurons/validator.py)
```

### Core Components

**Protocol** (`template/protocol.py`): Defines the `Synapse` subclass for miner-validator communication. The default `Dummy` synapse sends an integer and expects it doubled.

**Miner**: Runs an `axon` server that listens for validator requests. Key methods to customize:
- `forward()` - Process request and generate response
- `blacklist()` - Decide whether to reject requests (security)
- `priority()` - Order requests by validator stake

**Validator**: Uses a `dendrite` client to query miners. Key flow in `template/validator/forward.py`:
1. Select miners via `get_random_uids()`
2. Send synapse query via dendrite
3. Calculate rewards via `get_rewards()` in `template/validator/reward.py`
4. Update scores with exponential moving average
5. Set on-chain weights periodically (every `epoch_length` blocks)

### Key Files to Modify

When creating a custom subnet, edit these files (marked with TODOs):
- `template/protocol.py` - Define your wire protocol
- `neurons/miner.py` - Miner response logic
- `neurons/validator.py` - Validator query logic
- `template/validator/forward.py` - Validator forward pass
- `template/validator/reward.py` - Scoring/reward calculation

The `template/` directory should be renamed to your subnet name.

### Mock Testing

Use `--mock` flag to run without network connection:
- `MockSubtensor` - Simulates blockchain with n mock miners
- `MockMetagraph` - Syncs with mock subtensor
- `MockDendrite` - For validator testing

### Configuration

Key CLI arguments:
- `--netuid` - Subnet ID
- `--neuron.device` - cuda/cpu (auto-detected)
- `--neuron.epoch_length` - Blocks between weight updates (default: 100)
- `--neuron.sample_size` - Miners to query per forward pass
- `--blacklist.force_validator_permit` - Only accept validator requests
- `--mock` - Use mock components for testing

### State Management

Validators save/load state to `<neuron.full_path>/state.npz` containing:
- `step` - Current step counter
- `scores` - Miner score array
- `hotkeys` - Cached hotkey list

## Code Style

- Use `black` with line length 79
- Follow PEP 8 conventions
- Import order: standard library, third-party, local
- Class names: CapWords
- Functions/variables: snake_case
- Constants: ALL_CAPS
