# OIBSIP · Oasis Infobyte Data Analytics Internship Projects (Level 2)

A collection of five end-to-end data science projects completed for the **Oasis Infobyte Internship Program (OIBSIP)**. Each project is delivered as a Jupyter notebook with its own requirements file, README and written report, and covers the full workflow: data cleaning, exploration, modelling or analysis, evaluation, visualisation and clear conclusions.

## Projects

| Task | Project | Techniques |
|---|---|---|
| **1** | [House Price Prediction](DataAnalytics-L2-HousePricePrediction/) | EDA, feature engineering, one-hot encoding, Linear Regression, Ridge / Lasso, residual diagnostics, coefficient analysis |
| **2** | [Wine Quality Prediction](DataAnalytics-L2-WineQualityPrediction/) | Classification, class imbalance, target binning, Random Forest / SGD / SVC, feature importance |
| **3** | [Credit Card Fraud Detection](DataAnalytics-L2-FraudDetection/) | Extreme class imbalance, `class_weight` and SMOTE, Logistic Regression vs Random Forest, PR-AUC, threshold and cost analysis, scalability |
| **4** | [Google Play Store Analysis](DataAnalytics-L2-GooglePlayStoreAnalysis/) | Data cleaning, category / rating / pricing analysis, VADER sentiment analysis, interactive plotly charts |
| **5** | [Autocomplete & Autocorrect Analytics](DataAnalytics-L2-AutocompleteAutocorrect/) | NLP preprocessing, n-gram language models, edit-distance spelling correction, precision/recall evaluation |

## Repository layout

```
OIBSIP/
├── README.md                                   <- this file
├── LICENSE                                     <- MIT
├── .gitignore
├── DataAnalytics-L2-HousePricePrediction/
├── DataAnalytics-L2-WineQualityPrediction/
├── DataAnalytics-L2-FraudDetection/
├── DataAnalytics-L2-GooglePlayStoreAnalysis/
└── DataAnalytics-L2-AutocompleteAutocorrect/
    ├── Images/                                 <- images used by the project documentation
    ├── autocomplete_autocorrect.ipynb          <- the notebook
    ├── The_Project_Gutenberg.csv               <- dataset (plain text of a book despite the .csv name)
    ├── req_nlp.txt                             <- the project's libraries
    ├── README_autocomplete_autocorrect.md      <- how to run and configure the project
    └── REPORT_autocomplete_autocorrect.md      <- methodology, results and discussion
```

Every project folder follows the same layout: an `Images/` folder, the notebook, the dataset, a requirements file, a `README_*.md` and a `REPORT_*.md`. Each notebook looks for its dataset in its own folder (or in a `data/` sub-folder), so keep the data file beside the notebook.

## Getting started

Developed and tested with Python 3.12.

```bash
# 1. clone the repository
git clone https://github.com/AJ59-GIT/OIBSIP.git
cd OIBSIP

# 2. (optional) create a virtual environment
python -m venv .venv
source .venv/bin/activate            # Windows: .venv\Scripts\activate

# 3. install the dependencies for one project...
pip install -r DataAnalytics-L2-WineQualityPrediction/req.txt

# ...or for all five at once
pip install \
  -r DataAnalytics-L2-HousePricePrediction/req.txt \
  -r DataAnalytics-L2-WineQualityPrediction/req.txt \
  -r DataAnalytics-L2-FraudDetection/req_fraud.txt \
  -r DataAnalytics-L2-GooglePlayStoreAnalysis/req.txt \
  -r DataAnalytics-L2-AutocompleteAutocorrect/req_nlp.txt

# 4. open a notebook from inside its own folder and choose Kernel -> Restart & Run All
cd DataAnalytics-L2-WineQualityPrediction
jupyter notebook wine_quality_prediction.ipynb
```

The Task 1 and Task 4 notebooks are saved **with outputs** so results are visible on GitHub. The Task 2, 3 and 5 notebooks are saved **without outputs**; running them produces every table, chart and saved model.

> **Troubleshooting:** a `ModuleNotFoundError` usually means the notebook is running in a different Python environment from the one you installed into. Run `%pip install -r <requirements-file>` inside a notebook cell, restart the kernel, and run again.

## Project summaries

