# Data Sources and Structure

## What's in this document
This doc covers the raw data sources we're using for the fraud detection system - specifically the IEEE CIS dataset.

## Quick Overview
We're working with two main tables from the IEEE competition:
- `train_transaction.csv` - core transaction data
- `train_identity.csv` - device/identity info (when available)

Each row is one payment transaction.

## The Transaction Table (`train_transaction.csv`)
This is our main table.

### Key Fields
- **TransactionID**: Unique ID for each transaction
- **TransactionDT**: Seconds elapsed from a starting point
- **isFraud**: 1 = Fraud, 0 = Legit

### Feature Groups
- **Transaction basics**: TransactionAmt, ProductCD, card1-card6
- **Address stuff**: addr1, addr2
- **Email domains**: P_emaildomain, R_emaildomain
- **Time-based**: D1-D15
- **The V features**: V1-V339 (anonymized)

## The Identity Table (`train_identity.csv`)
Device data. Not all transactions have identity records.

### What's in Identity Table
- **Device info**: DeviceType, DeviceInfo
- **Browser/OS**: id_30 through id_41
- **Risk signals**: id_01-id_38 (anonymized)

## How Tables Connect

```python
import pandas as pd
txn = pd.read_csv('train_transaction.csv')
idty = pd.read_csv('train_identity.csv')
df = txn.merge(idty, on='TransactionID', how='left')
```


## Data Quality Notes
- Identity table has lots of nulls (expected)
- V features and id_* features are black boxes
- TransactionDT is relative time only

## Dataset Stats
- ~590,000 transactions
- ~3.5% fraud rate
- Training set only

## Main Entities We Can Track
- Cards (card1-card6)
- Devices (when identity exists)
- Email domains
- Addresses (addr1, addr2)

---

In next document [3_exploratory_data_analysis.md](3_exploratory_data_analysis.md) we will be exploring the above data.