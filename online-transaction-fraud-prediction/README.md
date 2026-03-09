# Online Transaction Fraud Detection

A machine learning system to detect fraudulent online transactions using the IEEE-CIS Fraud Detection dataset.

## Overview

This project implements a fraud detection model with proper consideration for business economics and production constraints. The focus is on building a classifier that maximizes profit per transaction, not just minimizing fraud.

**Key principle:** Approve transactions that make us money, decline those that lose money—accounting for both fraud risk and legitimate customer impact.

## Documentation

- **[Problem Framing](docs/1_problem_framing.md)** – Business objective, decision rules, and constraints. Start here to understand what we're optimizing for and why.

## Dataset

**IEEE-CIS Fraud Detection** (Kaggle Competition)

https://www.kaggle.com/competitions/ieee-fraud-detection/data

- Training set: 590K transactions with fraud labels
- Test set: 506K transactions  
- Features: 433 (transaction and identity information)
- Class distribution: ~3.5% fraudulent (severe imbalance)

## Project Status

🚧 Under development

## Getting Started

Download the dataset from Kaggle and place in the `data/` directory.