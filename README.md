# Jane Street Real-Time Market Data Forecasting

A leakage-aware, time-ordered research pipeline for the Jane Street Real-Time Market Data Forecasting competition. The project predicts `responder_6` and evaluates predictions with the competition's sample-weighted, zero-mean \(R^2\).

The repository now contains the complete research notebook, from data validation and EDA through chronological LightGBM experiments, multi-responder tests, regularization, stacking, cross-sectional features, online updating, and the final locked Phase 8D comparison.

## Final Research Result

The accepted Phase 8D pipeline combines:

- a cross-sectional LightGBM ensemble with multiple seeds and tree depths;
- a no-intercept meta-model fitted only on earlier out-of-fold predictions;
- a 10-date online update interval using previously revealed labels;
- leakage-safe calibration fitted only on prior dates.

| Evaluation | Weighted zero-mean \(R^2\) |
|---|---:|
| Development A-C, selected online rule | 0.011395 |
| Late-period Fold D | 0.009634 |
| Late-period Fold E | 0.010263 |
| **Late-period D-E mean** | **0.009949** |
| Previous Phase 8C D-E mean | 0.009884 |

Phase 8D improved both the mean and worst-fold D-E score relative to Phase 8C and was therefore accepted under the predeclared rule. The A-C result is a development score, not the final generalization estimate. D-E had already been inspected in earlier phases, so the D-E comparison is reported as descriptive confirmation rather than a pristine unbiased test. Partition 9 remains unused by Phase 8D.

The saved decision and fold-level results are available in [`results/phase8d`](results/phase8d).

## Prediction Contract

- **Target:** `responder_6`.
- **Observation:** one anonymized instrument (`symbol_id`) at one `date_id` and `time_id`.
- **Metric:**

$$
R^2 = 1 - \frac{\sum_i w_i(y_i-\hat{y}_i)^2}{\sum_i w_i y_i^2}
$$

- **Information rule:** current and future responders are unavailable at prediction time. Online updates use only labels already revealed from earlier dates.
- **Validation rule:** train on earlier dates, validate on later dates, preserve chronological order, and fit every learned transformation inside the relevant training period.

## Project Workflow

| Phase | Scope | Status |
|---|---|---|
| 0-2 | Research contract, inventory, integrity, missingness | Complete |
| 3 | Time-stratified EDA and stability analysis | Complete |
| 4 | Chronological LightGBM baseline and feature screening | Complete |
| 5 | Leakage-safe feature engineering | Complete |
| 6 | Regime, volatility, window, and ensemble analysis | Complete |
| 7-7B | PCA, multi-responder learning, online validation, regularization | Complete |
| 8-8D | Stacking, cross-sectional context, recency, combined and final locked pipeline | Complete |

## Main Findings

- The training set contains 47,127,338 rows across 10 chronological Parquet partitions.
- Feature availability changes over time; columns that are entirely missing in early partitions can become available later and must not be removed using one-partition evidence.
- The panel is unbalanced across symbols and dates, while every observed symbol-date pair has a complete intraday grid.
- The intraday grid expands from 849 to 968 time steps beginning at `date_id == 677`.
- Sample weights are finite and strictly positive, and their distribution changes chronologically.
- Chronological validation is materially different from random row splitting and is required throughout the project.
- Shallower and diversified LightGBM components, OOF stacking, and prior-label online adaptation improve stability, but performance still varies across time regimes.

## Repository Structure

```text
.
├── notebooks/
│   ├── 01_data_contract_and_validation.ipynb
│   ├── 01_research_contract_and_data_inventory.ipynb
│   └── 02_full_research_and_modeling_pipeline.ipynb
├── results/
│   └── phase8d/
├── .gitignore
├── README.md
└── requirements.txt
```

The final notebook retains its executed outputs so the reported analysis can be reviewed without rerunning the full 47-million-row workflow. Personal filesystem paths in the public copy have been replaced with `<PROJECT_ROOT>`.

The earlier `01_data_contract_and_validation.ipynb` is retained as the repository's original reusable validation template; the Jane Street-specific research is contained in the other two notebooks.

## Setup

The reported notebook was run with Python 3.12.4.

```bash
python -m venv .venv
```

Activate the environment and install the dependencies:

```bash
python -m pip install -r requirements.txt
```

## Data

Download the data from the [official Kaggle competition page](https://www.kaggle.com/competitions/jane-street-real-time-market-data-forecasting). Competition data is not redistributed here.

Place the partitioned training data at the repository root:

```text
train.parquet/
├── partition_id=0/part-0.parquet
├── partition_id=1/part-1.parquet
├── ...
└── partition_id=9/part-9.parquet
```

## Running the Project

Start Jupyter from the repository root:

```bash
jupyter lab
```

Open [`notebooks/02_full_research_and_modeling_pipeline.ipynb`](notebooks/02_full_research_and_modeling_pipeline.ipynb). The notebook uses `Path.cwd()` as the project root, so Jupyter should be started from this repository.

The notebook contains explicit `RUN_...` switches around expensive one-time computations. They are saved as `False` to prevent accidental multi-hour reruns. For a full recomputation:

1. run the notebook in chronological order;
2. enable each documented switch only for its corresponding cell;
3. run that cell once and save its artifact;
4. return the switch to `False` before continuing;
5. do not use later confirmation periods to retune an earlier selection.

The Phase 8D run-order section in the notebook gives the exact dependency order for the final pipeline.

## Reproducibility and Leakage Controls

- No random row split is used for model selection.
- Feature selection, calibration, PCA, stacking weights, and model fitting use training dates only.
- OOF meta-models are trained on earlier folds and evaluated on the next chronological fold.
- Lagged or rolling information is grouped by `symbol_id` and ordered by `date_id`, then `time_id`.
- Online updating assumes that prior-date responder labels have been revealed; this assumption must hold in the deployment environment.
- The final Phase 8D comparison does not use partition 9.

## Limitations

- D-E confirmation is descriptive because those periods had been examined in earlier research phases.
- The final online pipeline does not yet save a strictly comparable end-to-end training \(R^2\); component training scores and temporal validation scores are available, but a single train-validation spread would require rerunning the locked pipeline with additional diagnostics.
- Reproducing all experiments requires substantial memory, disk space, and runtime.
- The online improvement depends on access to previously revealed labels.

## Disclaimer

This is an educational research project, not financial advice. Jane Street and Kaggle retain their respective rights to the competition and dataset.
