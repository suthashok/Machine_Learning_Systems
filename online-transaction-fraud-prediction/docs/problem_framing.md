
# Fraud Detection System – Production Problem Framing

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


