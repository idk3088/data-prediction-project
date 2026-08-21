# Data Prediction Project

A reusable foundation for tabular data prediction projects.

The project separates data-contract decisions, structural validation, and later modeling work. It is designed for local datasets and intentionally contains no raw data, sample rows, or executed notebook outputs.

## What This Project Covers

- Define a prediction target, evaluation rule, and information-availability assumptions.
- Discover partitioned Parquet files without loading the full dataset into memory.
- Inspect file metadata, row groups, schemas, dtypes, and on-disk sizes.
- Validate key uniqueness and chronological ordering using configurable columns.
- Prepare a clean starting point for time-aware validation and predictive modeling.

## Repository Structure

```text
.
├── notebooks/
│   └── 01_data_contract_and_validation.ipynb
├── .gitignore
├── README.md
└── requirements.txt
```

## Setup

```bash
python -m venv .venv
python -m pip install -r requirements.txt
jupyter lab
```

Place local input files under `data/`, then open the notebook and update the configuration cell to match the dataset schema.

## Principles

- Keep raw data outside version control.
- Inspect structural assumptions before feature engineering or modeling.
- Use only information available at prediction time.
- Use chronological validation whenever record order carries predictive meaning.
- Keep notebook outputs compact and reproducible.
