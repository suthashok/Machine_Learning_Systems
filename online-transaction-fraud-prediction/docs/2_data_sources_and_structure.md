# Data Sources and Structure

## Goal

This doc captures what raw data we have for the IEEE-CIS problem and what parts of the structure matter for modeling.

The main point here is not to describe every column. It is to make clear:
- what tables exist
- how they join
- what entities we can track
- what constraints this creates for feature engineering and validation
---
## Data used in this project

We use the IEEE-CIS fraud detection dataset.

Main training files:
- `train_transaction.csv`
- `train_identity.csv`

Main test files:
- `test_transaction.csv`
- `test_identity.csv`

The transaction table is the base table.

The identity table adds device / identity-related fields for only part of the population.

Join key:
- `TransactionID`
Typical merge:
```python

df = train_transaction.merge(train_identity, on="TransactionID", how="left")
```

Left join is important because not every transaction has identity coverage.

---

## What sits in each table

### 1. Transaction table

This is the main table and contains:

- row-level transaction info
- the fraud label in train
- most of the usable modeling features

Important fields:

- `TransactionID`
- `TransactionDT`
- `isFraud`
- `TransactionAmt`
- `ProductCD`
- `card1` to `card6`
- `addr1`, `addr2`
- `P_emaildomain`, `R_emaildomain`

It also includes grouped feature families:

- `C*`
- `D*`
- `M*`
- `V*`

Some of these are interpretable at a high level, some are heavily anonymized.

### 2. Identity table

This table contains additional device / identity signals, for example:

- `DeviceType`
- `DeviceInfo`
- `id_*`

The important thing here is not just the fields themselves, but the fact that this table is incomplete. A lot of rows in the merged dataset will have nulls for identity-related features.

---

## What matters structurally

### 1. One row = one transaction

This is a transaction-level prediction problem. Everything downstream — EDA, feature engineering, validation, modeling — should stay aligned to that unit.

### 2. Transaction table is primary

`train_transaction` is the source of truth because:

- all transactions are there
- the target is there
- identity data is optional

So all modeling should be built on top of the joined transaction-first table.

### 3. Time order exists, even if timestamp is relative

`TransactionDT` is not a real timestamp. But it still gives event order.

That is enough for:

- sorting transactions
- building historical features
- preventing leakage
- designing train/validation splits

We should treat it as the ordering variable throughout the project.

---

## Main entity types available

The dataset gives a few obvious entity families that are useful for fraud work.

### Card-like entities

- `card1` to `card6`

These are not guaranteed to map cleanly to a customer or account, but they are still useful as repeatable payment identifiers.

### Address-like entities

- `addr1`
- `addr2`

These can help with location / billing consistency and aggregation.

### Email entities

- `P_emaildomain`
- `R_emaildomain`

Useful for domain-level risk, consistency checks, and simple historical aggregates.

### Device / identity entities

- `DeviceType`
- `DeviceInfo`
- selected `id_*` fields

Useful when present, but coverage is incomplete, so sparse handling matters.

These entity groups are important because fraud usually shows up as repeated behavior around the same cards, devices, email patterns, or address patterns — not just through one-off row-level values.

---

## Constraints that matter for modeling

### Identity data is partial

A lot of transactions do not have identity records.

Implication:

- null handling is not optional
- identity presence itself may be informative
- missingness features are likely worth testing

### Many fields are anonymized

Columns like `V*`, `C*`, `D*`, `M*`, and many `id_*` fields do not have clean business definitions.

Implication:

- use them empirically
- rely on validation, stability, and feature importance
- avoid over-explaining what they “mean”

### Time-aware processing is required

A lot of useful fraud features depend on history:

- prior transaction count
- prior average amount
- prior device/card usage
- prior consistency patterns

Implication:

- features must only use past data
- random splitting is risky
- leakage prevention has to be built into the pipeline, not added later

### Entity fields are noisy

A single field like `card1` or `DeviceInfo` may not uniquely identify a person or device.

Implication:

- single-entity features will help
- but later we may also need consistency features or composite identifiers

---

## Working table for the rest of the project

After joining transaction and identity data, the working dataset contains:

- transaction-level features
- optional identity/device features
- target label in train
- transaction order via `TransactionDT`

This joined table is the starting point for:

1. exploratory analysis
2. feature engineering
3. split design
4. model training

---

## Takeaway

The structure of the dataset points to a pretty clear modeling direction:

- treat this as a **transaction-level** fraud problem
- preserve **transaction order**
- build features around **entity history**, **velocity**, and **consistency**
- handle **missingness** as potential signal
- be careful with anonymized feature groups and let validation decide what survives

---
## Next

[3_exploratory_data_analysis.md](3_exploratory_data_analysis.md)