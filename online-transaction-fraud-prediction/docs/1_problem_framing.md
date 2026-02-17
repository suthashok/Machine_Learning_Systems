
# Online Transaction Fraud Detection System – Problem Framing

This document outlines my thinking of how to approach a Fraud detection System.

---

## 1. Business Objective

### 1.1 What is Optimization target?

Design a real-time fraud risk prediction system that **maximizes net expected value (NEV)** per transaction.

For transaction _i_:

Let:

- pi​ = predicted fraud probability
    
- TAi​ = transaction amount
    
- M = revenue margin rate
    
- Cx = Customer contact cost (Asuume $4 per contact)
    

Expected value if the model **Approve**:

    EVa,i = (1−pi) (m⋅TAi)− pi.TAi

    Expect Value at Approve = Non Fraud Probability * Margin Rate * Transaction Amount - Fraud Probability * Transaction Amount

    (Benefit/Margin of Approving Good transaction - Cost of Approving a Bad transaction)


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

Model outputs risk scores pi, typically ranges from (0 to 1000, or 0.000 to 1.000). However the Risk decision still needs to be defined clearly based on risk score.

Risk Decision framework maps model score to action:

|Score Range|Action|
|---|---|
|(0-300) Low Risk|Approve|
|(301-700) Medium Risk|Step-up Authentication|
|(701-1000) High Risk|Decline|

Thresholds should be decided in accoradance with **maximize net expected** (defined above) value subject to guardrails.

There will also be default Risk action in-case model score is not available or doesn't fall in these buckets. 

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