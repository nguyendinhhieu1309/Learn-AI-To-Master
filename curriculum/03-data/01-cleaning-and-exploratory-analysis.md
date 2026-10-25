# Clean and Explore a Tabular Dataset

## Learning objectives

Start from an unfamiliar table, find data quality problems, make defensible cleaning decisions, and summarize what the data can tell you.

## 1. Ask what each row means

Before plotting anything, write down the unit of observation: one row per person, order, sensor reading, or day? Identify the target (if any), the time range, the collection process, and the intended use. If each person has many rows, random row-level splitting may later put the same person in train and test data.

## 2. Inspect before changing

```python
import pandas as pd

df = pd.read_csv("data/example.csv")
print(df.shape)
print(df.head())
print(df.dtypes)
print(df.isna().sum().sort_values(ascending=False))
print(df.nunique(dropna=False))
```

Check unique identifiers, duplicates, units, impossible ranges, inconsistent spelling, and date parsing. A value outside the usual range is a question to investigate, not automatically an error.

## 3. Missing values

Missingness can mean “not measured,” “not applicable,” a failed sensor, or a value hidden for privacy. These meanings differ. Measure how often values are missing and whether missingness concentrates in a group or time period.

Possible choices include: retain a missing category; impute a numerical median or a categorical most-frequent value; add a missingness indicator; or drop a row/column when justified. For predictive evaluation, learn imputation values from the training set and apply those values unchanged to validation/test data. Otherwise, information from held-out data leaks into training.

## 4. Duplicates, types, and outliers

Exact duplicate rows may be accidental, but repeated rows may also be valid events. Check the key that should be unique before dropping duplicates. Parse dates and categories intentionally. For numerical outliers, use plots and domain rules; extreme real observations may be precisely the cases the model must handle.

Example: if a sensor's documented range is 0–100, a value of 999 might be an error code. If no such documented range exists, do not clip it just because it looks unusual.

## 5. Explore distributions and relationships

Use histograms or quantiles for numerical features, bar charts for categories, and scatter plots for pairs. Compare the target across relevant groups. Always inspect sample counts: a noisy mean from three examples should not be read like a mean from thousands.

Exploration suggests questions; it does not certify a causal explanation. If a feature is strongly related to the target, it might be a proxy, an artifact of how labels were collected, or a post-outcome variable.

## Worked cleaning example

```python
df["age"] = pd.to_numeric(df["age"], errors="coerce")
df["joined_at"] = pd.to_datetime(df["joined_at"], errors="coerce", utc=True)

invalid_age = df["age"].notna() & ~df["age"].between(0, 120)
print("Invalid ages:", int(invalid_age.sum()))
print("Missing age:", int(df["age"].isna().sum()))
```

This code flags questionable data; it does not silently delete it. Next, inspect the source and decide how to handle these rows, then document the rule.

## Practice

For any public dataset, make a one-page data audit with: row meaning, target, column types, missingness, duplicate/key checks, suspicious values, class balance (if applicable), and three plots. Add one question that the dataset cannot answer reliably.

## Common mistakes

- Dropping all rows with any missing value without measuring how many are lost.
- Replacing every unusual value with the median or a boundary.
- Plotting a relationship without checking sample size or collection bias.
- Cleaning labels based on what makes model accuracy look better.
