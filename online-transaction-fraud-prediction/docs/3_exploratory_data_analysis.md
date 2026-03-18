# Online Transaction Fraud Detection System – Exploratory Data Analysis


## What We're Trying to Figure Out

Before jumping into feature engineering, we need to understand what we're dealing with. Main questions:

- How much fraud is actually in this dataset? (~3-4% based on competition docs)
- Does fraud look different across transaction types?
- Any obvious patterns that separate fraud from legit?
- What might be useful for feature engineering?

Bottom line: looking for **where risk concentrates**. Fraud isn't random - it clusters around certain cards, devices, email domains, etc.

---

## Dataset at a Glance

- **~590k transactions** - decent size
- **~3.5% fraud rate** - heavily imbalanced (typical - if fraud was common, the business would have bigger problems)

This means:
- Accuracy is useless (99% accuracy just by predicting "not fraud" - meaningless)
- Need to think about precision/recall tradeoffs
- Eventually need to optimize for expected value ($ saved vs $ pissed-off customers from false declines)

---

## Missing Data - First Red Flag

Quick scan of missingness:

Identity fields are a disaster - lots of nulls:
- `DeviceInfo` - missing everywhere
- `id_30`, `id_31` - browser/OS signals, also missing a ton
- Most of the `id_*` columns have huge gaps

Why? Real world problems:
- Device fingerprinting doesn't always work
- Privacy settings / tracking blockers
- Some payment flows just don't collect this stuff

**Interesting observation**: Missingness itself might be predictive. If fraudsters are deliberately blocking tracking, rows with null device info might have higher fraud rates. Need to check this.

---

## Transaction Amount - Follow the Money

Looking at `TransactionAmt`:

Questions running through my head:
- Do fraudsters go for small amounts (card testing) or big tickets (maximize value)?
- Is there a "sweet spot" where fraud concentrates?
- Are legit transactions clustered differently?

Typical fraud patterns from experience:
- **Small transactions** (<$10) - card testing, checking if cards work
- **Medium transactions** ($50-500) - goods that are easy to resell (electronics, gift cards)
- **Large transactions** - less common but higher impact

Need to bucket these and check fraud rates per bucket.

---

## Where Fraud Clusters - Entity Risk

Fraud doesn't spread evenly. It pools around specific cards, devices, email domains. This is where we need to look:

| Entity | Features |
|--------|----------|
| Card | `card1` - `card6` |
| Email domain | `P_emaildomain`, `R_emaildomain` |
| Address | `addr1`, `addr2` |
| Device | `DeviceType`, `DeviceInfo` |

For each one, calculate:

```
fraud_rate = fraud_count / total_transactions_for_that_entity
```


Look for:
- Entities with crazy high fraud rates (red flags)
- Entities with huge volume but moderate fraud (impact)
- Rare entities that are 100% fraud (probably compromised)

These become features later - like "historical fraud rate for this card1 value".

---

## Timing Patterns - Fraud Comes in Waves

Using `TransactionDT` (relative time, but ordering is what matters):

Check:
- Transaction volume over time
- Fraud rate over time

Fraud campaigns usually hit in **bursts**. If you see fraud rate spike on Tuesday then drop back down, someone was probably running cards.

This is useful for:
- Detecting attack windows
- Building velocity features ("how many transactions from this card in last hour")

---

## Device Signals - Fraudsters Get Sloppy

When fraudsters use emulators, bots, or spoofed devices, the device fingerprints often look weird.

Look at:
- `DeviceType` - mobile vs desktop fraud rates
- `DeviceInfo` - specific device models
- Browser/OS combos that don't make sense (like "Safari on Windows" - possible but suspicious)

Common red flags:
- Device fingerprints that appear too frequently
- Emulated browser environments
- Mismatched OS/browser combos
- Unusual screen resolutions (bots sometimes use weird defaults)

---

## Correlations - But Take with a Grain of Salt

With 300+ V features, correlations can help spot redundancy. But fraud patterns are rarely linear - two features might be uncorrelated but highly predictive when combined.

Mainly using this to:
- Spot obvious duplicates
- Group related features
- Avoid feeding the model 50 versions of the same signal

---

# The 5 Things We Actually Check First

Every fraud team runs these same 5 analyses. They're quick and usually tell you 80% of what you need to know.

---

## 1. Fraud Rate by Amount Bucket

Bin amounts:
- $0-10
- $10-50  
- $50-100
- $100-500
- $500+

Check fraud rate per bucket.

What this tells you:
- Is card testing happening? (small buckets elevated)
- Where's the biggest loss exposure? (medium buckets might have highest fraud volume)
- Thresholds for rules later

---

## 2. Top Risky Entities

Rank by fraud rate:
- `card1` values
- Email domains
- Device types
- Addresses

Look at both:
- **Highest fraud rate** (even if low volume)
- **Highest fraud volume** (rate * count)

This shows you where fraud is concentrating. Usually a handful of entities drive most of the risk.

---

## 3. Velocity - How Fast Things Happen

Count transactions per entity over time windows:
- Transactions per card in last hour/day
- Transactions per device in last hour
- Transactions per email in last day

Fraudsters:
- Hit cards fast before they get blocked
- Test multiple cards from same device
- Run sequences of small amounts

These are your strongest features.

---

## 4. Fraud Rate Over Time

Plot fraud rate by hour/day.

Look for:
- Spikes (fraud campaigns)
- Drops (something changed - new rule? fraudsters moved on?)
- Patterns (fraudsters work nights/weekends?)

This matters later for monitoring drift in production.

---

## 5. Missingness as a Feature

Compare fraud rate when fields are null vs present:
- DeviceInfo missing? Higher fraud rate?
- Browser attributes missing?
- Identity fields missing?

If missing = risky, that's a feature we can use directly.

---

# What We Learned

Quick takeaways before moving to feature engineering:

1. **Fraud clusters** - entity-level risk is real. Need to capture this.
2. **Velocity matters** - how fast things happen is as important as what happens.
3. **Missing data might be meaningful** - not always, but worth checking.
4. **Amount patterns exist** - small transactions for testing, medium for resale.
5. **Fraud is bursty** - time patterns matter.

---

# Next Up

Feature engineering time. Based on this, we'll build:
- Entity risk scores
- Velocity features (counts over time windows)
- Behavioral aggregations
- Identity consistency checks

See: [4_feature_engineering.md](4_feature_engineering.md)
