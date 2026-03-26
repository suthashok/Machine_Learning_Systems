# Feature Engineering

## Goal

This doc lists the main feature groups worth building for IEEE-CIS.

The point is not to create a huge feature list.
The point is to build features that reflect how fraud usually shows up:
- risky entities
- repeat activity in a short window
- behavior that looks unusual for the same card / device / email
- missingness that may carry signal

---

## Main rule

For any transaction at time `t0`, features must use only data available at or before `t0`.

No future rows.
No future labels.
No full-dataset aggregates pretending to be history.

If this rule breaks, validation results will be misleading.

---

## 1. Raw transaction features

Start with the basic row-level fields:
- `TransactionAmt`
- `ProductCD`
- `card1` to `card6`
- `addr1`, `addr2`
- `P_emaildomain`, `R_emaildomain`

These are useful as a base, but usually not enough on their own.

Likely handling:
- log transform for amount
- basic encoding for categoricals
- rare bucket for very sparse values if needed

---

## 2. Entity history features

Fraud often repeats around the same cards, devices, addresses, or email patterns.

Useful examples:
- prior transaction count for a card
- prior average amount for a card
- prior max amount for a card
- prior transaction count for an email domain
- prior transaction count for a device
- time since last transaction for the same entity

These help because fraud often looks unusual relative to the entity’s own history.

---

## 3. Velocity features

This is one of the most important groups.

Useful signals:
- transaction count in last 1h
- transaction count in last 24h
- amount sum in last 1h
- repeated amount count in last 24h
- number of cards seen on the same device in a short window

Typical windows to test:
- 1 hour
- 24 hours
- 7 days
- 30 days

These help catch bursty behavior and card testing patterns.

---

## 4. Temporal features

Even with relative time, a few simple time features are useful.

Examples:
- hour-of-day
- day-of-week
- time since last transaction
- time since first seen for an entity

These are simple, but often useful when combined with entity history.

---

## 5. Consistency features

A lot of fraud is basically mismatch.

Useful checks:
- how many devices has this card used?
- how many cards has this device seen?
- how many emails are tied to this card?
- how many addresses are tied to this card?

Examples:
- distinct devices per card
- distinct cards per device
- distinct emails per card
- distinct addresses per card

Normal behavior is usually more stable than fraud behavior.

---

## 6. Device / identity features

Where identity data exists, use it.

Useful fields:
- `DeviceType`
- `DeviceInfo`
- selected `id_*`

Possible signals:
- how often the device appears
- how many cards use the same device
- whether identity data is present at all

Since identity coverage is incomplete, these need careful null handling.

---

## 7. Missingness features

Missing data should be treated as signal, not just a cleanup issue.

Examples:
- `DeviceInfo` missing flag
- identity-present flag
- browser / OS missing flag
- count of missing identity fields

This is worth testing because missingness may not be random here.

---

## 8. Composite entities

Single fields like `card1` or `DeviceInfo` may be noisy on their own.

So it is worth testing combinations like:
- card + address
- card + email domain
- card + device

Some of these may behave more like a stable pseudo-identity.

---

## What to keep

Before keeping a feature, ask:
- is it leakage-safe?
- does it have enough coverage?
- is it stable across time?
- does it add signal beyond simpler alternatives?

The goal is not feature count.
The goal is useful signal.

---

## Takeaway

The feature set should move beyond raw row values and focus on:
- entity history
- velocity
- consistency
- missingness
- a few simple time features

That is the most likely path to a strong IEEE-CIS model.

---

## Next

[5_data_leakage_prevention.md](5_data_leakage_prevention.md)