# IEEE-CIS Credit Card Fraud Detection

End-to-end fraud detection using **DuckDB, Pandas, and LightGBM** on the [IEEE-CIS Fraud Detection dataset](https://www.kaggle.com/competitions/ieee-fraud-detection/data). Includes exploratory analysis, feature selection, chronological validation, model comparison, and Kaggle test predictions.

## Results

| Evaluation | Metric | Score |
|---|---|---:|
| Kaggle public test subset | ROC-AUC | **0.896908** |
| Kaggle private test subset | ROC-AUC | **0.870183** |
| Chronological local validation | ROC-AUC | **0.876964** |
| Chronological local validation | PR-AUC (average precision) | 0.439794 |
| Chronological local validation | Precision (threshold 0.20) | 0.535311 |
| Chronological local validation | Recall (threshold 0.20) | 0.408465 |
| Chronological local validation | F1-score (threshold 0.20) | 0.463364 |

**Selected model:** baseline LightGBM, 26 features, 200 estimators, learning rate 0.05, 31 leaves, max depth 6. The 0.20 threshold is used for binary decisions; **Kaggle ROC-AUC is calculated from the submitted fraud probabilities, not from thresholded labels**.

The two Kaggle scores are from the submission marked **"Complete (after deadline)"**; they do not represent a live-competition ranking. ROC-AUC is not classification accuracy.

![Validation model comparison](reports/model_comparison.svg)

## Dataset and validation

- **Training labels:** 590,540 transactions; 20,663 frauds (~3.50%).
- **Chronological split:** first 472,432 transactions for training, later 118,108 for validation, separated using `TransactionDT`.
- **Model variants:** 26-feature baseline, class-weighted 26-feature baseline, and a 20-feature variant ranked by LightGBM gain importance.
- **Selection:** best F1-score among the three models at thresholds from 0.10 to 0.50 on the same validation set.
- **Final training:** retrained the winning model on all 590,540 labelled transactions.
- **Kaggle test inference:** 506,691 transactions, output as probabilities in `submission.csv`.

Feature selection, model selection and threshold tuning were performed on the validation period, so the reported local validation estimates may be optimistic. External Kaggle scores provide separate evidence of ranking performance. A single notebook cannot establish calibration or production readiness.

## Repository contents

```text
.
├── Fraud_Detection_Analysis.ipynb    # EDA, training, evaluation, submission
├── PROJECT_FINDINGS.md               # Methods, metrics, interpretation, limitations
├── requirements.txt
├── .gitignore
└── reports/
    ├── validation_metrics.csv
    ├── model_comparison.csv
    ├── feature_importance_top15.csv
    ├── model_comparison.svg
    ├── confusion_matrix.svg
    ├── precision_recall_curves.html
    ├── roc_curves.html
    ├── confusion_matrix.html
    └── feature_importance.html
```

The original Kaggle CSV files, saved model and submission predictions are **not included** because they were not uploaded with the notebook. The stored notebook outputs were removed to keep the repository small and avoid exposing local file paths; the four standalone HTML charts were reconstructed from saved notebook chart data and simplified for display.

## Run locally

1. Download `train_transaction.csv`, `train_identity.csv`, `test_transaction.csv` and `test_identity.csv` from the [Kaggle competition](https://www.kaggle.com/competitions/ieee-fraud-detection/data).
2. Put the CSV files in a folder named `data` beside the notebook.
3. Install dependencies and start Jupyter:

   ```bash
   python -m pip install -r requirements.txt
   jupyter lab
   ```

4. Open `Fraud_Detection_Analysis.ipynb` from the project root and run cells in order. It writes prediction, model and chart files under `outputs/` (ignored by Git).

**Environment:** Python 3.10+ recommended. The notebook was adapted from a local Windows run; **the cleaned version has not been executed end-to-end here** because the four input CSVs were not supplied.

## Interpretation

At the selected 0.20 threshold, the local validation period contains **1,660 true positives, 1,441 false positives, 2,404 false negatives and 112,603 true negatives**. Higher recall would require a different operating threshold or model; whether that is desirable depends on the operational costs of missed fraud and false alarms.

See [PROJECT_FINDINGS.md](PROJECT_FINDINGS.md) for the full analysis and limitations.
