# Report · Credit Card Fraud Detection

## 1. Objective

Build a pipeline that detects fraudulent card transactions in a heavily imbalanced dataset, treating class imbalance as the central challenge, and evaluate it with metrics that are meaningful for fraud.

## 2. Data

- European cardholder transactions over two days: 284,807 rows, features `V1`-`V28` (anonymised PCA components), `Time` (seconds since the first transaction), `Amount` and the label `Class`.
- No missing values. **1,081 exact duplicate rows** were removed (leaving 283,726) so copies cannot sit in both train and test.
- **Imbalance:** 492 frauds in the raw file = **0.173%**, about 1 in 580. After de-duplication, 473 frauds (0.167%).

## 3. Exploratory analysis

**Amounts.** Fraud is not simply larger: median fraud is about 10 vs 22 for legitimate transactions, yet the mean is higher (about 124 vs 88). About 36% of frauds are 1 or less versus about 11% of legitimate transactions, and the fraud rate in the 0-1 bucket (about 0.56%) is more than three times the average. Large purchases (250-1,000) also show a raised rate (about 0.3-0.4%) while the 10-50 range has the lowest (about 0.06%). The largest legitimate amount (about 25,700) far exceeds the largest fraud (about 2,100), so "big amount means fraud" is not a usable rule.

**Time of day** (hour relative to the first transaction, since the real start time is not given). Legitimate volume is lowest in hours 1-6 and flat and high from about hour 9 to 22. About 26% of frauds occur in hours 0-6 versus about 10% of legitimate transactions; the fraud rate peaks at hour 2 (about 1.45%) and hour 4 (about 1.04%), roughly 6-9 times the overall rate. Hourly fraud counts are small, so single hours are noisy.

**Features.** `V14`, `V10`, `V12`, `V4` and `V17` have the strongest association with fraud.

## 4. Why accuracy is misleading

A model that labels everything legitimate is about 99.8% accurate and catches zero frauds. Accuracy weights all errors equally, but a missed fraud and a false alarm have very different costs. The evaluation therefore uses precision, recall and F1 on the fraud class, ROC-AUC, and PR-AUC, and includes the "always legitimate" baseline in the results.

## 5. Method

- **Features:** `V1`-`V28` kept; `Amount` replaced by `log(1 + Amount)`; `Time` replaced by cyclical hour of day (`hour_sin`, `hour_cos`) because a raw elapsed-seconds counter does not generalise.
- **Split:** stratified 80/20 (226,980 train rows with 378 frauds; 56,746 test rows with 95 frauds). The test set is never resampled or tuned on.
- **Imbalance strategies compared:** none, `class_weight="balanced"` (`balanced_subsample` for the forest), and SMOTE applied to the scaled training data only. Undersampling was rejected because it discards over 99% of the legitimate data.
- **Models:** Logistic Regression (scaled features) and Random Forest (100 trees, `min_samples_leaf=2`).

## 6. Results (default threshold 0.5)

| Model | Accuracy | Precision | Recall | F1 | ROC-AUC | PR-AUC | Caught | Missed | False alarms |
|---|---|---|---|---|---|---|---|---|---|
| Baseline: always legitimate | 0.998 | 0 | 0 | 0 | 0.5 | - | 0 | 95 | 0 |
| LR, no handling | - | 0.846 | 0.579 | 0.688 | - | about 0.70 | - | - | - |
| LR + class_weight | 0.975 | 0.056 | 0.874 | 0.105 | 0.963 | about 0.68 | 83 | 12 | 1,405 |
| RF, no handling | - | 0.959 | 0.737 | 0.833 | - | 0.80 | 70 | 25 | 3 |
| RF + class_weight | - | 0.972 | 0.726 | 0.831 | 0.935 | 0.80 | 69 | 26 | 2 |

(Dashes mark values not reproduced in this report; they are produced by the notebook.) The SMOTE row is omitted because it could not be generated with the real library in the test environment.

**Key observations**