### Task 1 · House Price Prediction
Predicts house sale prices from area, quality, neighbourhood, rooms and age on 1,460 houses with 80 features. Missing values are handled (most blanks mean "feature absent"), categorical features are one-hot encoded, and rare categories are pooled to avoid unstable predictions. A Linear Regression baseline reaches a test R² of 0.871 (RMSE 31,405, about 17% of the average price); **Lasso** is slightly better (R² 0.875) and keeps only 98 of 194 features. Living area, overall quality, garage capacity, neighbourhood and house age drive price, and all models under-predict the most expensive houses.

### Task 2 · Wine Quality Prediction
Predicts whether a red wine is "good" (quality 7 or above) from 11 chemical measurements. After removing 240 duplicate rows and grouping the quality scores into a binary target, three tuned classifiers are compared. **Random Forest** is recommended (accuracy 0.875, macro-F1 0.728). Runtime is about one minute.

### Task 3 · Credit Card Fraud Detection
Detects fraud in a dataset where only about 0.17% of transactions are fraudulent. Shows why accuracy is misleading, compares imbalance-handling strategies, and evaluates with precision, recall, F1, ROC-AUC and PR-AUC. **Random Forest** is recommended (precision about 0.96-0.97, recall about 0.73-0.74, PR-AUC about 0.80), and the notebook also analyses decision thresholds, a temporal hold-out and throughput at 1 million transactions per hour. The Random Forest cells take a few minutes.

### Task 4 · Google Play Store Analysis
Cleans messy app data (9,659 unique apps) and 29,692 user reviews, then analyses categories, ratings, size, installs and pricing, and classifies review sentiment with VADER (68% positive, 20% negative, 12% neutral). The conclusion gives three insights for a developer launching a new app: choose a category by demand per app (Education has high demand and little competition), plan a free app with in-app monetisation (only 7.8% of apps are paid), and treat quality as the entry ticket (average rating 4.17). Includes three interactive plotly charts.

### Task 5 · Autocomplete & Autocorrect Analytics
Builds unigram, bigram and trigram autocomplete models and three edit-distance spelling correctors (Levenshtein, Damerau-Levenshtein, Norvig-style) on the text of *Alice's Adventures in Wonderland*. A trigram model with back-off gives the best next-word accuracy (Hit@3 28.0%); Damerau-Levenshtein ranking with candidate generation corrects best and fastest (F1 about 0.85, about 2 ms per word). Runtime is about one minute.

## Datasets

| Project | Dataset | Notes |
|---|---|---|
| House Price Prediction | `train.csv`: 1,460 houses, 80 features plus `SalePrice` | Place beside the notebook if missing |
| Wine Quality | UCI Wine Quality (red), 1,599 rows, semicolon-separated | Included |
| Fraud Detection | European cardholder transactions (Time, V1-V28, Amount, Class), 284,807 rows | **Not included** (about 150 MB), add it yourself |
| Google Play Store | `googleplaystore.csv` (10,841 rows) and `googleplaystore_user_reviews.csv` (64,295 rows) | Place beside the notebook if missing |
| Autocomplete / Autocorrect | Project Gutenberg eBook #11 (*Alice's Adventures in Wonderland*), about 26,600 words | Included |

## Known limitations

- **SMOTE (fraud detection):** `imbalanced-learn` could not be installed in the environment where the notebooks were tested, so that step ran with a minimal stand-in using the same k-nearest-neighbour idea. The code uses the standard `imblearn` API; run the notebook to get the real SMOTE results. The reports avoid quoting SMOTE numbers for this reason.
- **NLTK and `pyspellchecker` (NLP):** neither was installed during testing. The notebook falls back to a built-in copy of the NLTK English stop-word list, and the optional `pyspellchecker` comparison is skipped automatically if the library is missing. Those two paths are untested.
- **Play Store data:** a single 2018 snapshot. Install counts are bucketed lower bounds, so install and revenue figures are estimates, and sentiment covers only about 800 of the 9,659 apps.
- **House prices:** all models under-predict houses above roughly 400k; modelling `log(price)` would be a natural next step.
- **Small samples:** the test sets are small (37 good wines; 95 frauds; about 5,300 autocomplete queries), so small differences between close models are within noise.
- **Reproducibility:** all notebooks use seed 42, but exact numbers can differ slightly across library versions.
- **Timings** quoted in the reports were measured on a single CPU core and will differ on your hardware.

## Where to find more

Each project folder contains its own `README_*.md` (how to run and configure it) and `REPORT_*.md` (methodology, results and discussion).

## License

Released under the [MIT License](LICENSE).

## Author

[AJ59-GIT](https://github.com/AJ59-GIT) · Oasis Infobyte Internship Program (OIBSIP), Data Analytics track
