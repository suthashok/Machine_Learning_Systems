# Feature Engineering

## Goal

This doc captures the main feature families we want for the IEEE-CIS fraud problem.

The focus is not “add more features.”
The focus is to build features that reflect how fraud actually shows up:
- repeated risky entities
- sudden changes in behavior
- high transaction velocity
- identity inconsistency
- sparse / missing device signals

Main rule throughout: **no leakage**.

---

## Guiding principle

Raw transaction columns are useful, but they are usually not enough on their own.

Fraud is more about:
- how this transaction compares to past behavior
- whether this entity has shown risk before
- whether activity is happening too fast
- whether identity signals are consistent

So the feature set should move from **row-level values** to **behavioral signals**.

---

## Leakage rule

For any transaction at time `t0`, features must only use data available at or before `t0`.

That means:
- no future transactions in aggregates
- no future labels
- no global target encoding
- no train/test mixing in historical features

If this rule is broken, validation results will look better than reality.

---

## Feature families

### 1. Raw transaction features

These are the base features from the row itself.

Examples:
- `TransactionAmt`
- `ProductCD`
- `card1` to `card6`
- `addr1`, `addr2`
- `P_emaildomain`, `R_emaildomain`

These are useful as context, but usually need transforms or aggregation to become strong fraud signals.

Likely handling:
- log transform for amount
- categorical encoding for low / medium cardinality columns
- careful treatment of high-cardinality columns

---

### 2. Entity history features

Fraud tends to repeat around the same cards, devices, emails, or addresses.

So for a given entity, useful features are things like:
- prior transaction count
- prior average amount
- prior max amount
- prior distinct counterpart count
- prior time since last seen

Examples:
- transactions previously seen on this card
- average amount for this email domain
- number of cards seen on this device
- number of devices used by this card

These features matter because they give a baseline.
Fraud often looks abnormal relative to an entity’s own history, not just the population average.

---

### 3. Velocity features

Velocity is one of the strongest fraud patterns.

Things worth capturing:
- transaction count in recent windows
- amount sum in recent windows
- repeated amounts in short windows
- number of entities touched in a short time

Typical windows:
- 1 hour
- 24 hours
- 7 days
- 30 days

Examples:
- card transaction count in last 1h
- device transaction count in last 24h
- amount spent by card in last 1h
- distinct cards used by device in last 7d

These help detect:
- card testing
- bursty fraud attacks
- device reuse
- abnormal spikes in activity

---

### 4. Temporal features

Fraud is often time-sensitive, so transaction order should be turned into features where possible.

Useful examples:
- hour-of-day
- day-of-week
- time since last transaction for same entity
- time since first seen for same entity

These are simple, but often useful when combined with entity history and velocity.

---

### 5. Consistency features

A lot of fraud is basically identity mismatch.

Useful checks:
- one card used across many devices
- one device used across many cards
- one card tied to many email domains
- one address tied to many cards

Examples:
- distinct devices used by card
- distinct cards seen on device
- distinct emails seen for card
- distinct addresses seen for card

These features are useful because legit behavior is usually more stable than fraudulent behavior.

---

### 6. Device / identity features

Where identity data exists, we should use it.

Useful groups:
- `DeviceType`
- `DeviceInfo`
- selected `id_*` columns

Potential signals:
- device frequency
- device-card diversity
- browser / OS style mismatch
- presence / absence of identity data

Because identity coverage is sparse, these features should be handled carefully:
- nulls are expected
- missingness may be signal
- some fields may be noisy or unstable

---

### 7. Missingness features

Missing data should be treated as its own feature family, not just a preprocessing issue.

Examples:
- device info missing flag
- browser / OS missing flag
- identity record present / absent
- count of missing identity fields

This matters because in fraud data, missingness is often not random.

---

## Encoding strategy

### Continuous features
Use standard numeric handling.
Examples:
- log transform for `TransactionAmt`
- normalization only if model choice needs it

### Low-cardinality categorical features
Use simple encoding:
- one-hot if small enough
- otherwise label / ordinal style encoding depending on model

### High-cardinality categorical features
Avoid naive one-hot.

Better options:
- frequency encoding
- count encoding
- aggregated history features
- rare bucket grouping

### Target encoding
Use only with care.

If used at all:
- compute in a leakage-safe way
- use only past data or out-of-fold logic
- never use full-dataset fraud rate directly

---

## Composite entities

Single columns like `card1` or `DeviceInfo` may be noisy.

So it is worth testing composite identifiers such as:
- card + address
- card + email domain
- card + device
- device + browser / OS style field

These can sometimes behave more like a stable pseudo-identity than any one raw column.

Not all of them will survive validation, but they are worth testing.

---

## What makes a feature worth keeping?

Before keeping a feature, check:

- does it use only past data?
- does it have enough coverage?
- is missingness manageable?
- is it stable over time?
- does it add signal beyond simpler alternatives?
- can we explain why it might help?

The point is not to maximize feature count.
It is to keep features that are both useful and defensible.

---

## Production lens

Even though this project is offline, features should still be designed with production logic in mind.

That means:
- feature definitions should be time-safe
- training and inference logic should match
- historical aggregates should be reproducible
- online / offline skew should be avoided

For IEEE-CIS this is mostly a design constraint, but it matters if the project is meant to reflect real fraud-system thinking.

---

## Feature direction for this project

The first feature groups to prioritize are:

1. raw transaction context
2. amount transforms
3. entity history
4. velocity
5. consistency checks
6. missingness indicators
7. selected identity / device features
8. composite entity features where useful

That gives a good balance between:
- baseline tabular performance
- fraud-specific behavior modeling
- leakage-safe implementation

---

## Next

`5_data_leakage_prevention.md`