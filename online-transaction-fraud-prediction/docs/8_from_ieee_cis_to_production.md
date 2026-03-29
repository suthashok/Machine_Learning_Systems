# From IEEE-CIS to Production

## Goal

Docs 1–7 cover the offline IEEE-CIS solution.

This doc lists what else would be needed to turn that into a real fraud system.

The main point:
- IEEE-CIS gives us the offline modeling core
- production needs the system around that model

---

## What changes in production

In IEEE-CIS:
- data is already labeled
- feature generation is offline
- validation is offline
- threshold analysis is offline

In production:
- transactions arrive one by one
- features must be available at scoring time
- labels arrive late
- thresholds affect real customers
- model quality has to be tracked after deployment

---

## 1. Live feature generation

Offline feature code is not enough.

In production, we need:
- the same feature definitions
- computed at transaction time
- using only data available at that moment

That usually means:
- precomputed history tables
- fast lookups for entity features
- online updates for recent activity

If train-time and run-time features do not match, model quality will drop.

---

## 2. Score serving

The model has to run inside a real decision flow.

That means:
- input arrives in real time
- feature values are assembled
- model returns a fraud score quickly
- the score is passed to decision logic

At this point, latency starts to matter in a way it does not in IEEE-CIS.

---

## 3. Decision logic around the model

The model score is still not the full decision.

A production system usually needs:
- score cutoff / threshold
- action for risky transactions
- action for borderline cases
- fallback logic if data is missing

Typical actions could be:
- allow
- decline
- review
- extra verification

IEEE-CIS stops at score + threshold analysis.
Production needs the rest of the action layer too.

---

## 4. Delayed labels

This is one of the biggest real-world differences.

In production, fraud labels often arrive late.
That means:
- we may not know right away if a transaction was actually fraud
- recent data may be incomplete for training
- retraining data has to account for label delay

This part does not exist in the same way in IEEE-CIS because the dataset is already labeled.

---

## 5. Monitoring

Once the model is live, we need to track whether it is still behaving well.

Main things to watch:
- score distribution
- fraud rate in scored traffic
- false positive rate
- approval rate
- calibration drift
- feature drift

The point is simple:
- fraud patterns change
- customer behavior changes
- models get stale

---

## 6. Retraining

If the live data starts to drift, the model may need to be retrained.

Possible triggers:
- worse fraud capture
- more false positives
- feature distributions moving
- score calibration getting worse

Retraining is not just “run training again”.
We need:
- clean training windows
- mature labels
- stable feature definitions
- comparison against the current model

---

## 7. Review workflow

In a real fraud system, not every risky transaction should be auto-declined.

Some cases may need:
- manual review
- follow-up checks
- feedback loop from investigators

That feedback can later help:
- improve rules
- improve labels
- improve future model versions

---

## 8. Governance and versioning

A production model should be traceable.

At minimum we should know:
- which data version was used
- which feature logic was used
- which model version is live
- which threshold is active
- when something changed

This is important for debugging and rollback.

---

## Takeaway

IEEE-CIS covers the offline core:
- data
- features
- leakage prevention
- model
- score
- threshold analysis

Production needs the rest of the system around it:
- live feature generation
- score serving
- decision logic
- delayed-label handling
- monitoring
- retraining
- review flow
- version control

That is the gap between a good offline model and a real fraud system.