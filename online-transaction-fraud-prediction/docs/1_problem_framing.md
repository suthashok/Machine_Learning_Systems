
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
    
- Cx = Customer contact cost (Asumme $4 per contact, for a transaction customer contact once)
    
**If we approve:**

We make margin on legit transactions and lose the whole amount on fraudulent ones:

    EVa,i = (1−pi) (m⋅TAi)− pi.TAi

In plain terms: (chance it's good × what we make) - (chance it's fraud × what we lose)



Expected value if model **Decline**:

    EVd,i = -(1−pi) (m⋅TAi) − Cx

    (Lost margin of declining a Good transaction + Good Customer Contact Cost)

Decision rule:

    Choose Action with Highest expected Value

---

### 1.2 Business Constraints (Guardrails)

The ML system must satisfy following constraints:

- No significant impact on Approval rate (Acceptable range ≥ baseline – 0.5%)
    
- No Delay on SLA (P95 latency < 100ms)
    
- Manual review queue within operational capacity (basline + 5%)
    
- Customer contact increase < 2% (Considering already hit of 2% take rate )
    
- Fraud loss within regulatory tolerance (<100bps or equivalent for specific Geo)
    

---


## 2. High Level Timeline Framing

### 2.1 Prediction Timestamp

- t0 = transaction attempt time
    
    - Constraint 1 : Model must score using only data available at or before t0
    
    For ith transaction, feature set should meet following:-

    Feature_i=f(Data≤t0)

    This ensures on post-transaction leakage.

---

### 2.2 Feature Lookback Window

Features can include aggregations over following periods:

- Previous 30 to 90 days
    - transaction history
    - Device usage history
    - Account behavioral velocity
    - Profile/Funding Instrument changes
    
All Lookback computations will be relative to t0.

---

### 2.3 Outcome Window

- For Card transactions, fraud is defined as chargeback events occurring within 1–180 days post transaction. Chargeback window is t0+ 1 days (specific Geo/Processor might have different end date cutoff).

- For model outcome window will be set based on Historical Vintage Arrivals. tm will be the period when nearly ~80% of chargebacks are arrived.

- Outcome window = tm (typically 30day/45days for most Geos)

---

### 2.4 Label Maturity & Right Censoring

To avoid bias due to maturity, the data will be censored for following criteria:

- Exclude transactions where t0+tm exceeds data cutoff. (These will be recent transactions (t < t0+tm) which will be biased (for Non-Fraud) as arrival window is incomplete and should be excluded from training/validation sets.

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

- Net expected value per transaction lift    
- Fraud loss reduction (bps) lift    
- Fraud capture rate at fixed approval rate
- Incremental customer contact rate
    
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