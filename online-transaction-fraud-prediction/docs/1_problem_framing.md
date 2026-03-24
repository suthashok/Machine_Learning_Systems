# Problem Framing

## Goal

This project solves the IEEE-CIS fraud detection problem.

Task:
- predict `isFraud` for each transaction

The project is built as an offline modeling exercise using the IEEE-CIS dataset.
A separate document will cover what would need to change for a real production fraud system.

---

## What problem are we solving?

This is a transaction-level binary classification problem.

For each transaction, we want one output:

> probability that the transaction is fraud

The dataset is split across transaction and identity tables, joined by `TransactionID`.
`TransactionDT` is a relative time field, so it should be used for ordering, not as a real timestamp.

---

## What matters for this project

Three things matter most:

1. good features
2. no leakage
3. realistic validation

A model with strong offline metrics is not useful if:
- features use future data
- split logic is weak
- performance does not hold on newer transactions

---

## Main assumptions

### 1. Prediction happens at transaction time
For any transaction at time `t0`, features must use only data available at or before `t0`.

No future rows.
No future labels.
No future aggregates.

### 2. Transaction order matters
`TransactionDT` is not a wall-clock timestamp, but it still gives the event order.

We should use that order for:
- feature generation
- train / validation split
- leakage prevention

### 3. This is an offline setup
IEEE-CIS is a labeled historical dataset.
So the focus here is:
- offline feature engineering
- offline model training
- offline validation
- offline score / threshold analysis

---

## What success looks like

For this project, success means:

- strong validation performance on newer transactions
- leakage-safe feature engineering
- stable score behavior across time splits
- clear fraud ranking from the model

Since fraud is imbalanced, raw accuracy is not useful.
Model quality should be judged using ranking-focused metrics and score behavior.

---

## Validation philosophy

We should not use random split here.

Use:
- older transactions for training
- newer transactions for validation

Reason:
- fraud changes over time
- many useful features depend on history
- random split can overstate performance

Validation should reflect the way the model would be used.

---

## What this project will produce

By the end of the IEEE-CIS flow, we want:

- a clean joined dataset
- leakage-safe engineered features
- a fraud model trained on older data
- validation results on newer data
- a score cutoff / threshold analysis

That is the core offline solution.

---

## What this document does not cover

This doc does not cover full production concerns like:
- label maturity
- live scoring
- monitoring
- retraining
- review workflows

Those are important, but they are outside the IEEE-CIS scope for docs 1–7.

---

## Design principles for the rest of the repo

1. Keep the setup strictly transaction-time safe.
2. Respect transaction order everywhere.
3. Prefer realistic validation over optimistic validation.
4. Build features around behavior, history, and consistency.
5. Keep the offline solution separate from production extension.

---

## Next

[2_data_sources_and_structure.md](2_data_sources_and_structure.md)