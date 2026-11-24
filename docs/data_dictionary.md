# Data dictionary

## Conventions

Each data field should consist of name, type, unit, valid range, generation logic, and rationale.

## Data dictionary

| Column name | Column data type | Data unit | Valid range | Generation logic |
|-------------|------------------|-----------|-------------|------------------|
|user_agent|object|NaN|1 of 6 user agents<sup>1|Different distributions for fraud and non-fraud|
|orig_account_age|int|days|1-1200|Uniform distribution for non-fraud, skewed towards 1 for fraud <br> Consistent to account (one account assigned one age)|
|dest_account_age|int|days|1-1200|Same logic and consistent with orig_account_age|
|address_mismatch|int|NaN|boolean 1/0 for yes/no|Generate randomly per account at 5% for non fraud and 13% for fraud <br> Flag if either orig or dest hits 1|
|failed_attempts|int|NaN|0-6|Sample from distribution with mean = 0.009 to simulate 0.9% fail rate normally <br> Up to mean = 0.012 to simulate potential existing antifraud/irregularities in fraud transactions|
|distance_to_billing|float64|km|0-12000|Most transactions local => 0 <br> Some non-fraud transactions are to neighbouring countries around 750-2000km, very few if any >2000 <br> Fraud transactions have much higher occurance of higher values|

<sup>*1: User agents: Android, iOS, Windows, MacOS, Linux, Others*

## Rationale


