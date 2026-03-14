# Data Sources and Structure

This document describes the raw data sources used for building the online transaction fraud detection system. In this case it is IEEE CIS dataset.


## 1. Overview

The IEEE dataset consists of two primary tables:

- `train_transaction.csv`
    
- `train_identity.csv`
    

These tables contain transaction-level attributes and associated identity/device information used to predict fraudulent transactions.

Each transaction represents a single online payment authorization event.

---

# 2. Data Sources

## 2.1 Transaction Table

File:

train_transaction.csv

This is the primary dataset containing transaction-level information.

Each row represents **one payment transaction**.

### Primary Key

TransactionID

Unique identifier for each transaction.

### Timestamp

TransactionDT

Represents time elapsed since a reference start point.

For modeling purposes:

t0 = transaction authorization time

All features must be derived using data available **at or before t0**.

---

### Target Variable

isFraud

Binary indicator:

|Value|Meaning|
|---|---|
|1|Fraudulent transaction|
|0|Legitimate transaction|

Fraud is defined as transactions that later resulted in **unauthorized chargebacks or fraud reports**.

---

### Key Feature Categories

The transaction table contains several groups of features:

#### Transaction Details

Examples:

TransactionAmt  
ProductCD  
card1–card6

These describe the payment instrument and transaction attributes.

---

#### Address Information

Examples:

addr1  
addr2

Approximate billing or geographic attributes associated with the transaction.

---

#### Email Domain

Examples:

P_emaildomain  
R_emaildomain

Domain of purchaser and recipient email addresses.

---

#### Behavioral / Temporal Signals

Examples:

D1–D15

Relative timing signals related to previous activity.

---

#### Engineered / Obfuscated Variables

Examples:

V1–V339

These variables represent anonymized engineered features derived by the dataset creators.

Exact semantics are not publicly disclosed.

---

# 2.2 Identity Table

File:

train_identity.csv

This table contains **device and identity attributes** associated with transactions.

It links to the transaction table via:

TransactionID

Not all transactions have identity records.

This reflects real-world scenarios where device fingerprinting or identity signals may be partially available.

---

### Identity Feature Categories

Examples include:

#### Device Information

DeviceType  
DeviceInfo

Information about the device used during the transaction.

---

#### Browser and OS Signals

Examples:

id_30  
id_31  
id_33

Attributes related to browser, operating system, and display characteristics.

---

#### Identity Risk Signals

Examples:

id_01 – id_38

These represent anonymized identity verification signals.

---

# 3. Table Relationships

The transaction and identity tables are joined using:

TransactionID

Join type:

LEFT JOIN

Transaction table is the **primary dataset**, while identity attributes provide **additional signals when available**.

---

Example join:

transaction_data  
LEFT JOIN identity_data  
ON TransactionID

---

# 4. Missing Data Characteristics

Identity attributes are missing for many transactions.

This reflects realistic operational scenarios where:

- device fingerprinting may fail
    
- customers may block tracking signals
    
- certain payment channels do not collect device data
    

Handling missingness will be addressed during feature engineering.

---

# 5. Temporal Interpretation of TransactionDT

`TransactionDT` represents seconds elapsed from a dataset-specific start time.

While the exact reference timestamp is not publicly disclosed, the relative ordering of transactions enables:

- chronological model training
    
- temporal validation splits
    
- behavioral feature generation
    

For modeling purposes:

t0 = transaction timestamp derived from TransactionDT

---

# 6. Data Volume

The dataset contains approximately:

590k transactions

Fraud rate is approximately:

~3.5%

This class imbalance reflects real-world fraud detection scenarios where fraudulent transactions represent a small fraction of total activity.

---

# 7. Key Modeling Entities

The dataset contains signals related to multiple behavioral entities:

|Entity|Example Features|
|---|---|
|Card|card1–card6|
|Device|DeviceType, DeviceInfo|
|Email|email domains|
|Address|addr1, addr2|

These entities enable construction of behavioral and identity-based features for fraud detection.

---