- **ROC-AUC flatters weak models:** the weighted Logistic Regression has the highest ROC-AUC (0.963) yet raises about 17 false alarms per fraud found. PR-AUC separates the models far better and ranks the forests first.
- **Imbalance handling shifts the precision/recall balance rather than improving the model.** For Logistic Regression, `class_weight` lifted recall from 0.58 to 0.87 but cut precision from 0.85 to 0.06.
- **Random Forest** is the strongest model type: precision about 0.96-0.97 with recall about 0.73-0.74, and only 2-3 false alarms among 56,746 transactions. With and without class weights it is almost identical.

## 7. Which metric matters most

Recall on the fraud class is the primary goal, since a missed fraud costs far more than a review. It cannot be maximised alone, so models are chosen on PR-AUC and the decision threshold is then set from costs and review capacity. For the best forest, a threshold of 0.1 instead of 0.5 catches 4 more frauds (21 missed instead of 25) at the price of 16 more false alarms (19 instead of 3) and lowers total cost by about 13% under the stated assumptions (average fraud amount as the cost of a miss and an **assumed** cost of 5 per false alarm). The sweep was done on the test set for illustration; a real system needs a separate validation set.

## 8. Feature analysis

Logistic Regression coefficients and Random Forest importance agree on a group of anonymised components: `V14`, `V10`, `V12`, `V4` and `V17`/`V1`. For Logistic Regression, higher `V4` raises the fraud odds while higher `V10`, `V12` and `V14` lower them. The forest puts about a quarter of its importance on `V14` and about 63% on its top five features. Engineered features add little beyond the V columns (`log_amount` ranks 17th of 31; hour features are near zero).

## 9. Temporal check

Training on the earliest 75% and testing on the latest 25% (93 frauds) gave PR-AUC 0.805 vs 0.800 for the random split, recall 0.699 vs 0.726 and precision 0.985 vs 0.972. The model therefore does not just exploit a random mix of the two days, though the check covers very little time.

## 10. Scalability: 1 million transactions per hour

One million per hour is about 278 transactions per second on average (assume a 10x peak of about 2,780). Measured on a single CPU core in the test environment:

| Model | Single-transaction latency | Per core | Cores for average load | Cores for 10x peak |
|---|---|---|---|---|
| Logistic Regression | about 1 ms | about 950 / s | 1 | 3 |
| Random Forest | about 6.6 ms | about 150 / s | 2 | about 19 |

Batch scoring ran at roughly 200,000 transactions per second (forest) and over 3 million (Logistic Regression). Inference is therefore not the bottleneck. The real constraints are the latency budget, real-time feature computation (the PCA transform and any per-card history), burst buffering, retraining and drift monitoring, and **alert volume**: at the fraud rate in this data there would be about 1,670 frauds per hour. The balanced Logistic Regression would raise roughly 26,000 alerts per hour (about 25,000 false), whereas the balanced forest raises about 1,250 alerts with about 35 false alarms and catches about 1,200 of the frauds. Suggested design: stateless replicas behind a load balancer, streaming ingestion, optional two-stage scoring (cheap model first, forest for suspicious cases), scheduled retraining with analyst feedback, and monitoring of score distribution, precision and recall.

## 11. Conclusion

Only 0.17% of transactions are fraud, so accuracy is meaningless. Fraud concentrates in very small amounts and quiet hours, but a handful of PCA components do most of the discrimination. **The Random Forest clearly beats Logistic Regression** on the metrics that matter (PR-AUC about 0.80 vs 0.68-0.70) and produces a manageable alert volume. Recommendation: deploy the Random Forest with a cost-based threshold behind a streaming scoring service, keeping Logistic Regression as a cheap first stage or fallback.

## 12. Limitations

- Only 95 test frauds and two days of data; close results are within noise and the temporal check is short.
- About a quarter of frauds are still missed at the default threshold.
- Single anonymised European sample from 2013; patterns, rates and costs elsewhere differ.
- SMOTE results were not verified with the real `imbalanced-learn` library.
- Scalability numbers are single-core measurements, and the false-alarm cost is an assumption.

## 13. Next steps

Gradient boosting (XGBoost / LightGBM), probability calibration, a validation set for threshold selection, cost-sensitive learning with real transaction values, and history-based features per card or merchant.
