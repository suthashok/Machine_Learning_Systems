
# Problem Framing

## Goal

This project has two goals:

1. Build a strong offline solution for the IEEE-CIS fraud detection problem.
2. Show how the same problem would be framed in a real production fraud system.

The IEEE-CIS dataset is a labeled offline snapshot. So the implementation here is an offline modeling exercise, but the documentation will also cover the production thinking behind it.

---

## What problem are we solving?

At the dataset level, the task is simple:

> Predict `isFraud` for each transaction.

At the system level, the real problem is broader:

> Use that fraud risk estimate to make a good transaction decision.

That means this is not just a classification problem. In production, the model is one part of a larger decision system:
- rules
- ML model
- thresholds / policy
- review workflow
- monitoring

For IEEE-CIS, we only implement the modeling core. The rest is documented as production context.

---

## Business framing

Fraud modeling is not just about catching the most fraud.

Each decision has a tradeoff:
- approve a bad transaction -> lose money
- decline a good transaction -> lose revenue + hurt customer experience

So the model should output a fraud probability, and policy should decide what to do with it.

For this project:
- **offline objective** = learn a strong fraud ranking / probability model
- **production objective** = use calibrated scores in a decision framework

---

## Scope

### In scope
- offline fraud prediction on IEEE-CIS
- time-aware validation
- leakage-safe feature engineering
- model evaluation
- production-style system thinking in documentation

### Out of scope
- live deployment
- real-time feature store
- actual review queue / ops workflow
- real chargeback pipeline
- online threshold experimentation

---

## Key assumptions

### 1. Prediction happens at transaction time
For any transaction at time `t0`, features must only use data available at or before `t0`.

No future transactions.
No future labels.
No future aggregates.

This is the main anti-leakage rule.

### 2. Transaction order matters
`TransactionDT` is not a real timestamp, but it does give transaction ordering.

We’ll use that ordering for:
- feature generation
- train/validation split
- leakage prevention

### 3. IEEE-CIS is offline, but production concepts still matter
The dataset is already labeled, so we do not need to solve label maturity operationally here.

But in a real fraud system:
- labels arrive late
- thresholds matter
- calibration matters
- monitoring matters

So those concepts are included to show production thinking, even if not fully implemented.

---

## What does success look like?

### For the IEEE-CIS task
- strong out-of-time model performance
- no leakage
- robust feature engineering
- good generalization to newer transactions

### For production-style thinking
- calibrated probabilities
- clear separation between model and policy
- ability to reason about approval rate vs fraud loss
- monitoring plan for drift and degradation

---

## Evaluation approach

We should not validate this like a generic tabular ML problem.

Fraud is time-dependent, and many features depend on historical behavior. So validation should reflect the real usage pattern:

- train on older transactions
- validate on newer transactions

Random splits are risky here because they can hide leakage and overstate performance.

---

## Decision-system view

The model output is not the final decision.

The model produces:
- fraud probability / risk score

A production policy would then decide:
- approve
- decline
- maybe review / step-up (not implemented here)

That separation matters because:
- model quality is about ranking + probability estimation
- business quality is about decision outcomes

---

## Design principles for the rest of the project

1. Treat fraud as a decision problem, not just a classification problem.
2. Respect transaction-time ordering everywhere.
3. Engineer features around behavior, velocity, and entity risk.
4. Assume missingness may itself be useful signal.
5. Keep the IEEE-CIS solution grounded in what would work in production.

---

## What comes next

- `2_data_sources_and_structure.md` -> what data we actually have
- `3_exploratory_data_analysis.md` -> what patterns are worth testing
- `4_feature_engineering.md` -> how those patterns become model features
- later docs -> leakage prevention, modeling, thresholds, monitoring