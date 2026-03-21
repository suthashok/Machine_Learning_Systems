# Exploratory Data Analysis

## Goal

This doc captures the main things worth checking before feature engineering.

The purpose of EDA here is not to describe every column. It is to answer a few practical questions:

- how imbalanced is fraud?
- where does fraud concentrate?
- what looks useful for feature engineering?
- what data issues do we need to account for?

---

## What we want to learn first

For this dataset, the first pass of EDA should focus on:

1. class imbalance
2. missingness
3. amount patterns
4. entity concentration
5. time patterns

These are the main things that usually shape fraud features and validation strategy.

---

## 1. Class imbalance

Fraud is a small minority class in this dataset.

That has a few direct implications:
- raw accuracy is not useful
- we need to care more about ranking, separation, and calibration
- feature engineering should focus on finding concentrated pockets of risk, not global averages

This also means we should expect a lot of features to look weak in isolation.

---

## 2. Missingness

A lot of identity-related fields are sparse.

Examples:
- `DeviceInfo`
- `DeviceType`
- several `id_*` fields

This matters for two reasons:

### Data quality angle
We cannot assume these fields are consistently available, so null handling has to be part of the pipeline.

### Signal angle
Missingness may itself be useful.
For example:
- identity missing vs present
- device info missing vs present
- number of missing identity fields

This is worth checking early because sparse fraud datasets often contain signal in what is absent, not just what is present.

---

## 3. Transaction amount

`TransactionAmt` is one of the first things to inspect.

Main questions:
- do fraud and legit transactions differ by amount?
- is fraud concentrated in small, medium, or large amounts?
- does amount behave differently across product or entity groups?

Useful first cuts:
- raw distribution
- log distribution
- fraud rate by amount bucket

Reason to do this early:
- amount often interacts with risk
- amount buckets can become useful features
- later thresholding / decision logic usually depends on transaction value anyway

---

## 4. Entity concentration

Fraud is usually not spread uniformly across rows.
It tends to cluster around repeated entities.

Main groups to inspect:
- card fields
- email domains
- address fields
- device / identity fields

What to check for each:
- transaction count
- fraud count
- fraud rate
- concentration of volume

This is important because feature engineering will likely rely on:
- entity history
- entity risk
- cross-entity consistency
- velocity

EDA should help confirm which entity groups are worth prioritizing.

---

## 5. Time patterns

`TransactionDT` is relative time, but ordering still matters.

Things worth checking:
- transaction volume over time
- fraud rate over time
- whether fraud appears in bursts
- whether there are visible shifts in behavior over the dataset span

This matters for two reasons:
1. it helps with fraud intuition
2. it affects validation design

If fraud is non-stationary over time, random splitting becomes even less reliable.

---

## 6. Device and identity signals

Where identity data exists, it is worth checking whether fraud behaves differently by:
- `DeviceType`
- `DeviceInfo`
- selected browser / OS style fields
- selected `id_*` fields

The main point is not to over-interpret these columns.
It is to see whether they:
- carry signal when present
- create strong missingness patterns
- help define repeatable entities or consistency checks

---

## 7. What we are not trying to do in EDA

This EDA is not meant to:
- fully explain every anonymized feature
- optimize final feature selection
- replace validation
- draw strong conclusions from one-way plots alone

The goal is just to narrow the search space for feature engineering.

---

## Initial takeaways to carry forward

Based on the dataset structure, the main working assumptions going into feature engineering are:

1. fraud is rare, so we need concentrated signals
2. missingness may be predictive
3. amount is likely useful but probably not enough on its own
4. entity-level behavior matters more than single rows
5. time order has to be respected in both features and validation

---

## What this means for feature engineering

The next stage should focus on feature families like:
- entity history
- velocity
- consistency checks
- missingness indicators
- amount transforms / buckets
- temporal features derived from transaction order

EDA should help prioritize which of these are worth building first.

---

## Next

[4_feature_engineering.md](4_feature_engineering.md)