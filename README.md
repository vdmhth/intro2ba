# Real-Time Payment Fraud Detection for an E-Commerce Platform

Group project for the Business Analytics course. The project develops a classification model that flags potentially fraudulent payment transactions, deploys it as a real-time scoring service with an analyst review interface, and monitors its behaviour after deployment. The project requirements are described in the [course guideline](docs/guideline/Ecommerce_Fraud_Detection_Project_Announcement.pdf).

## Repository structure

| Path | Content | Responsible role | Module |
|---|---|---|---|
| `docs/guideline/` | Course guideline | All | |
| `docs/data_dictionary.md` | Dictionary of synthetic fields: name, type, unit, valid range, generation logic | DE | 1 |
| `docs/feature_spec_da.md` | Features proposed from the exploratory analysis | DA | 2, 4 |
| `docs/report/` | Report sections prepared by each role | All | 8 |
| `data/raw/` | Original Kaggle data and its parquet conversion | DE | 1 |
| `data/interim/` | Synthetic fields, merged data and cleaned data | DE | 1, 3 |
| `data/processed/` | Training, validation and test sets | DE | 3 |
| `data/stream/` | Stream period used for replay, including drift scenarios | DE, MLOps | 7 |
| `notebooks/data_generation/` | Development and checks of the data generator | DE | 1 |
| `notebooks/eda/` | Exploratory analysis of the Kaggle data and of the merged data | DA | 2 |
| `notebooks/cleaning/` | Data cleaning with before and after comparisons | DE | 3 |
| `notebooks/features/` | Feature construction and selection | Modeling | 4 |
| `notebooks/modeling_evaluation/` | Model training, class imbalance handling, threshold selection and evaluation | Modeling, Evaluation | 4, 5 |
| `notebooks/monitoring/` | Drift and performance analysis | MLOps | 7 |
| `src/fraud/data/` | Data generation, cleaning and splitting | DE | 1, 3 |
| `src/fraud/features/` | Feature functions based on past transactions only | Modeling | 4 |
| `src/fraud/models/` | Shared training function | Modeling | 5 |
| `src/fraud/evaluation/` | Metrics, cost functions and threshold selection | Evaluation | 5 |
| `src/fraud/monitoring/` | Drift detection and retraining triggers | MLOps | 7 |
| `app/api/` | FastAPI scoring service | MLOps | 6 |
| `app/dashboard/` | Streamlit review queue for fraud analysts | MLOps | 6 |
| `models/` | Trained model, decision thresholds and baseline metrics | Modeling, Evaluation | 5 |
| `scripts/` | Command-line scripts, e.g. data generation and stream replay | All | |
| `tests/` | API consistency test on a reference set and unit tests | Evaluation, MLOps | 6 |
| `reports/figures/eda/kaggle/` | Figures from the analysis of the original Kaggle data | DA | 2 |
| `reports/figures/eda/merged/` | Figures from the analysis of the data after synthetic generation | DA | 2 |
| `reports/monitoring/` | Evidently monitoring reports | MLOps | 7 |
| `reports/final/` | Final report (PDF) and presentation slides (PPTX) | All | 8 |

## Conventions

- Notebooks are used for exploration and documentation. Code shared between roles or used by the API is placed in `src/fraud/`.
- Files are named by content, for example `eda_kaggle.ipynb`, `eda_merged.ipynb` and `generator.py`.
- The `data/` directory is not versioned because the files exceed the GitHub size limit. Each member places the Kaggle data in `data/raw/`.
- Code, documentation and notebooks are written in English.

## Setup

Install the dependencies:

```bash
pip install -r requirements.txt
```

Download the [Online Payments Fraud Detection Dataset](https://www.kaggle.com/datasets/rupakroy/online-payments-fraud-detection-dataset) and place `kaggle dataset.zip`, or the extracted `PS_20174392719_1491204439457_log.csv`, in `data/raw/`.

Notebooks can be executed from the command line:

```bash
python -m nbconvert --to notebook --execute --inplace notebooks/eda/eda_kaggle.ipynb
```
