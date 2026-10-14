# Task 4 · Unveiling the Android App Market (Google Play Store Analysis)

A complete data analysis of the Google Play Store: cleaning messy real-world data, exploring categories, ratings, size and pricing, running sentiment analysis on user reviews, and finishing with three data-driven insights for a developer planning to launch a new app.

## Key findings

1. **Choose a category by demand per app, not by size.** Education has only 106 apps but a median of 1,000,000 installs and 78% positive reviews, while Business (420 apps) and Medical (395 apps) have a median of only 1,000 installs.
2. **Plan a free app with in-app monetisation.** Only 7.8% of apps are paid, with a median of 1,000 installs versus 100,000 for free apps. The top 5 paid apps earn 48% of estimated paid revenue.
3. **Quality is the entry ticket.** The average rating is 4.17 and 77% of apps score 4.0 or higher. Social, Entertainment and News have the most negative reviews, which points to underserved users.

See [`report.md`](report.md) for the full write-up.

## Files

| File | Description |
|---|---|
| `playstore_analysis.ipynb` | The complete, executed Jupyter notebook (all outputs and charts, including 3 interactive plotly charts) |
| `report.md` | Written project report: method, results, insights, limitations |
| `req.txt` | Python libraries required to run the notebook |

## Datasets (not included here; place them next to the notebook)

| File | Description |
|---|---|
| `googleplaystore.csv` | App metadata: category, rating, reviews, size, installs, type, price and more (10,841 rows) |
| `googleplaystore_user_reviews.csv` | Individual user reviews (64,295 rows) |

The notebook locates the files by their ending, so names with a prefix such as `1791..._googleplaystore.csv` also work.

## How to run

1. Put the notebook and both CSV files in the same folder.
2. Install the libraries:
   ```bash
   pip install -r req.txt
   ```
3. Open the notebook and choose **Run All**:
   ```bash
   jupyter notebook playstore_analysis.ipynb
   ```

> **Troubleshooting:** if `ModuleNotFoundError: No module named 'vaderSentiment'` appears, the notebook is running in a different Python environment from the one you installed into. Run `%pip install vaderSentiment plotly` in a notebook cell, restart the kernel, and run again. The import must be the full form: `from vaderSentiment.vaderSentiment import SentimentIntensityAnalyzer`.

## Tech stack

Python, pandas, NumPy, matplotlib, seaborn, plotly, VADER sentiment (`vaderSentiment`), Jupyter Notebook.

## Feature checklist coverage

- [x] Load the apps and reviews datasets separately
- [x] Data cleaning: fix data types (Installs, Price, Size, Reviews, dates), handle nulls, remove duplicates
- [x] Category analysis: bar chart of apps per category and a saturation assessment
- [x] Ratings analysis: rating distribution and average rating by category
- [x] Size vs installs: scatter plot and correlation
- [x] Pricing analysis: paid vs free, price distribution, revenue estimate by category
- [x] Sentiment analysis of reviews (positive / negative / neutral) with VADER
- [x] Sentiment by category
- [x] Interactive plotly visualisations (market map, sentiment mix, paid-app price explorer)
- [x] Conclusion: 3 data-driven insights for a new developer

## Cleaning summary

| Issue | Fix |
|---|---|
| 1 malformed row with shifted columns | Dropped |
| `Installs`, `Price`, `Size`, `Reviews` stored as text | Converted to numbers (`Size` to MB) |
| 1,181 duplicate app listings | Kept one row per app (the one with the most reviews), leaving 9,659 unique apps |
| Missing ratings (about 15%) | Left as null rather than inventing values |
| 26,868 reviews with no text; 7,735 repeated reviews | Removed, leaving 29,692 reviews |

## Project structure

```
task4_playstore/
├── playstore_analysis.ipynb
├── googleplaystore.csv                  (you provide)
├── googleplaystore_user_reviews.csv     (you provide)
├── req.txt
├── report.md
└── README.md
```
