# Task 2 · Wine Quality Prediction

Train and compare three classifiers (Random Forest, SGD, SVC) that predict whether a red wine is "good" from its physicochemical properties.

## Files

| File | Purpose |
|---|---|
| `wine_quality_prediction.ipynb` | The full analysis: EDA, imbalance discussion, binning, training, evaluation, comparison, conclusion |
| `req.txt` | Python dependencies |
| `winequality-red.csv` | The dataset (UCI Wine Quality, red; semicolon-separated, 1,599 rows) |
| `REPORT_wine_quality.md` | Written report of method and results |

The notebook is saved without outputs; run it to generate all tables and charts.

## Quick start

```bash
pip install -r req.txt
jupyter notebook wine_quality_prediction.ipynb     # then: Kernel -> Restart & Run All
```

Keep `winequality-red.csv` in the same folder as the notebook (or in a `data/` sub-folder). Runtime is about one minute (most of it Random Forest tuning).

## Configuration (first code cell)

| Setting | Default | Meaning |
|---|---|---|
| `DATA_PATH` | `winequality-red.csv` | Dataset location |
| `RANDOM_STATE` | `42` | Seed for splits and models |
| `TEST_SIZE` | `0.20` | Share of data held out for testing |
| `DROP_DUPLICATES` | `True` | Remove exact duplicate rows before splitting (240 in this file) |
| `BINNING` | `"binary"` | `"binary"` (good >= 7), `"three_class"` (low 3-4 / medium 5-6 / high 7-8) or `"raw"` |
| `CV_FOLDS` | `5` | Stratified cross-validation folds used for tuning |

## What the notebook does

1. Loads and inspects the data, checks the class distribution of quality scores.
2. EDA: histograms of all 11 features, boxplots by quality, correlation heatmap.
3. Discusses class imbalance and why accuracy alone is misleading.
4. Bins the target and justifies the choice, with a quick experiment on raw vs 3-class vs binary.
5. Stratified 80/20 train/test split.
6. Trains Random Forest, SGD and SVC in pipelines (scaling for SGD and SVC), tuned with `GridSearchCV` on macro-F1.
7. Evaluates each: accuracy, classification report, confusion matrix.
8. Random Forest feature importance (impurity and permutation).
9. Side-by-side comparison table and chart, plus a repeated cross-validation stability check.
10. Conclusion, then saves the best pipeline to `wine_quality_model.joblib` and demonstrates `predict_quality()`.

## Headline results (seed 42, binary target)

| Model | Accuracy | Macro F1 | F1 on "good" wines |
|---|---|---|---|
| Random Forest | 0.875 | 0.728 | 0.53 |
| SVC | 0.871 | 0.723 | 0.52 |
| SGD | 0.772 | 0.672 | 0.49 |

Recommended model: **Random Forest**. Exact figures can shift slightly with other library versions.

## Notes and limitations

- Only 37 "good" wines are in the test set, so the Random Forest vs SVC difference is not statistically significant.
- Red wine only; do not apply the model to white wine without re-validation.
- Taster scores are subjective and the 11 lab measurements cannot capture everything, so there is a ceiling on achievable accuracy.
