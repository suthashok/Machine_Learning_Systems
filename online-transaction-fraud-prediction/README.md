# Online Transaction Fraud Detection

A machine learning project focused on solving the IEEE-CIS Fraud Detection problem.

## Overview

This project builds an offline fraud detection pipeline using the IEEE-CIS dataset.

The current focus is:
- understanding the dataset
- building leakage-safe features
- using time-aware validation
- training a strong transaction-level fraud model
- analyzing model score cutoffs

The goal is to build the IEEE-CIS solution cleanly first.
Production-specific extensions are kept separate.

## Dataset

Dataset: IEEE-CIS Fraud Detection (Kaggle)  
https://www.kaggle.com/competitions/ieee-fraud-detection/data

Files used:
- `train_transaction.csv`
- `train_identity.csv`
- `test_transaction.csv`
- `test_identity.csv`

High-level shape:
- training set: ~590K transactions
- test set: ~506K transactions
- fraud rate in train: ~3.5%

## Documentation

The docs follow the same order as the project flow:

1. `docs/1_problem_framing.md`
2. `docs/2_data_sources_and_structure.md`
3. `docs/3_exploratory_data_analysis.md`
4. `docs/4_feature_engineering.md`
5. `docs/5_data_leakage_prevention.md`
6. `docs/6_modelling_and_calibration.md`
7. `docs/7_threshold_optimization.md`

A separate doc will cover what changes are needed to move from the IEEE-CIS setup to a production fraud system.

## Project Status

🚧 In progress

Current phase:
- design docs completed
- baseline implementation is the next step

## Getting Started

1. Download the IEEE-CIS dataset from Kaggle
2. Place the files under `data/raw/`
3. Start with the first data join and split step