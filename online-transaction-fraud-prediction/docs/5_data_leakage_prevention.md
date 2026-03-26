# Data Leakage Prevention

## Goal

This doc lists the main ways leakage can happen in the IEEE-CIS setup and the rules used to avoid it.

Main rule:

> for any transaction at time `t0`, features must use only data available at or before `t0`

If this rule breaks, offline results will look better than they should.

---

## Why this matters

Leakage is easy to create in fraud problems because many useful features depend on history.

Examples:
- prior transaction count
- prior average amount
- prior device usage
- prior entity risk

If those features use future rows, the model is no longer being tested fairly.

---

## Main leakage risks

### 1. Future-aware aggregates

Unsafe examples:
- card average amount built on full data
- device transaction count using future rows
- entity history computed before sorting by `TransactionDT`

Rule:
- all history features must be built from past rows only

### 2. Target leakage

Unsafe examples:
- fraud rate by `card1` built on full data
- target encoding done before split
- any feature using `isFraud` without fold-safe logic

Rule:
- if a feature uses the target, treat it as high risk
- prefer count / frequency / recency features first

### 3. Random split

Random split is risky here because:
- rows have time order
- many features depend on history
- shuffling can hide leakage

Rule:
- train on older rows
- validate on newer rows

### 4. Preprocessing leakage

Unsafe examples:
- fitting encoders on full data
- fitting imputers before split
- scaling before split
- using train + validation together to build mappings

Rule:
- fit preprocessing on training only
- apply the same transform to validation / test

---

## Safe feature rules

For any history-based feature:
1. sort by `TransactionDT`
2. compute using prior rows only
3. do not use future rows for the same entity

For any target-based feature:
- use only if the setup is clearly leakage-safe
- if unclear, drop it

Simple safe features are better than stronger unsafe ones.

---

## Validation rules

Validation should always be time-aware:

- older rows -> training
- newer rows -> validation

Also:
- validation labels should never affect training features
- final holdout, if used, should be later than training data

---

## Practical checks

Before trusting a result, check:

1. does every history feature use only past rows?
2. was any target-based feature built on full data?
3. was preprocessing fit only on training data?
4. is the split time-aware?
5. does performance hold under stricter validation?

If performance drops a lot after fixing split logic, leakage is a likely reason.

---

## What is usually safe

Usually safer:
- raw row-level fields
- missingness flags
- log amount
- past-only counts
- past-only recency features

Usually riskier:
- fraud-rate features
- target encodings
- rolling stats built on full data
- entity summaries created before split

---

## Takeaway

Leakage prevention here comes down to a few rules:

1. respect transaction order
2. use past-only history
3. be careful with target-based features
4. fit preprocessing on training only
5. validate on newer data

If these rules hold, offline results are much more trustworthy.

---

## Next

[6_modelling_and_calibration.md](6_modelling_and_calibration.md)
