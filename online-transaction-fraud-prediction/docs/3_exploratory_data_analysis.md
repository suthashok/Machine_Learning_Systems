# Exploratory Data Analysis

## Goal

This doc captures the first things worth checking in the IEEE-CIS data before feature engineering.

The goal is not to summarize every column.
It is to answer a few practical questions:

- how imbalanced is fraud?
- where does fraud seem to concentrate?
- what looks useful for feature engineering?
- what data issues do we need to handle carefully?

---

## What I want to check first

For this dataset, the first pass of EDA should focus on:

1. class balance
2. missingness
3. amount behavior
4. entity concentration
5. time patterns

These are the things most likely to shape features and validation.

---

## 1. Class balance

Fraud is a small minority class in this dataset.

That immediately means:
- raw accuracy is not useful
- we care more about ranking and separation
- useful fraud signal will likely be concentrated, not broad

So the first assumption going in is: most rows will look normal, and the model has to find the pockets that do not.

---

## 2. Missingness

A lot of identity-related fields are sparse.  
That includes things like:
- `DeviceInfo`
- `DeviceType`
- several `id_*` fields 

This matters in two ways:

- we need to handle nulls cleanly
- missingness itself may be predictive

So early checks should compare fraud rate for:
- field present vs missing
- identity present vs absent
- low-missingness vs high-missingness rows

This is one of the first things worth testing because it can turn into simple, useful features later.

---

## 3. Transaction amount

`TransactionAmt` is one of the first columns worth looking at.

Main questions:
- does fraud behave differently by amount?
- is fraud concentrated in small, medium, or large transactions?
- does the amount distribution suggest useful buckets or transforms?

Useful first checks:
- raw distribution
- log distribution
- fraud rate by amount bucket

Amount is usually not enough on its own, but it often becomes more useful once combined with entity history or velocity.

---

## 4. Entity concentration

Fraud is usually not spread evenly across rows.
It tends to show up around repeated entities.

Main groups to inspect:
- card fields
- email domains
- address fields
- device / identity fields

For each one, the first checks should be:
- transaction count
- fraud count
- fraud rate

This helps answer a simple question:
- are there entities that look unusually risky?
- are there entities that show up a lot in fraud even if their raw count is small?

If yes, that is a strong signal for entity-based features later.

---

## 5. Time patterns

`TransactionDT` is relative time, but it still gives row order.

Things worth checking:
- transaction volume over time
- fraud rate over time
- whether fraud appears in bursts
- whether the later part of the dataset looks different from the earlier part

This matters for two reasons:
- it gives some intuition about how fraud behaves
- it helps confirm that validation should stay time-aware

---

## 6. What this EDA is really for

This stage is not meant to fully explain the dataset.
It is mainly there to narrow the search space for feature engineering.

If this pass goes well, we should come out with a few working ideas:

- missingness may carry signal
- amount likely needs transformation or bucketing
- fraud risk may cluster by entity
- transaction order matters
- history-based features are likely more useful than row-level values alone

That is enough to move to the next stage.

---

## Next

[4_feature_engineering.md](4_feature_engineering.md)