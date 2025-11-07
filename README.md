# Cross-Site Water Demand Forecasting-Reproducible Code (Public Release)

This repository contains a clean, minimal, and *data-agnostic* implementation of the pipeline described in our manuscript on **cross-site water demand forecasting under covariate shift**. 
It includes baselines and models mentioned in the paper (Seasonal Naïve baseline, LightGBM, LSTM) and hooks for transfer mechanisms (statistical alignment via CORAL, adversarial (DANN) stub, and meta-learning (MAML) stub).

> **Note on data:** The operational SCADA data used in the study is **under NDA** and cannot be shared. We therefore ship a **synthetic generator** that replicates key statistical properties (diurnal/weekly seasonality, noise, covariate shift across sites) so the full code path can be executed end-to-end by reviewers/readers.

## Quick start

```bash
# 1) create env (optional)
python -m venv .venv && source .venv/bin/activate  # Windows: .venv\Scripts\activate

# 2) install
pip install -r requirements.txt

# 3) generate synthetic data
python -m src.data.make_synthetic --out ./experiments/synth --n_days 120 --n_sites 3

# 4) run a model
python -m src.main --data_dir ./experiments/synth --model seasonal_naive --horizon 24
python -m src.main --data_dir ./experiments/synth --model lightgbm --horizon 24
python -m src.main --data_dir ./experiments/synth --model lstm --horizon 24 --epochs 5
```

Outputs (metrics, residuals, and figures) are saved under `./outputs`.

## Repository layout

```
src/
  main.py                # CLI entry point
  datasets.py            # loaders + sequence building
  evaluation.py          # MSE/RMSE/MAE + residual ACF plot
  plots.py               # plotting helpers
  utils.py               # misc helpers
  data/
    make_synthetic.py    # synthetic SCADA-like data generator
  models/
    seasonal_naive.py
    lightgbm_model.py
    lstm.py
    informer.py          # placeholder (see notes)
  transfer/
    coral.py
    dann.py              # stub
    maml.py              # stub
```

## Models

- **Seasonal Naïve**: repeats the last observed value from the previous season (e.g., `t - 24h` for hourly horizon).
- **LightGBM**: gradient boosting tree model on tabular lag features. If `lightgbm` is not available, the code will raise a clear import error.
- **LSTM**: sequence-to-one regression using PyTorch.

> **Informer**: Providing a compact, *from-scratch* Informer implementation is outside the scope of this public package. We include a well-documented placeholder with input/output signatures so the training loop remains stable. If readers want to use Informer, they can plug in an existing implementation respecting the same interface.

## Transfer mechanisms

- **CORAL (statistical alignment)**: feature covariance alignment layer callable before model fitting.
- **DANN (adversarial)** and **MAML (meta-learning)**: provided as stubs with API-ready signatures and comments describing expected behavior to match the manuscript (for transparency).

## Reproducibility

- Deterministic seeds are set in loaders and trainers where practical.
- Synthetic generator uses fixed procesess for diurnal/weekly seasonality and site-level shifts.

## License

Released under the MIT License (see `LICENSE`).

## Citation / Data Availability

Add the following to your manuscript under **Data Availability** (adapt wording to your journal style):

> **Data Availability:** The operational SCADA datasets used in this study are protected under a non‑disclosure agreement with the utility provider and cannot be shared. To support transparency and reproducibility, we release the full source code of the modeling and evaluation pipeline together with a synthetic dataset generator that mimics key statistical properties of the original data (diurnal/weekly seasonality and cross‑site covariate shift). The public code and synthetic data generator are available at: **[repository link / DOI to be added]**.


## Working with a YAML config

You can drive experiments with a single config file (recommended for clarity):

```bash
# generate synthetic data (once)
python -m src.data.make_synthetic --out ./experiments/synth --n_days 120 --n_sites 3

# run with config
python -m src.main --config ./experiments/config_example.yaml
```

Override values at the CLI if needed (nice for quick sweeps):
```bash
python -m src.main --config ./experiments/config_example.yaml --model lstm --epochs 10
```
