# Project findings — IEEE-CIS fraud detection

## Scope and source

The project uses transaction and optional identity features from Kaggle's IEEE-CIS Fraud Detection competition. `TransactionID` joins the transaction and identity tables using a left join, preserving transactions with no identity record. The target is `isFraud`.

## Data characteristics

| Property | Value |
|---|---:|
| Total labelled transactions | 590,540 |
| Fraudulent training examples | 20,663 |
| Fraud prevalence | ~3.50% |
| Chronological training period | 472,432 transactions |
| Chronological validation period | 118,108 transactions |
| Fraud cases in validation period | 4,064 |
| Unlabelled Kaggle test transactions | 506,691 |

Missing values are retained for LightGBM. The EDA notebook also examines transaction amounts, product categories, card types and feature availability. These descriptive associations are not causal evidence.

## Modelling

Three LightGBM classifiers were compared on a chronological validation split:

| Model | Features | ROC-AUC | PR-AUC | Best tested threshold | Precision | Recall | F1 |
|---|---:|---:|---:|---:|---:|---:|---:|
| Baseline | 26 | 0.8770 | 0.4398 | 0.20 | 0.5353 | 0.4085 | **0.4634** |
| Top 20 features | 20 | 0.8762 | 0.4341 | 0.20 | 0.5299 | 0.4058 | 0.4596 |
| Class-weighted | 26 | 0.8732 | 0.4292 | 0.50 | 0.1982 | 0.6543 | 0.3043 |

The baseline was selected by maximum F1 over thresholds `0.10, 0.20, 0.30, 0.40, 0.50`. The class-weighted variant had higher recall at its selected threshold but considerably lower precision and F1.

### Chosen model: original notebook validation output

| Metric | Value |
|---|---:|
| ROC-AUC | 0.876964 |
| PR-AUC (average precision) | 0.439794 |
| Precision (threshold 0.20) | 0.535311 |
| Recall (threshold 0.20) | 0.408465 |
| F1 (threshold 0.20) | 0.463364 |

![Confusion matrix](reports/confusion_matrix.svg)

| | Predicted legitimate | Predicted fraud |
|---|---:|---:|
| Actual legitimate | 112,603 | 1,441 |
| Actual fraud | 2,404 | 1,660 |

The selected threshold detects 1,660 of the 4,064 validation fraud cases. 1,441 legitimate transactions are false alarms. These are historical holdout figures, not counts from Kaggle's hidden test set.

### Kaggle test result

The model was retrained on all 590,540 labelled examples and assigned probabilities to 506,691 unlabelled Kaggle test transactions. The user-reported, post-deadline submission received:

- **Public ROC-AUC:** 0.896908
- **Private ROC-AUC:** 0.870183

Kaggle scores evaluate the **ranking** of transactions across private/public hidden-label subsets. They do not give precision, recall, F1 or classification accuracy on those subsets. The binary threshold 0.20 was not used to construct the submission probabilities.

## Feature importance

The top ranked features by LightGBM gain include `card1`, `C4`, `C1`, `D2`, and `R_emaildomain`. Gain importance describes model split contribution, not feature causality. A chart and extract are in `reports/`.

## Limitations

1. Repeated model/threshold selection on the same holdout can bias local performance estimates upward; use a second future-time holdout or rolling-time validation for stronger evaluation.
2. Severe class imbalance and delayed fraud labels affect precision-recall interpretation.
3. All input data is from a competition; deployment may require better feature availability, monitoring, probability calibration, and drift testing.
4. Raw Kaggle files and the trained model were not part of the uploaded notebook and are not redistributed in this repository.
5. The GitHub-prepared notebook is refactored for relative paths, ordered dependencies and smaller outputs; it has not been rerun end-to-end against Kaggle files in this environment.

## Model artifacts

Running the notebook locally exports `outputs/submission.csv`, `outputs/saved_model/fraud_detection_model.joblib`, `outputs/reports/model_comparison.csv`, and other evaluation charts. These files are intentionally ignored by Git to avoid accidentally committing competition data or large binaries.
