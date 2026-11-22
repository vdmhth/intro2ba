# Real-Time Payment Fraud Detection for an E-Commerce Platform

Group project for the Business Analytics course. The project develops a classification model that flags potentially fraudulent payment transactions, deploys it as a real-time scoring service with an analyst review interface, and monitors its behaviour after deployment. The project requirements are described in the [course guideline](docs/guideline/Ecommerce_Fraud_Detection_Project_Announcement.pdf).

## Repository structure

```
intro2ba/
├── app/
│   ├── api/                                            # FastAPI scoring service
│   └── dashboard/                                      # Streamlit review queue for fraud analysts
├── data/                                               # not versioned
│   ├── interim/                                        # synthetic fields, merged data and cleaned data
│   ├── processed/                                      # training, validation and test sets
│   ├── raw/                                            # original Kaggle data and its parquet conversion
│   └── stream/                                         # stream period used for replay, including drift scenarios
├── docs/
│   ├── guideline/
│   │   └── Ecommerce_Fraud_Detection_Project_Announcement.pdf   # course guideline
│   ├── report/                                         # report sections prepared by each member
│   └── feature_spec_da.md                              # features proposed from the exploratory analysis
├── models/                                             # trained model, decision thresholds and baseline metrics
├── notebooks/
│   ├── cleaning/                                       # data cleaning with before and after comparisons
│   ├── data_generation/                                # development and checks of the data generator
│   ├── eda/
│   │   └── eda_kaggle.ipynb                            # exploratory analysis of the original Kaggle data
│   ├── features/                                       # feature construction and selection
│   ├── modeling_evaluation/                            # training, class imbalance handling, threshold selection, evaluation
│   └── monitoring/                                     # drift and performance analysis
├── reports/
│   ├── figures/
│   │   └── eda/
│   │       ├── kaggle/                                 # figures from the analysis of the original Kaggle data
│   │       │   ├── amount_distribution.png             # amount distribution of fraudulent and legitimate transactions
│   │       │   ├── balance_leakage.png                 # balance patterns that reveal the label
│   │       │   ├── feature_auc.png                     # single-feature AUC of candidate and excluded features
│   │       │   ├── feature_correlation.png             # Spearman correlation between candidate features
│   │       │   ├── fraud_rate_by_amount_and_type.png   # fraud rate by amount decile for TRANSFER and CASH_OUT
│   │       │   ├── fraud_rate_by_recipient_history.png # fraud rate by number of earlier transactions to the recipient
│   │       │   ├── fraud_value_concentration.png       # cumulative share of fraud value by case
│   │       │   ├── split_cumulative_share.png          # cumulative transactions and frauds by step, with split points
│   │       │   ├── type_volume_fraud_rate.png          # volume and fraud rate by transaction type
│   │       │   ├── volume_fraud_by_day.png             # legitimate volume, frauds and fraud rate by day
│   │       │   └── volume_fraud_by_hour.png            # legitimate volume, frauds and fraud rate by hour of day
│   │       └── merged/                                 # figures from the analysis of the data after synthetic generation
│   ├── final/                                          # final report (PDF) and presentation slides (PPTX)
│   └── monitoring/                                     # Evidently monitoring reports
├── scripts/                                            # command-line scripts, e.g. data generation and stream replay
├── src/
│   └── fraud/
│       ├── data/                                       # data generation, cleaning and splitting
│       ├── evaluation/                                 # metrics, cost functions and threshold selection
│       ├── features/                                   # feature functions based on past transactions only
│       ├── models/                                     # shared training function
│       └── monitoring/                                 # drift detection and retraining triggers
├── tests/                                              # API consistency test on a reference set and unit tests
├── .gitignore                                          # files excluded from version control
├── README.md                                           # project overview and repository structure
└── requirements.txt                                    # Python dependencies
```

Empty directories contain a `.gitkeep` file so that they are kept in version control.

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
