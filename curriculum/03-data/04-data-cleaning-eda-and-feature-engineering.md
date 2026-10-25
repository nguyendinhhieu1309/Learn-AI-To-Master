# Data Cleaning, EDA, and Feature Engineering

Good models start with a trustworthy table. Exploratory data analysis (EDA) is a disciplined way to understand the data, detect defects, and form hypotheses without leaking test information into training.

## 1. Establish the unit and schema

Before cleaning, state what one row represents, what each column means, the key, the time at which a prediction is made, and how labels were assigned. Check row counts, duplicate keys, dtypes, units, timezones, and category spellings. A numeric ID is not necessarily a numeric measurement.

## 2. Missing values

Missingness can be MCAR (unrelated to observed or unobserved values), MAR (explained by observed variables), or MNAR (related to the missing value itself). The mechanism affects both modeling and interpretation.

Inspect missingness by label, time, source, and cohort. Options include imputation, an explicit missing indicator, model-native missing handling, or correcting the collection process. Fit imputation values on training data only and apply the same fitted transformation to validation and production data.

## 3. Duplicates, invalid values, and outliers

Deduplicate using the real entity/event key, not by deleting all repeated feature rows. Repeated measurements can be valid. Validate constraints such as nonnegative quantities, plausible dates, and mutually compatible fields.

Outliers may be errors, rare valid cases, or the most important cases. Plot distributions and inspect examples before clipping or removing them. If using clipping or robust scaling, estimate bounds on training data and document the policy.

## 4. EDA without test leakage

Use training data for feature exploration and transformation design. Validation data may guide choices, but extensive tuning can overfit it; reserve the test set for final evaluation. Compare target and feature distributions by split to spot accidental leakage or population mismatch.

Useful views include histograms, quantiles, missingness maps, pair plots for small feature sets, class counts, time trends, and target rates by category. Each chart should answer a question. Aggregate patterns can hide subgroup failures, so inspect meaningful slices.

## 5. Categorical and numerical features

- One-hot encoding is a strong baseline for low-cardinality nominal categories.
- Ordinal encoding is valid only when order is meaningful and understood.
- Frequency or target encodings need careful cross-fitting to avoid leaking labels.
- Log transforms can help with positive, right-skewed features.
- Standardization is common for distance-based and regularized linear models; tree splits usually do not require it.
- Cyclical variables such as hour-of-day can use sine/cosine features when wraparound matters.

For a cycle of period $T$, encode value $x$ as:

$$
\left(\sin\frac{2\pi x}{T},\;\cos\frac{2\pi x}{T}\right)
$$

This places the first and last values of the cycle next to each other geometrically.

## 6. Avoid target and time leakage

Ask whether each feature exists at the decision time. Aggregates must use only events that have already happened. For example, “total purchases this month” leaks future data for a prediction made mid-month unless computed only up to that date. Use point-in-time joins for historical features.

## 7. Build a reproducible preprocessing pipeline

Fit learned transforms on training folds only and package them with the model. Keep feature names and order stable. Add checks for missing/unexpected columns, units, valid ranges, and output shape. Do not manually repeat preprocessing in a notebook and production service.

## 8. Exercises

1. A feature is missing more often for users who churn. Why might median imputation alone hide useful information?
2. A database has multiple records per customer and the target is future churn. What should the split unit be?
3. Why can target encoding leak information when calculated once over the entire training table before cross-validation?
4. A date-of-week variable is encoded as integers 0 through 6. Why might sine/cosine features help?

## References

- [pandas: Getting started](https://pandas.pydata.org/docs/getting_started/index.html)
- [scikit-learn: Preprocessing data](https://scikit-learn.org/stable/modules/preprocessing.html)
