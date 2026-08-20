# Jane Street Real-Time Market Data Forecasting

A leakage-aware, reproducible research project for the Jane Street Real-Time Market Data Forecasting competition. The modeling target is `responder_6`, evaluated with the competition's sample-weighted, zero-mean \(R^2\) metric.

This repository currently contains the research contract and structural data audit only. Exploratory analysis and model training will be added in later phases after the data assumptions are verified.

## Project Status

| Phase | Scope | Status |
|---|---|---|
| Phase 0 | Research contract, target, metric, and leakage rules | Complete |
| Phase 1 | Data inventory and structural validation | Complete |
| Phase 2 | Missingness and feature availability analysis | Not published yet |
| Phase 3+ | EDA, time-based validation, baselines, and modeling | Planned |

## Prediction Contract

- **Target:** predict `responder_6`.
- **Unit of observation:** one anonymized instrument (`symbol_id`) at one `date_id` and `time_id`.
- **Metric:** sample-weighted, zero-mean \(R^2\):

$$
R^2 = 1 - \frac{\sum_i w_i(y_i-\hat{y}_i)^2}{\sum_i w_i y_i^2}
$$

- **Information rule:** use only information available when the prediction is made.
- **Validation rule:** train on earlier dates, validate on later dates, and preserve the most recent period as a sealed holdout.

## Verified Phase 1 Findings

- 10 Parquet partitions.
- 47,127,338 training rows.
- Approximately 11.45 GiB on disk.
- 92 columns: 4 identifier/weight columns, 79 features, and 9 responders.
- Identical Arrow schema across all partitions.
- Continuous `date_id` coverage from 0 through 1698, with no partition gaps or overlaps.
- No duplicate `(date_id, time_id, symbol_id)` keys within any partition.
- Rows are monotonically ordered by `date_id` and then `time_id` in every partition.

These checks support partition-wise, column-selective processing and chronological validation in later phases.

## Repository Structure

```text
.
├── notebooks/
│   └── 01_research_contract_and_data_inventory.ipynb
├── .gitignore
├── README.md
└── requirements.txt
```

The original competition data, local working notebooks, presentations, and unfinished EDA are intentionally excluded from version control.

## Setup

This project was prepared with Python 3.12.4.

```bash
python -m venv .venv
```

Activate the environment, then install the dependencies:

```bash
python -m pip install -r requirements.txt
```

## Data

Download the data from the [official Kaggle competition page](https://www.kaggle.com/competitions/jane-street-real-time-market-data-forecasting). Competition data is not redistributed in this repository.

Place the partitioned training dataset at the repository root with this layout:

```text
train.parquet/
├── partition_id=0/part-0.parquet
├── partition_id=1/part-1.parquet
├── ...
└── partition_id=9/part-9.parquet
```

## Run the Notebook

Start Jupyter from the repository root:

```bash
jupyter lab
```

Then open `notebooks/01_research_contract_and_data_inventory.ipynb` and run the cells in order. The audit reads Parquet metadata or selected columns instead of loading the entire dataset into memory.

## Research Principles

- Treat time order as part of the problem definition.
- Prevent target leakage and future-fitted preprocessing.
- Fit every learned transformation on training dates only.
- Compare models with the exact weighted competition metric.
- Keep the final holdout sealed until model-selection decisions are complete.

## Next Step

Phase 2 will measure missingness and feature availability over time before any imputation strategy or predictive model is selected.

## Disclaimer

This is an educational research project and not financial advice. Jane Street and Kaggle retain their respective rights to the competition and dataset.
