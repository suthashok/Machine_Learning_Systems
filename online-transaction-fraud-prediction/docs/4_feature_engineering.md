# Feature Engineering

## What We're Building

Raw data is useless by itself. We need to transform it into signals that actually catch fraud. This doc covers the features we'll build and why.

Bottom line: fraud detection is all about **behavior, velocity, and entity risk**. Not just "is this transaction big?" but "is this transaction weird for this card/device/email?"

---

## Golden Rule - No Leakage

Everything we compute must use only data available **before or at the exact moment of this transaction**.

Sounds obvious, but it's easy to screw up:
- Can't use future transactions to compute averages
- Can't use tomorrow's fraud label in today's features
- When aggregating historical data, cutoff is strictly `t0`

If we leak future info, model looks great in training and falls apart in production. Seen it happen.

---

## Feature Categories

### 1. Basic Transaction Features

The raw stuff, maybe with some transforms:
- `TransactionAmt` - amount (try log transform, amounts are usually skewed)
- `ProductCD` - product type (categorical)
- `card1` through `card6` - card fingerprints
- `addr1`, `addr2` - billing/shipping-ish

Nothing fancy here, just baseline context.

---

### 2. Entity Risk Features (This is where it gets interesting)

Fraud clusters around specific cards, devices, emails. So we need to track historical behavior for each entity.

**For each card, what's its history?**
- `card_txn_count_30d` (transaction volume)
- `card_avg_amount_30d` (typical spending)
- `card_fraud_rate_30d` (historical risk - careful with target encoding)
- `card_max_amount_7d` (biggest purchase recently)

**For email domains:**
- `domain_txn_count_30d`
- `domain_fraud_rate_30d`
- `domain_avg_amount_30d`

**For devices:**
- `device_txn_count_24h`
- `device_distinct_cards_7d` (how many different cards used from this device)

The idea: if this card normally does 2 transactions a week and suddenly does 10 today, something's off.

---

### 3. Velocity - Speed Matters

Fraudsters move fast before cards get blocked. Velocity features catch this.

**Time windows (need to experiment with these):**
- 1 hour - catches rapid testing bursts
- 24 hours - daily patterns
- 7 days - weekly trends

**Examples:**
- `card_txn_count_1h`
- `card_txn_count_24h`
- `device_txn_count_1h`
- `email_txn_count_24h`

**Also useful:**
- `card_amount_sum_1h` (total $ volume in last hour)
- `card_txn_count_same_amount_24h` (repeated amounts - card testing pattern)

---

### 4. Temporal Patterns

When things happen matters.

- `hour_of_day` (fraudsters work nights?)
- `day_of_week` (weekends different?)
- `time_since_last_txn` (gap since last activity from this card)

Legit users have routines. Fraudsters have bursts then silence.

---

### 5. Identity Consistency - The "Does This Make Sense?" Features

Fraudsters often mix and match stolen credentials. Legit users tend to be consistent.

**For this card, how many different...?**
- `distinct_devices_per_card_30d`
- `distinct_emails_per_card_30d`
- `distinct_addresses_per_card_30d`

**For this device, how many different...?**
- `distinct_cards_per_device_7d`
- `distinct_emails_per_device_7d`

If one card suddenly shows up on 5 different devices this week, that's suspicious.

**Also useful:**
- `browser_os_mismatch_flag` (Safari on Windows? possible but odd)

---

### 6. Device Risk

Devices can be burned (used for fraud) or clean.

- `device_txn_count_24h`
- `device_fraud_rate_30d` (historical risk of this device)
- `device_avg_amount_7d`
- `device_velocity_hour` (how fast this device transacts)

Also look at device clusters - if a device is hitting multiple cards, probably fraud.

---

### 7. Missingness as Signal

Remember from EDA? Missing data might be intentional.

- `is_deviceinfo_missing`
- `is_browser_missing`
- `is_identity_field_missing`
- `is_email_domain_missing`

If fraudsters block tracking, missing fields become features.

---

## Encoding Strategy - Keep It Simple

| Type | Approach | Notes |
|------|----------|-------|
| Continuous | Log transform, standardize | Amounts usually right-skewed |
| Low-cardinality categorical | One-hot | If <10 categories |
| High-cardinality categorical | Frequency encoding | Count of occurrences / total |
| Target encoding | Use with CAUTION | Leakage risk - must use expanding window |
| Boolean | 0/1 flags | Fine as-is |

**On target encoding**: If you encode "average fraud rate for this card1 value", you MUST compute it using only historical data before `t0`. No global averages.

---

## Window Sizes - What Works

Starting points (will tune later):

| Window | What It Catches |
|--------|-----------------|
| 1 hour | Card testing, rapid bursts |
| 6 hours | Short fraud campaigns |
| 24 hours | Daily patterns |
| 7 days | Weekly trends |
| 30 days | Medium-term behavior |

Need to be careful with very long windows (90d+) - data sparsity becomes an issue.

---

## High Cardinality - Too Many Values

Features like `card1`, `DeviceInfo`, `id_30` have thousands of unique values. Can't one-hot encode.

**Options:**
- **Frequency encoding** - replace value with its count in training data
- **Count encoding** - transactions per this value
- **Aggregate features** - instead of raw device ID, use `device_txn_count_24h`
- **Rare value grouping** - bin values appearing <5 times into "rare" bucket

---

## Production Reality Check

Training is easy. Production is where this gets hard.

**Training pipeline:**
```python
df_train = compute_features(transactions, cutoff_time=t0)
```

**Production pipeline (real-time):**  
Need same features, computed same way, in milliseconds. Can't scan 30 days of history at inference time - too slow. Need pre-computed aggregates in a feature store.

**Ideal setup:**

- Batch jobs pre-compute daily aggregates
    
- Real-time service fetches pre-computed features + adds online velocity
    
- Feature definitions are identical between training and serving
    

Otherwise you get training/serving skew. Model works in notebook, fails in prod.

---

## Quick Validation Checklist

Before modeling, sanity check each feature:

- Any leakage? (does it use future data?)
    
- Distribution stable across time?
    
- Missing rate acceptable? (if 99% null, probably useless)
    
- Cardinality reasonable? (if every row has unique value, model can't generalize)
    
- Makes intuitive sense? (if you can't explain why it might work, maybe skip it)
    

---

## Summary - What We're Building

|Category|Example|Why|
|---|---|---|
|Velocity|`card_txn_count_1h`|Catches testing bursts|
|Entity risk|`card_avg_amount_30d`|Baseline for anomaly detection|
|Identity consistency|`distinct_devices_per_card_30d`|Catches identity switching|
|Device risk|`cards_per_device_7d`|Detects mule devices|
|Missingness|`is_deviceinfo_missing`|Fraudsters block tracking|

---

## Next Up

Feature engineering done. Now we need to make damn sure we didn't accidentally leak future info.

See: [5_data_leakage_prevention.md](https://5_data_leakage_prevention.md/)

Because nothing's worse than a model that looks great but fails because of leakage.