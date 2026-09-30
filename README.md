# Using Zero-Noise Extrapolation (ZNE) to improve Variational Quantum Eigensolver(VQE) for Portfolio Optimization

Portfolio optimization with VQE in PennyLane, with Zero-Noise Extrapolation (ZNE) to reduce the effect of simulated hardware noise.

The notebook runs the same VQE optimization three ways (ideal, noisy, and ZNE-mitigated) and compares their cost curves and output probabilities.

## Features

- Downloads stock data from Yahoo Finance and builds the portfolio Hamiltonian automatically
- Hardware-efficient ansatz with configurable depth
- Phase damping noise model on a mixed-state simulator
- ZNE with global folding and polynomial extrapolation
- Brute-force solver to check the exact optimal portfolio
- Exports per-iteration costs to Excel and plots the results

## Requirements

- Python 3.12
- pennylane
- mitiq
- cirq
- yfinance
- pandas
- openpyxl
- matplotlib
- jupyter

## Installation

```bash
git clone https://github.com/Kimchhor/zne_vqe_po.git
cd zne_vqe_po
pip install pennylane mitiq cirq yfinance pandas openpyxl matplotlib jupyter
```

## Usage

Open the notebook and run the cell:

```bash
jupyter notebook zne_vqe_po.ipynb
```

The pipeline starts at the bottom of the cell:

```python
vqe = CustumVQE()
vqe.run()
```

To change the settings, pass arguments to the constructor:

```python
vqe = CustumVQE(gamma=1, B=2, P=1.0, p=1, stepsize=0.02, max_steps=60)
vqe.run()
```

## Configuration

| Argument | Default | Description |
|---|---|---|
| `gamma` | 1 | Risk-aversion factor |
| `B` | 2 | Number of assets to select |
| `P` | 1.0 | Penalty weight for the budget constraint |
| `p` | 1 | Ansatz depth (number of entangling layers) |
| `stepsize` | 0.02 | Gradient descent learning rate |
| `max_steps` | 60 | Number of optimization steps |

Other settings are set inside the class:

| Setting | Location | Default |
|---|---|---|
| Tickers | `_fetch_stock_data` | AAPL, MSFT, AMZN, TSLA, GOOG, BRK-B |
| Date range | `_fetch_stock_data` | 2018-01-01 to 2018-12-31 |
| Number of qubits | `self.N` | 2 |
| Noise gate / strength | `__init__` | `qml.PhaseDamping`, 0.7 |
| ZNE scale factors | `__init__` | [1, 2, 3] |
| Extrapolation order | `__init__` | 2 |

## Output

Running `vqe.run()` produces:

- Cost printed every 5 steps for each run
- `vqe_results.xlsx` with ideal, noisy, and mitigated cost per iteration
- A line plot of cost vs. iteration for the three runs
- A bar chart of bitstring probabilities for the noisy and mitigated circuits

## How It Works

1. `_fetch_stock_data` downloads prices and computes annualized returns and covariance.
2. `_build_hamiltonian` turns the portfolio problem into a Pauli-Z Hamiltonian.
3. `ansatz` prepares the trial state with `RY` and `CNOT` gates.
4. `optimize` minimizes the Hamiltonian expectation with gradient descent.
5. `run` repeats the optimization on the ideal, noisy, and ZNE-mitigated QNodes, then saves and plots the results.

## Project Structure

```
zne_vqe_po/
└── zne_vqe_po.ipynb   # CustumVQE class and run script
```

## Troubleshooting

- **`KeyError: 'Adj Close'`**: newer `yfinance` versions adjust prices by default. Use `yf.download(..., auto_adjust=False)` or replace `['Adj Close']` with `['Close']`.
- **Noise insertion errors**: `qml.transforms.insert` on a device may behave differently across PennyLane versions. Pin a compatible PennyLane version if needed.
- **Probabilities look unoptimized**: `run()` passes `init_params` to the probability plots. Use the parameters returned by `optimize` to see the final distribution.
- **Only two assets used**: the default `N = 2` uses the first two tickers in the downloaded data. Increase `self.N` to include more.

## Related Paper

This code accompanies "Using Zero-Noise Extrapolation to improve Variational Quantum Eigensolver for Portfolio Optimization" (https://www.dbpia.co.kr/Journal/articleDetail?nodeId=NODE12034782) (KICS Fall Conference 2024, South Korea).
