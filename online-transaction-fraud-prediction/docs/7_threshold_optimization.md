# Threshold Optimization

## Goal

This doc covers how we turn model scores into decisions.

The model gives a fraud probability for each transaction.
We still need a cutoff to decide what score is risky enough to act on.

For IEEE-CIS, this stays an offline exercise.
The goal here is to understand the tradeoff, not to build a full policy engine.

---

## What threshold means

A threshold is just a score cutoff.

Example:
- score >= threshold -> flag as risky
- score < threshold -> treat as safe

For now, keep the action simple:
- above threshold -> decline / flag
- below threshold -> allow

The exact action can change later, but the tradeoff stays the same.

---

## Why this matters

A good model can still give bad outcomes if the threshold is wrong.

Too low:
- catch more fraud
- block too many good transactions

Too high:
- protect approval rate
- miss too much fraud

So threshold choice should come from validation results, not intuition.

---

## What to use for tuning

Use:
- validation scores
- true labels from the validation set
- transaction amount if we want value-based checks

Do not tune threshold on training predictions.

---

## What to measure

For a range of thresholds, track:

- precision
- recall
- false positive rate
- % transactions flagged
- fraud captured

This is enough for the first pass.

If needed later, we can add amount-weighted views.

---

## Simple workflow

1. score the validation set
2. test a range of thresholds
3. compare precision vs recall
4. check how many transactions get flagged
5. inspect fraud captured at each cutoff
6. pick a sensible operating point

Do not choose threshold from one metric alone.

---

## Start simple

Use one global threshold first.

That gives us:
- one clear tradeoff curve
- one easy operating point
- one simple story to explain

If needed later, we can check whether the threshold behaves differently across segments.

---

## What a good threshold should do

A good first threshold should:
- catch meaningful fraud
- avoid too many false positives
- stay reasonably stable across time splits
- be easy to explain

If the best threshold changes a lot from one split to another, the score may not be stable enough yet.

---

## Failure signs

Watch for:
- tiny threshold change causing huge decision swings
- recall improving but precision collapsing
- very high cutoff still catching little fraud
- threshold behaving very differently across time splits
- score distribution too compressed to separate good vs bad transactions

These usually point back to model or calibration issues.

---

## Output of this stage

By the end of this stage we should have:
- threshold tradeoff table
- first recommended cutoff
- notes on stability across validation splits

That is enough to finish the IEEE-CIS flow.

---

## Takeaway

The model score is not the decision.
Threshold is what turns score into action.

The goal here is not to find one perfect number.
It is to understand the tradeoff and choose a sensible first cutoff.

---

## Next

[8_from_ieee_cis_to_production.md](8_from_ieee_cis_to_production.md)
