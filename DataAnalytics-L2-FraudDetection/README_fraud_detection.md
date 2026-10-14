# Task 3 · Credit Card Fraud Detection

A machine-learning pipeline for detecting fraudulent card transactions in an extremely imbalanced dataset (about 0.17% fraud), comparing Logistic Regression and Random Forest with different imbalance-handling strategies.

## Files

| File | Purpose |
|---|---|
| `fraud_detection.ipynb` | The full pipeline: EDA, metrics discussion, imbalance handling, models, evaluation, feature analysis, scalability |
| `req_fraud.txt` | Python dependencies (includes `imbalanced-learn` for SMOTE) |
| `REPORT_fraud_detection.md` | Written report of method and results |

**The dataset is not included** (about 150 MB). Place `creditcard_6.csv` (or the original `creditcard.csv`) next to the notebook or in a `data/` folder. Columns expected: `Time`, `V1`-`V28`, `Amount`, `Class`.

The notebook is saved without outputs; run it to generate all tables and charts.

## Quick start

```bash
pip install -r req_fraud.txt
jupyter notebook fraud_detection.ipynb      # then: Kernel -> Restart & Run All
```

The Random Forest cells take a few minutes on a laptop (each forest took about 3 minutes on a single core).

## Configuration (first code cell)

| Setting | Default | Meaning |
|---|---|---|
| `DATA_PATH` | `creditcard_6.csv` | Dataset (also tries `creditcard.csv`, in `.` and `./data`) |
| `RANDOM_STATE` | `42` | Seed |
| `TEST_SIZE` | `0.20` | Stratified test share |
| `DROP_DUPLICATES` | `True` | Remove about 1,000 exact duplicate rows |
| `THRESHOLD` | `0.50` | Decision threshold (a threshold sweep is included) |
| `N_JOBS` | `-1` | Cores for the Random Forest |
| `COST_FALSE_ALARM` | `5.0` | **Assumed** cost of reviewing one flagged transaction; change to your own |

## What the notebook does

1. Loads the data and quantifies the imbalance.
2. EDA: amount distributions fraud vs legitimate, time-of-day analysis, feature associations.
3. Explains why accuracy is misleading (includes an "always legitimate" baseline).
4. Feature engineering (`log_amount`, cyclical hour of day) and the imbalance strategy discussion.
5. Stratified train/test split so fraud appears in both sets.
6. Trains five models: Logistic Regression (no handling, `class_weight`, SMOTE) and Random Forest (no handling, `class_weight`).
7. Evaluates with precision, recall, F1, ROC-AUC, PR-AUC, confusion matrices, ROC and precision-recall curves.
8. Discusses recall vs precision and sweeps the decision threshold with a cost model.
9. Logistic Regression coefficients and Random Forest feature importance.
10. Temporal check (train on the earliest 75%, test on the latest 25%).
11. Measures throughput and latency and discusses 1 million transactions per hour.
12. Conclusion; saves `fraud_model.joblib` and demonstrates `score_transactions()`.

## Headline results (test set: 56,746 transactions, 95 frauds)

| Model | Precision | Recall | F1 | PR-AUC |
|---|---|---|---|---|
| Logistic Regression, no handling | 0.85 | 0.58 | 0.69 | 0.70 |
| Logistic Regression, `class_weight` | 0.06 | 0.87 | 0.11 | 0.68 |
| Random Forest, no handling | 0.96 | 0.74 | 0.83 | 0.80 |
| Random Forest, `class_weight` | 0.97 | 0.73 | 0.83 | 0.80 |

Recommended: **Random Forest** with a threshold chosen from business costs.

## Notes and limitations

- **SMOTE was not verified with the real library.** `imbalanced-learn` could not be installed in the environment where the notebook was tested, so that step ran with a minimal stand-in using the same k-nearest-neighbour interpolation. The code calls the standard `imblearn` API; run it yourself to get the genuine SMOTE row. The written text avoids SMOTE-specific numbers for this reason.
- Only 95 test frauds and two days of data; small differences between close models are within noise.
- `V1`-`V28` are anonymised PCA components, so most features cannot be interpreted in business terms.
- Scalability figures were measured on a single CPU core in the test environment; re-run the cell on your hardware.
