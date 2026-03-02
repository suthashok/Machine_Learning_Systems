
# Online Transaction Fraud Detection System – Problem Framing

This document outlines my thinking of how to approach a Fraud detection System.

---

## 1. Business Objective

### 1.1 What are we optimizing for?

We want to maximize how much money we make per transaction. Not just catch fraud—make money.

For each transaction, we need to decide: approve it or decline it. Here's what we care about:

- pi​ = probability that is it a fraudulent transaction (between 0 and 1)
    
- TAi​ = transaction amount
    
- M = revenue margin rate
    
- Cx = Customer contact servicing cost (Asumme $4 per contact, for a transaction customer contact once)
    
**If we approve:**

We make margin on legit transactions and lose the whole amount on fraudulent ones:

    EVa,i = (1−pi) (m⋅TAi)− pi.TAi

In plain terms: (chance it's good × what we make) - (chance it's fraud × what we lose)

**If we decline:**


We lose the margin we would have made, plus some customers will call us asking why:

    EVd,i = -(1−pi) (m⋅TAi) − Cx

In plain terms: (chance it was legit × margin we lose) + (legit customers call support, costs money)

Note: Only legitimate customers call—fraudsters typically don't.

Decision rule:

    We pick the action whichever makes us more money.

---

### 1.2 Business Constraints

For this exercise with IEEE CIS data:

- Keep approval rate close to baseline (don't over-decline)
- Model must output well-calibrated probabilities (so our EVa/EVd math works)
    
---


## 2. Timeline and Data Labeling

### 2.1 When do we make the prediction?

- We score at t0 = the moment the customer tries to make the transaction.
    
    - Constraint 1 : Model must score using only data available at or before t0
    
    For ith transaction, feature set should meet following:-

    Feature_i=f(Data≤t0)

    This ensures no post-transaction leakage.

---

### 2.2 How much history do we look at?

We look back 30-90 days to build features:
- How many transactions in the last 30 days?
- What devices has this customer used?
- Is their spending pattern changing?
- Any recent changes to their account or payment method?
    
All calculations are relative to t0 (the transaction time).

---

### 2.3 Outcome Window

- For card transactions, fraud shows up as chargebacks. They can happen anywhere from 1-180 days after the transaction.
- But we can't wait 180 days to label data. So we wait for tm days (usually 30-45 days) until ~80% of chargebacks have shown up. That's our label cutoff.

**Example:** If a transaction happens on Jan 1, we label it as fraud or not-fraud around Feb 10-15 (after most chargebacks arrive).


- Outcome window = tm (typically 30day/45days for most Geos)

---

### 2.4 Dealing with incomplete data

- We can't use transactions that don't have a complete label window yet.

**Example:** If today is March 1 and we need 45 days to label, we can't use transactions from after Feb 14 (because we don't know yet if they're fraud).

- If we include them, we artificially bias the training data toward "not fraud" because chargebacks haven't arrived yet.

---

## 3. Risk Decision framework

We need to decide: approve or decline each transaction. The model gives us a fraud probability (pi between 0 and 1).

When we approve, we make margin but lose money if it's fraud:.

**Decision Logic:**

If EVa,i > EVd,i → Approve
Else → Decline


**For this exercise (IEEE CIS dataset):**

We're assuming 2% margin and ignoring chargeback fees, recovery, step-up auth, etc. Real systems deal with all that.

**In production you'd add:**
- Chargeback fees ($25-$100 per fraudulent transaction)
- Some fraud gets recovered (~30%)
- Ask customers for extra auth (costs ~$4, succeeds ~85% of the time)
- Customers might drop off if you ask too many questions (hidden cost)

---

## 4. Evaluation Framework

### 4.1 Business Metrics

The main metric is: **did we make more money?**

- **Net Expected Value (NEV) per transaction** — This is what we optimized for. How much money do we make on average per transaction with our model vs. without it?

- **NEV lift vs. baseline** — If we just approved everything, what would NEV be? How much better is our model?

- **Fraud loss as % of transaction volume** — How much money do we actually lose to fraud? (This tells us if our thresholds are working)

- **Approval rate** — What % of transactions did we approve? (Important because it should be close to baseline)

 
---

### 4.2 Model Metrics

- PR-AUC (class imbalance aware)
- Calibration error
- Threshold stability
- Score distribution stability
    
---

### 4.3 Operational Metrics

- Latency (P50 / P95)
- Manual review volume
- System fallback rate

---

## 5. Retraining & Model Lifecycle

### 5.1 Base Cadence

- Quarterly retraining using matured labels. (This ensures any seasonal variations as well as recent fraud trends are covered.)

---

### 5.2 Trigger-Based Retraining

Triggered if:

- PSI > 0.2 on critical features (top 10% in feature immportance list)
- Fraud rate shift > 50%
- Fraud capture rate drop > 25%
- Score distribution shift > 10%

---

### 5.3 Validation Strategy

- Out-of-time validation
- Backtesting against historical policy  
- Stability analysis across segments
- Challenger Business Vintage based Forecast for overall loss rate


--- 


## 6. Monitoring Framework

### 6.1 Business Monitoring (Pre/Post)

- Approval rate 
- Fraud rate
- Net expected value
- Contact rate

---

### 6.2 Data Drift Monitoring

- PSI on top features (top 10)
- Missing rate changes 
- Cardinality shifts

---

### 6.3 Model Drift Monitoring

- Score distribution
- Calibration drift
- Capture rate stability

---

### 6.4 Operational Monitoring

- Latency 
- Manual review overflow
- System timeouts

---


## 7. Governance & Risk Controls

- Model versioning & lineage
- Feature reproducibility guarantees
- Bias & fairness evaluation
- Rollback strategy
- Champion / challenger setup

---