# Data Leakage Prevention

## Goal

This doc lists the main leakage risks in the IEEE-CIS setup and the rules used to avoid them.

Core rule:

> For any transaction at time `t0`, features must use only data available at or before `t0`.

---

## Why this matters

Fraud models are easy to overstate offline because many useful features depend on history:
- prior transaction count
- prior average amount
- prior device/card usage
- prior fraud rate
- prior consistency patterns

If those features use future rows, validation stops being trustworthy.

---

## Main leakage risks

### 1. Future-aware aggregates
Unsafe examples:
- card average amount computed on full data
- device transaction count using future rows
- entity stats built before sorting by `TransactionDT`

Rule:
- all history features must be past-only

### 2. Target leakage
Unsafe examples:
- fraud rate by `card1` computed on full train
- target encoding built before split
- any feature using `isFraud` outside fold-safe logic

Rule:
- if a feature uses the target, treat it as high risk
- prefer count / recency / frequency features first

### 3. Random split
Random train/validation split is risky because:
- data has time order
- history-based features depend on past behavior
- shuffling can hide leakage

Rule:
- train on older rows
- validate on newer rows

### 4. Preprocessing leakage
Unsafe examples:
- fitting encoders on train + validation together
- fitting imputers on full data
- scaling before split
- using shared frequency tables across splits

Rule:
- fit preprocessing on training only
- apply to validation / test after that

---

## Safe implementation rules

### Historical features
For entity-based features:
1. sort by `TransactionDT`
2. compute using prior rows only
3. never use future rows in the same entity history

### Split-aware feature generation
Do not precompute unsafe full-dataset aggregates and reuse them everywhere.

Feature generation should respect the validation setup.

### Target-based features
Only use if clearly leakage-safe:
- out-of-fold in training
- or strictly past-only by time

If unclear, drop them.

---

## Validation rules

- use time-aware split
- older data -> training
- newer data -> validation
- validation labels must never influence training features
- final holdout should be later than training window

---

## Practical checks

Before trusting results, verify:

1. every history feature uses only past rows
2. no target-based feature was built on full data
3. preprocessing was fit on training only
4. validation is time-aware
5. stricter validation does not collapse performance

If performance drops sharply after fixing split logic, leakage is a likely reason.

---

## Default bias for this repo

When in doubt:
- choose the simpler feature
- choose the safer split
- drop the suspicious encoding

A weaker safe model is better than a stronger leaky one.

---

## Takeaway

Leakage prevention here comes down to five rules:

1. respect transaction order
2. use past-only history
3. avoid unsafe target features
4. fit preprocessing on training only
5. validate on newer data

---

## Next

[6_modelling_and_calibration.md](6_modelling_and_calibration.md)