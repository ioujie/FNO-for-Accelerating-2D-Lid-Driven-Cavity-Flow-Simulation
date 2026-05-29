# FNO for Accelerating 2-D Lid-Driven Cavity Flow Simulation

ECE 228 Project - Team 68

This project trains Fourier Neural Operator (FNO) surrogates on transient 2-D lid-driven cavity flow data. The goal is to compare a data-only FNO against physics-informed variants that penalize incompressibility error and boundary mismatch.

## Project Overview

The model predicts the next flow field from a short sequence of previous fields. Each frame contains pressure and two velocity components:

```text
q_t(x, y) = [p_t(x, y), u_t(x, y), v_t(x, y)]
```

For incompressible flow, the velocity field should satisfy:

```text
div(u) = du/dx + dv/dy = 0
```

The notebook evaluates three experiments:

1. **Baseline FNO** with physical-space MSE only.
2. **Tuned PI-FNO** with physical-space MSE, divergence penalty, and boundary penalty.
3. **Fourier-loss ablation** with spectral-space data loss and the same physical penalties.

## Main Files

- `FNO_Cavity_Flow.ipynb`: main executable notebook with data loading, model definition, training, evaluation, stability study, and visualization.
- `outputs/experiment_summary.csv`: validation metrics for the main experiments.
- `outputs/holdout_re_results.csv`: held-out Reynolds-number metrics.
- `outputs/stability_results.csv`: seed/window-offset stability metrics.
- `outputs/figures/`: generated qualitative and training-history figures.

## Data

The lid-driven cavity CFD data used in this project comes from Rocha et al. [1], which benchmarks Fourier Neural Operator models on the lid-driven cavity case.

The notebook expects simulation CSV files organized by Reynolds number under:

```text
Transient_k-e/
```

Each CSV contains point coordinates and flow variables. The loader selects one thin `z` slice, sorts points by `(y, x)`, reconstructs the structured 2-D grid, and builds consecutive sliding temporal windows.

## Environment

The project was run with Anaconda using the `ml-pip` environment. Required Python packages include:

```text
numpy
pandas
matplotlib
torch
jupyter
```

CUDA is used automatically when available.

## Running the Notebook

Open and run:

```text
FNO_Cavity_Flow.ipynb
```

The notebook trains all enabled experiments:

- baseline FNO
- tuned PI-FNO
- Fourier-loss ablation
- stability study over random seeds and temporal-window offsets

Generated outputs are written to:

```text
outputs/
```

The full stability study trains multiple models and can take substantially longer than the three main experiments.

## Loss Functions

The baseline objective is:

```text
L = L_data
```

The tuned physics-informed objective is:

```text
L = L_data + 0.02 L_div + 0.01 L_bc
```

where:

- `L_data` is the field reconstruction MSE.
- `L_div` penalizes predicted velocity divergence.
- `L_bc` penalizes boundary mismatch.

The Fourier-loss ablation computes `L_data` in Fourier space while keeping `L_div` and `L_bc` in physical space.

## Result Summary

The main validation results show the expected accuracy-physics tradeoff:

| Model | Velocity RMSE | Divergence L2 | Boundary RMSE |
|---|---:|---:|---:|
| Baseline FNO | 0.00595 | 0.663 | 0.0279 |
| Tuned PI-FNO | 0.01415 | 0.208 | 0.0793 |
| Fourier-loss ablation | 0.01641 | 0.196 | 0.0511 |

The baseline has the best velocity RMSE, while the physics-informed models greatly reduce divergence error. Across held-out Reynolds numbers, the same trend is preserved: physics-informed training improves incompressibility at the cost of higher reconstruction error.

The stability study shows that the tuned PI-FNO has higher RMSE variance than the baseline but much lower divergence error across seeds and temporal-window offsets.

## References

[1] Paulo Alexandre Costa Rocha, Samuel Joseph Johnston, Victor Oliveira Santos, Amir A. Aliabadi, Jesse Van Griensven The, and Bahram Gharabaghi. "Deep Neural Network Modeling for CFD Simulations: Benchmarking the Fourier Neural Operator on the Lid-Driven Cavity Case." *Applied Sciences*, 13(5):3165, 2023.
