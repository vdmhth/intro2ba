# Feature Suggestions

This note lists the features proposed on the basis of the exploratory analysis of the Kaggle data (`notebooks/eda/eda_kaggle.ipynb`). All figures refer to TRANSFER and CASH_OUT transactions within steps 1 to 281, the earliest 60% of the data, which lies inside the training set under both split options considered. They are exploratory statistics, not estimates of model performance; feature choices should be confirmed on the validation set.

## Proposed features

| Feature | Definition | Justification |
|---|---|---|
| `amount_log` | `log1p(amount)`, i.e. `log(1 + amount)` | The median fraud (412,511) is 2.4 times the median legitimate transaction (170,498), and amounts above the usual outlier bound have a fraud rate of 1.08% against 0.12% otherwise. The logarithm reduces the influence of extreme values. |
| `hour` | `step % 24`, assuming step 0 corresponds to midnight | The fraud rate is 28.6–58.5% between 3:00 and 6:00, compared with 0.06–0.21% between 9:00 and 20:00. |
| `hour_sin`, `hour_cos` | `sin(2π · hour / 24)`, `cos(2π · hour / 24)` | Cyclical encoding allows linear models to treat 23:00 and 0:00 as adjacent. Tree-based models may use `hour` directly. |
| `is_night` (optional) | 1 if `hour` is between 1 and 7, otherwise 0 | A simpler alternative to the cyclical encoding, covering the hours with a fraud rate of at least 2%. |
| `is_transfer` | 1 if `type` is TRANSFER, otherwise 0 | The fraud rate is 0.50% for TRANSFER and 0.12% for CASH_OUT. If all transaction types are used for training, `type` should be one-hot encoded instead. |
| `amount_log_x_transfer` (linear models only) | `amount_log * is_transfer` | The effect of amount depends on the type. For CASH_OUT the fraud rate is 0.04–0.11% in the first nine amount deciles and 2.5% in the top decile; for TRANSFER it is highest for the smallest amounts (1.2%) and decreases as the amount grows. Tree-based models capture this interaction without an explicit term. |
| `dest_prior_count` | Number of earlier transactions received by the same `nameDest` | The fraud rate falls from 0.85% for recipients with no earlier transaction to 0.12% for one earlier transaction and 0.05% for twenty or more (AUC 0.77). |
| `is_new_dest` | 1 if `dest_prior_count` equals 0, otherwise 0 | 62.8% of fraudulent transactions go to a first-time recipient, against 13.9% of legitimate ones (AUC 0.74). |
| `dest_steps_since_last` | Number of steps since the same `nameDest` last received a transaction; missing if none | The highest single-feature AUC among the usable candidates examined (0.79). Missing values can be kept as NaN for XGBoost and LightGBM, and replaced by a large constant (e.g. 9999) for other models. |

The three recipient-based features are moderately correlated (absolute Spearman correlation 0.41 to 0.60). Correlation alone does not show that any of them is redundant; whether one or two are enough should be tested on the validation set.

## Computation of history-based features

History-based features must use only information available at the time of the transaction:

- transactions are ordered by `step`, and by original row order within the same step (the code below assumes this order, so check `df["step"].is_monotonic_increasing` first);
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

- **Balance columns** (`oldbalanceOrg`, `newbalanceOrig`, `oldbalanceDest`, `newbalanceDest`) and any derived variable, for two different reasons:
  - The post-transaction balances (`newbalanceOrig`, `newbalanceDest`) are not available when a transaction is scored, since they describe its outcome. Using them would be a temporal leak.
  - The pre-transaction balances (`oldbalanceOrg`, `oldbalanceDest`) are available at scoring time but follow simulator rules rather than observable behaviour. 97.9% of frauds transfer exactly the sender balance, against 0.0% of legitimate transactions, because the simulated fraud agent always empties the account. Legitimate balances are often implausible: 47.6% start from a sender balance of zero although money is sent, and only 9.6% add up, against 99.0% of frauds.

  The dataset description also states that fraudulent transactions are cancelled and that these columns must not be used for fraud detection. A comparison model with the pre-transaction sender balance (e.g. `amount / oldbalanceOrg`) may be reported to show how much performance would depend on this artefact.
- **`step`**, which is not used directly as a model input; it is used for the time-based split and to derive `hour` and the history features.
- **`isFlaggedFraud`**, which is triggered only 16 times in the full data and is retained as a baseline only.
- **`nameOrig`**, as almost every value is unique.
- **Aggregates computed with labels or over the full dataset**, such as the fraud rate per recipient.

## Limitations

- No fraudulent transaction exceeds 10,000,000 because of a simulation limit, so tree-based models may learn that larger amounts are safe.
- The high fraud rate at night is partly a simulation effect, as legitimate activity at night is very low.

## Planned additions

After the analysis of the merged dataset, customer-level features will be added once `customer_id` is available: number of transactions in the previous 24 steps, time since the customer's previous transaction, new-device indicator, and amount relative to the customer's usual amount. Synthetic contextual fields will be included if they pass the realism and leakage checks.
