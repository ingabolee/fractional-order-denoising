# Fractional-Order Denoising for Large HDF5 Datasets

A chunk-based signal denoising workflow using a **fractional Grünwald–Letnikov filter** and **Bayesian optimization**. The notebook is designed for large HDF5 datasets that may not fit into memory.

## Overview

The workflow processes four signal channels from an HDF5 dataset, optimizes the filter parameters for each chunk, and carries recent output history between chunks to reduce boundary discontinuities.

### Highlights

- **Memory-aware processing** — reads and writes HDF5 data in configurable chunks.
- **Fractional-order filtering** — computes Grünwald–Letnikov weights and applies a finite-memory recursive filter.
- **Adaptive parameter selection** — uses Bayesian optimization to select `beta` and `lambda` for each chunk.
- **Noise-energy analysis** — compares original and denoised channels using sum of squared residuals (SSE).
- **Interactive diagnostics** — uses Plotly to inspect original signals, denoised signals, and removed-noise residuals.

## Repository structure

```text
fractional-order-denoising/
├── data/                         # Place input .hdf5 files here (not tracked)
├── notebooks/
│   └── fractional_order_denoising.ipynb
├── .gitignore
├── LICENSE
├── README.md
└── requirements.txt
```

## Getting started

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/fractional-order-denoising.git
cd fractional-order-denoising
```

### 2. Create an environment and install dependencies

```bash
python -m venv .venv
# macOS / Linux
source .venv/bin/activate
# Windows PowerShell
.venv\\Scripts\\Activate.ps1

python -m pip install --upgrade pip
pip install -r requirements.txt
```

### 3. Add your HDF5 data

Place your input files in `data/`. The notebook discovers files matching `*.hdf5` in that directory. The expected input dataset is named `arr` and contains four signal columns.

> The original data files are not included because they are large and may contain private or proprietary measurements.

### 4. Run the notebook

```bash
jupyter lab
```

Open `notebooks/fractional_order_denoising.ipynb` and run the cells in order. The main processing section creates a `combined_data` dataset with the four original channels followed by four denoised channels.

## Configuration

The notebook exposes the following parameters:

| Parameter | Default | Purpose |
|---|---:|---|
| `CHUNK_SIZE` | `100000` | Rows processed per HDF5 chunk |
| `MEMORY_LEN` | `10000` | Maximum number of previous output samples used by the fractional filter |
| `OPT_SAMPLE_SIZE` | `2000` | Leading samples from each chunk used for parameter optimization |
| `bounds` | `beta: (0.1, 0.9)`, `lam: (1, 50)` | Search bounds for Bayesian optimization |

Adjust these values based on signal characteristics, dataset size, and available memory.

## Noise analysis

The analysis section computes the residual for each channel,

\[
n_i = y_i - \\hat{y}_i,
\]

and accumulates its sum of squared errors,

\[
E_{\\mathrm{noise}} = \\sum_i n_i^2.
\]

The channel with the largest SSE is reported as having the highest removed-noise energy. This is a measure of residual energy removed by the filter; it should not automatically be interpreted as a measure of denoising quality without additional signal-quality criteria.

## Important notes

- **Back up your input files.** The denoising pipeline writes `combined_data` into each HDF5 file.
- The final optional compaction cell is destructive: it replaces the selected dataset with a dataset containing only the columns after the first four. Run it only on a backed-up copy.
- Bayesian optimization runs independently for each channel and chunk, so processing time can be substantial.
- The implementation expects four input signal columns and writes eight output columns (`4 original + 4 denoised`).
- Validate the filter parameters and results on a small representative dataset before processing production data.

## Dependencies

See [`requirements.txt`](requirements.txt).

## License

Distributed under the MIT License. See [`LICENSE`](LICENSE).
