# Feature Suggestions

This note lists the features proposed on the basis of the exploratory analysis of the Kaggle data (`notebooks/eda/eda_kaggle.ipynb`). All reported figures refer to TRANSFER and CASH_OUT transactions.

## Proposed features

| Feature | Definition | Justification |
|---|---|---|
| `amount_log` | `log(1 + amount)` | The fraud rate increases with transaction amount, from about 0.15% in the middle range to 2% in the top 5%. The logarithm reduces the influence of extreme values. |
| `hour` | `step % 24`, assuming step 0 corresponds to midnight | The fraud rate reaches 16–22% between 3:00 and 6:00, compared with less than 0.1% during the day. |
| `hour_sin`, `hour_cos` | `sin(2π · hour / 24)`, `cos(2π · hour / 24)` | Cyclical encoding allows linear models to treat 23:00 and 0:00 as adjacent. Tree-based models may use `hour` directly. |
| `is_night` (optional) | 1 if `hour` is between 2 and 7, otherwise 0 | A simpler alternative to the cyclical encoding, covering the hours with a fraud rate above 2%. The interval should be re-estimated on the training set only. |
| `is_transfer` | 1 if `type` is TRANSFER, otherwise 0 | The fraud rate is 0.77% for TRANSFER and 0.18% for CASH_OUT. If all transaction types are used for training, `type` should be one-hot encoded instead. |
| `dest_prior_count` | Number of earlier transactions received by the same `nameDest` | Recipients in fraudulent transactions had received on average 2.7 earlier transactions, against 7.7 for legitimate ones. |
| `is_new_dest` | 1 if `dest_prior_count` equals 0, otherwise 0 | 63% of fraudulent transactions go to a first-time recipient, against 14% of legitimate ones. |
| `dest_steps_since_last` | Number of steps since the same `nameDest` last received a transaction; missing if none | The strongest single feature in the analysis (AUC 0.82). Missing values can be kept as NaN for XGBoost and LightGBM, and replaced by a large constant (e.g. 9999) for other models. |

The three recipient-based features are strongly related; feature selection is expected to retain one or two of them.

## Computation of history-based features

History-based features must use only information available at the time of the transaction:

- transactions are ordered by `step`, and by original row order within the same step;
- only earlier transactions are used;
- the history is computed over the full timeline rather than separately for each split, so that validation and test transactions retain the history of the training period;
- all transaction types are included in the history, even if the model is trained on TRANSFER and CASH_OUT only.

```python
df["dest_prior_count"] = df.groupby("nameDest").cumcount()
df["is_new_dest"] = (df["dest_prior_count"] == 0).astype(int)
df["dest_steps_since_last"] = df["step"] - df.groupby("nameDest")["step"].shift(1)
```

In deployment, the scoring API must store for each recipient the number of transactions received and the step of the most recent one.

## Excluded variables

- **Balance columns** (`oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, `newbalanceDest`) and any derived variable. They leak the label: 97.8% of fraudulent transactions empty the sender's account exactly, which almost never occurs in legitimate transactions (AUC 0.989 on its own). According to the PaySim description, fraudulent transactions were cancelled, so their balances are not reliable.
- **`step`**, which is used only for the time-based split.
- **`isFlaggedFraud`**, which is triggered only 16 times and is retained as a baseline only.
- **`nameOrig`**, as almost every value is unique.
- **Aggregates computed with labels or over the full dataset**, such as the fraud rate per recipient.

## Limitations

- No fraudulent transaction exceeds 10,000,000 because of a simulation limit, so tree-based models may learn that larger amounts are safe.
- The high fraud rate at night is partly a simulation effect, as legitimate activity at night is very low.

## Planned additions

After the analysis of the merged dataset, customer-level features will be added once `customer_id` is available: number of transactions in the previous 24 steps, time since the customer's previous transaction, new-device indicator, and amount relative to the customer's usual amount. Synthetic contextual fields will be included if they pass the realism and leakage checks.
