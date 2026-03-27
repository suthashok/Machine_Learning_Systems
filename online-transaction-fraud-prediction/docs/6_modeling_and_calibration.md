# Modelling and Calibration

## Goal

This doc covers two things:
1. how we train the fraud model
2. how we check whether the score is usable as a probability

This is still an offline IEEE-CIS setup.
Threshold choice comes after this.

---

## Prediction task

We want one output per transaction:

> probability that this transaction is fraud

Inputs:
- joined transaction + identity data
- leakage-safe features
- time-aware train / validation split

Target:
- `isFraud`

---

## Split strategy

Do not use random split here.

Use:
- older transactions for training
- newer transactions for validation

Reason:
- fraud patterns can shift over time
- many useful features depend on history
- random split can make results look better than they really are

So model comparison should always be based on out-of-time validation.

---

## Models to try

Start simple, then move to stronger tabular models.

Good starting set:
- logistic regression
- LightGBM
- XGBoost
- CatBoost

Why this set:
- logistic regression gives a simple baseline
- boosting models usually work well on this kind of data

No need to try too many families early.
Better to get split logic and features right first.

---

## Class imbalance

Fraud is rare, so imbalance needs attention.

Options:
- class weights
- `scale_pos_weight`
- thresholding later

Main point:
- ranking matters first
- but score quality still matters

If imbalance handling makes the score unstable, calibration needs extra care.

---

## What to measure

For model comparison:
- ROC-AUC
- PR-AUC

For score quality:
- calibration plot
- log loss
- Brier score

For sanity check:
- score distribution
- fraud rate in top score bands
- performance across time splits

---

## What a good model should do

A useful fraud model should:
- rank fraud above legit transactions
- work on newer data, not just the training period
- produce a reasonable spread of scores
- not depend on leaky features
- stay simple enough to debug

A model with one strong metric but weak validation is not useful.

---

## Calibration

Raw model scores are not always good probabilities.

That matters because later we may want to use the score for:
- threshold setting
- score band analysis
- simple decision tradeoffs

Basic question:
- when the model says 10% fraud, is the observed fraud rate somewhere close?

It does not need to be perfect.
It just should not be badly misleading.

---

## Calibration approach

Recommended order:

1. train model on training data
2. score validation data
3. check ROC-AUC / PR-AUC
4. check calibration
5. calibrate only if needed
6. compare before vs after

Methods worth testing:
- Platt scaling
- isotonic regression

Do not calibrate on the same data used to fit the base model.

---

## Common failure signs

Watch for:
- big gain after adding one suspicious feature group
- strong ROC-AUC but weak PR-AUC
- good ranking but poor calibration
- model works on one split and fails on another
- scores packed into a narrow band

These usually tell us more than one headline metric.

---

## Output of this stage

By the end of this stage we should have:
- chosen model
- chosen feature set
- out-of-time validation results
- decision on whether calibration is needed
- final transaction-level fraud score

Threshold optimization comes after this.

---

## Next

[7_threshold_optimization.md](7_threshold_optimization.md)