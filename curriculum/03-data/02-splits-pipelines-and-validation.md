# Splits, Pipelines, and Data Validation

## Learning objectives

Create an evaluation split that resembles future use, keep learned preprocessing inside the training boundary, and express data quality assumptions as checks.

## 1. Why split data?

Training data is used to fit model parameters. Validation data helps select models and settings. A final test set estimates performance after decisions are complete. Reusing the test set for many choices gradually turns it into training information.

Use a random split only when examples are independent and deployment resembles random sampling. Use a **group split** when multiple rows belong to the same person/device/site and groups must not cross splits. Use a **time split** when predicting the future from the past. Use stratification to preserve class proportions when suitable, but never let it override a more important time or group constraint.

## 2. Data leakage

Leakage occurs when training uses information unavailable at the real prediction time, or information from held-out examples. Examples:

- scaling all rows before the split;
- placing future events into a feature for a past prediction;
- splitting duplicate people across train and test;
- selecting features after examining test-set performance;
- a label-derived field accidentally left in the input.

Ask: “At the exact time this prediction is needed, would this field already exist?”

## 3. Fit transforms on training data only

```python
from sklearn.compose import ColumnTransformer
from sklearn.impute import SimpleImputer
from sklearn.pipeline import make_pipeline
from sklearn.preprocessing import OneHotEncoder, StandardScaler
from sklearn.linear_model import LogisticRegression

numeric = ["age", "income"]
categorical = ["region"]

prep = ColumnTransformer([
    ("num", make_pipeline(SimpleImputer(strategy="median"), StandardScaler()), numeric),
    ("cat", make_pipeline(SimpleImputer(strategy="most_frequent"),
                           OneHotEncoder(handle_unknown="ignore")), categorical),
])
model = make_pipeline(prep, LogisticRegression(max_iter=1000))
model.fit(X_train, y_train)       # preprocessing learns only from training data
predictions = model.predict(X_test)
```

The pipeline keeps preprocessing and model training together. During cross-validation, the whole pipeline is refit separately within each fold.

## 4. Validation by constraints

Write down simple rules that should always hold: required columns exist, IDs are unique where expected, dates parse, numeric ranges are valid, category sets are known (or unknowns are handled), and target labels are allowed. These checks catch upstream changes before they silently change a result.

Validation should distinguish **hard errors** (a required target column disappeared) from **warnings** (the fraction of missing income rose). A warning should trigger inspection, not automatic deletion.

## 5. Reproducibility and provenance

Record the source URL/version, access date if relevant, license, filters, transformation steps, split rule, random seed, and output schema. A random seed helps reproduce a pseudorandom split, but does not guarantee identical results across hardware or library versions.

## Practice with answer guide

You predict next month's demand using historical daily records. A random split may train on later months and test on earlier ones, making the task easier than forecasting. Prefer a chronological split: train on earlier dates, validate on a later window, and test on the latest untouched window. Derive rolling statistics using only past rows.

## Common mistakes

- Using random splits for repeated users, devices, or time series without checking dependence.
- Fitting scaling or imputation on all data before cross-validation.
- Deleting examples that violate assumptions without reporting the count or reason.
- Confusing reproducibility with correctness: a repeatable leak is still a leak.
