# Report · Wine Quality Prediction

## 1. Objective

Predict the quality of red *Vinho Verde* wine from 11 physicochemical measurements, and compare three classifiers: Random Forest, Stochastic Gradient Descent (SGD) and Support Vector Classifier (SVC).

## 2. Data

- 1,599 wines, 11 continuous features (acidity, sugar, chlorides, sulfur dioxide, density, pH, sulphates, alcohol, ...) and an integer quality score (observed range 3-8).
- No missing values. **240 exact duplicate rows** were found and removed, leaving 1,359 rows, so that identical rows cannot appear in both the training and test sets and inflate the score.
- Raw score distribution: 3 → 10, 4 → 53, 5 → 681, 6 → 638, 7 → 199, 8 → 18. Scores 5 and 6 dominate; the extremes are rare.

## 3. Exploratory analysis

- Several features are strongly right-skewed (residual sugar skew 4.5, chlorides 5.5, sulphates 2.4, total sulfur dioxide 1.5).
- **Alcohol** has the strongest positive correlation with quality, followed by sulphates and citric acid; **volatile acidity** has the strongest negative one. No feature exceeds |r| of about 0.5, so the signal is spread across features and partly non-linear.
- Clear multicollinearity: fixed acidity correlates with citric acid (0.67), density (0.67) and pH (-0.69); free and total sulfur dioxide correlate at 0.67.

## 4. Class imbalance and target engineering

Imbalance makes accuracy misleading (predicting "average" is usually right), biases models toward majority classes, and leaves too few rare-class examples to learn or even evaluate (a stratified test split holds only a handful).

Three target definitions were compared with an identical default Random Forest:

| Scheme | Accuracy | Worst-class F1 | Observation |
|---|---|---|---|
| Raw (6 classes) | 0.614 | 0.0 | rare scores never recovered |
| Three-class (low / medium / high) | 0.835 | 0.0 | the tiny "low" class is never recovered |
| **Binary (good >= 7 vs not)** | 0.893 | 0.473 | every class has enough samples to learn |

Accuracy is not comparable across schemes (fewer classes is easier), so the smallest-class size and worst-class F1 are the informative columns. **Binary was chosen**: it is the natural business question ("is this a good wine?") and leaves about 14% positives, handled with class weights and macro-averaged metrics.

## 5. Method

- Stratified 80/20 train/test split (random seed 42).
- All models use `class_weight="balanced"`; SGD and SVC sit in a pipeline with `StandardScaler`.
- Hyper-parameters tuned with 5-fold stratified `GridSearchCV` on the training set only, optimising **macro-F1**:
  - Random Forest: `n_estimators`, `max_depth`, `min_samples_leaf`
  - SGD: `loss` (hinge, log-loss, modified Huber), `alpha`, `penalty`
  - SVC: RBF kernel, `C`, `gamma`

## 6. Results (272 test wines, 37 of them "good")

| Model | Accuracy | Balanced acc. | Macro precision | Macro recall | Macro F1 | F1 "good" | Recall "good" | CV macro F1 (train) |
|---|---|---|---|---|---|---|---|---|
| Random Forest | 0.875 | 0.723 | 0.733 | 0.723 | 0.728 | 0.53 | 0.51 | 0.708 |
| SVC | 0.871 | 0.721 | 0.726 | 0.721 | 0.723 | 0.52 | 0.51 | 0.704 |
| SGD | 0.772 | 0.788 | 0.658 | 0.788 | 0.672 | 0.49 | 0.81 | 0.684 |

**Stability check.** Repeated stratified 5-fold cross-validation on the full data (3 repeats) gave macro-F1 of 0.722 (Random Forest), 0.690 (SVC) and 0.687 (SGD), with a spread of roughly 0.03. Random Forest ranks first on both the single split and the repeated cross-validation.

**Feature importance.** Impurity-based and permutation importance agree on the main drivers: alcohol, sulphates, citric acid, volatile acidity and total sulfur dioxide, consistent with the correlation analysis. Correlated acidity features share importance, so exact rankings should not be over-read.

## 7. Discussion

- **Random Forest** has the best or tied-best score on every headline metric, needs no scaling and is robust to skew and outliers.
- **SVC** is practically tied on the test split (one or two wines out of 272) and trains and predicts faster, but falls behind in repeated cross-validation.
- **SGD** is limited by its linear boundary: accuracy is about 10 points lower. It finds the most good wines (81% recall) but only about a third of the wines it labels "good" truly are (precision 0.35). It would suit a "never miss a good wine, false alarms are cheap" use case.

## 8. Conclusion and recommendation

Deploy the **Random Forest**: best accuracy and macro-F1, most stable across resampling, interpretable through feature importance. The tuned pipeline is saved as `wine_quality_model.joblib`.

## 9. Limitations

- Performance on good wines is moderate (F1 about 0.5): roughly half of good wines are missed and half of those flagged are not good.
- Only 37 good wines in the test set, so the Random Forest vs SVC difference is within noise.
- Quality labels are subjective, and grape, vineyard and ageing are not measured.
- Red wine only; duplicates were removed (keeping them would raise scores without improving the model).

## 10. Possible next steps

Gradient boosting (XGBoost / LightGBM), probability-threshold tuning for the good class, adding the white-wine dataset, and collecting more examples of top-quality wines.

## 11. Reproducibility

Python with pandas, numpy, scikit-learn, seaborn, matplotlib (see `req.txt`). Seed 42 throughout. Exact numbers can differ slightly across library versions.
