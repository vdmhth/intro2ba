# Real-Time Payment Fraud Detection for an E-Commerce Platform

Group project for the Business Analytics course. The project develops a classification model that flags potentially fraudulent payment transactions, deploys it as a real-time scoring service with an analyst review interface, and monitors its behaviour after deployment. The project requirements are described in the [course guideline](docs/guideline/Ecommerce_Fraud_Detection_Project_Announcement.pdf).

## Repository structure

```
intro2ba/
├── docs/
│   ├── guideline/                 # course guideline
│   ├── report/                    # report sections prepared by each member
│   ├── data_dictionary.md         # synthetic fields: name, type, unit, valid range, generation logic
│   └── feature_spec_da.md         # features proposed from the exploratory analysis
├── data/                          # not versioned
│   ├── raw/                       # original Kaggle data and its parquet conversion
│   ├── interim/                   # synthetic fields, merged data and cleaned data
│   ├── processed/                 # training, validation and test sets
│   └── stream/                    # stream period used for replay, including drift scenarios
├── notebooks/
│   ├── data_generation/           # development and checks of the data generator
│   ├── eda/                       # exploratory analysis of the Kaggle data and of the merged data
│   ├── cleaning/                  # data cleaning with before and after comparisons
│   ├── features/                  # feature construction and selection
│   ├── modeling_evaluation/       # training, class imbalance handling, threshold selection, evaluation
│   └── monitoring/                # drift and performance analysis
├── src/fraud/
│   ├── data/                      # data generation, cleaning and splitting
│   ├── features/                  # feature functions based on past transactions only
│   ├── models/                    # shared training function
│   ├── evaluation/                # metrics, cost functions and threshold selection
│   └── monitoring/                # drift detection and retraining triggers
├── app/
│   ├── api/                       # FastAPI scoring service
│   └── dashboard/                 # Streamlit review queue for fraud analysts
├── models/                        # trained model, decision thresholds and baseline metrics
├── scripts/                       # command-line scripts, e.g. data generation and stream replay
├── tests/                         # API consistency test on a reference set and unit tests
├── reports/
│   ├── figures/eda/kaggle/        # figures from the analysis of the original Kaggle data
│   ├── figures/eda/merged/        # figures from the analysis of the data after synthetic generation
│   ├── monitoring/                # Evidently monitoring reports
│   └── final/                     # final report (PDF) and presentation slides (PPTX)
└── requirements.txt               # Python dependencies
```

## Conventions

- Notebooks are used for exploration and documentation. Code shared between components or used by the API is placed in `src/fraud/`.
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
