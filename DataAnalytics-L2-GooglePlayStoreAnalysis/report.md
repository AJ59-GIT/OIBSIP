# Report: Unveiling the Android App Market (Google Play Store Analysis)

## 1. Objective

Analyse the Google Play Store ecosystem: clean the raw data, explore categories, ratings, size and pricing, run sentiment analysis on user reviews, and derive practical insights for a developer planning to launch a new app.

## 2. Data

| Dataset | Raw size | After cleaning |
|---|---|---|
| Apps (`googleplaystore.csv`) | 10,841 rows, 13 columns | 9,659 unique apps |
| Reviews (`googleplaystore_user_reviews.csv`) | 64,295 rows, 5 columns | 29,692 reviews across 865 apps |

The apps data is a single snapshot with last-updated dates up to August 2018.

## 3. Data cleaning

| Problem | Fix |
|---|---|
| One malformed row (shifted columns: category `1.9`, rating `19`) | Dropped, as the values cannot be realigned reliably |
| `Installs` stored as text such as `10,000+` | Stripped `+` and `,` and converted to integers (each value is the minimum of its install bucket) |
| `Price` stored as text such as `$2.99` | Converted to a number |
| `Size` stored as `19M`, `201k` or `Varies with device` | Converted to megabytes; "Varies with device" became null (1,228 apps) |
| `Reviews`, `Last Updated` stored as text | Converted to integer and datetime |
| `Type` had a null and a stray `0` | Re-derived from `Price` (price above 0 means Paid) |
| 1,181 duplicate listings (483 exact copies, the rest older snapshots of the same app) | Kept one row per app, the one with the most reviews |
| Missing `Rating` (1,463 apps after cleaning, about 15%) | **Kept as null**: these apps have no ratings yet, and imputing would invent data |
| Reviews with no text (26,868, or 42%) and 7,735 repeated reviews | Removed |

## 4. Category analysis

The store has 33 categories and is concentrated: the **top 5 categories hold 46% of all apps**.

| Category | Apps | Share | Median installs |
|---|---|---|---|
| FAMILY | 1,876 | 19.4% | 50,000 |
| GAME | 945 | 9.8% | 1,000,000 |
| TOOLS | 829 | 8.6% | 50,000 |
| BUSINESS | 420 | 4.3% | 1,000 |
| MEDICAL | 395 | 4.1% | 1,000 |

Saturation was judged by comparing supply with demand (median installs):
- **Crowded with modest demand:** FAMILY and TOOLS (median 50,000), and especially BUSINESS and MEDICAL (median 1,000; only 16% and 5% of their apps reach 1M installs).
- **Crowded but strong demand:** GAME (57% of games exceed 1M installs).

## 5. Ratings

- 8,196 apps (85%) have a rating. **Mean 4.17, median 4.30**, and **77% of rated apps score 4.0 or above**.
- Category differences are small (0.46 stars from best to worst). Highest: EVENTS (4.44), ART_AND_DESIGN (4.36), EDUCATION (4.35). Lowest: TOOLS and MAPS_AND_NAVIGATION (4.04) and DATING (3.98).
- Users are generous, so a rating below 4 stands out.

## 6. Size and installs

Analysed on 8,417 apps with a known size and at least one install. Installs are bucketed, so a log scale and a rank correlation were used.

- **Pearson 0.30, Spearman 0.31**: a weak-to-moderate positive relationship.

| Size | Apps | Median installs |
|---|---|---|
| Under 5 MB | 2,312 | 5,000 |
| 5-10 MB | 1,607 | 50,000 |
| 10-20 MB | 1,533 | 50,000 |
| 20-50 MB | 2,090 | 100,000 |
| 50-100 MB | 875 | 1,000,000 |

Larger apps tend to be richer products (games, media tools) with bigger audiences. This is correlation, not proof that size causes installs, and small apps can still reach millions of installs.

## 7. Pricing

### 7.1 Paid vs free
- **Free:** 8,905 apps (92.2%). **Paid:** 754 apps (7.8%).
- Median installs: free 100,000 versus paid 1,000. Average rating: paid 4.26 versus free 4.17.

### 7.2 Price distribution of paid apps
Median price $2.99, 75% cost $4.99 or less, mean $14.06 (pulled up by outliers). 29 paid apps cost over $30 (novelty apps priced up to $399.99), which are excluded from charts for readability.

### 7.3 Revenue estimate
Estimated revenue is installs times price. Because installs are a bucket minimum, this is a **lower-bound, gross estimate** (before store commission and refunds) and covers upfront prices only, not ads or in-app purchases.

- Total: about **$291 million**, or **$204 million** excluding the 29 apps priced above $30 (novelty apps such as "I am rich" would otherwise distort category totals).
- Revenue is very concentrated: the **top 5 paid apps earn 48%** of the total, and Minecraft alone about a third.

| Category | Paid apps | Estimated revenue (excl. apps over $30) |
|---|---|---|
| FAMILY | 179 | $105.7M |
| GAME | 82 | $40.7M |
| PHOTOGRAPHY | 19 | $8.5M |
| PERSONALIZATION | 81 | $6.8M |
| LIFESTYLE | 12 | $6.2M |

## 8. Sentiment analysis (VADER)

Each review was scored with VADER's compound score (-1 to +1) and labelled **positive** (0.05 or above), **negative** (-0.05 or below) or **neutral**.

| Sentiment | Reviews | Share |
|---|---|---|
| Positive | 20,197 | 68.0% |
| Negative | 5,866 | 19.8% |
| Neutral | 3,629 | 12.2% |

**Validation:** VADER agrees with the dataset's own pre-computed labels on 76% of reviews (88% agreement for positive reviews, 59% for negative, 50% for neutral). VADER tends to read mixed reviews as positive, so the negative share is probably slightly understated.

## 9. Sentiment by category

Reviews were joined to apps (28,250 reviews, 816 apps). Only categories with at least 100 reviews from at least 10 different apps were compared (23 categories; 10 excluded for thin evidence).

| | Category (positive share) |
|---|---|
| **Most positive** | HEALTH_AND_FITNESS (78%), EDUCATION (78%), FOOD_AND_DRINK (74%) |
| **Least positive** | ENTERTAINMENT (59%), NEWS_AND_MAGAZINES (58%), SOCIAL (55%) |
| **Most negative** | SOCIAL (31%), ENTERTAINMENT (27%), NEWS_AND_MAGAZINES (25%) |

Because the reviews file covers only about 800 apps, each category rests on 15 to 84 apps, so these results are indicative rather than exact.

## 10. Interactive visualisations

Three plotly charts are included in the notebook: a **market map** (competition vs rating, bubble size = installs, colour = share of paid apps), an **interactive sentiment mix by category**, and a **paid-app price vs installs explorer**.

## 11. Conclusion: three insights for a new developer

1. **Choose a category by demand per app, not by size.** EDUCATION has just 106 apps yet a median of 1,000,000 installs, 58% of apps above 1M, a 4.35 average rating (third highest) and 78% positive reviews. HEALTH_AND_FITNESS also has very happy users (78% positive) with more moderate demand (median 100,000). GAME has strong demand but is crowded (945 apps) and needs a clear differentiator. BUSINESS and MEDICAL are crowded with weak typical demand.
2. **Plan a free app with in-app monetisation, not an upfront price.** Paid apps are 7.8% of the market, with a median of 1,000 installs against 100,000 for free apps, and paid revenue is dominated by a few hits. If charging, stay within the norm: median price $2.99, and 75% of paid apps cost $4.99 or less. Ad and in-app-purchase revenue are not in this dataset and need separate validation.
3. **Quality is the entry ticket, and weak sentiment marks underserved users.** With an average rating of 4.17 and 77% of apps at 4.0 or above, a launch below 4 stands out negatively. SOCIAL, ENTERTAINMENT and NEWS_AND_MAGAZINES have the most negative reviews, suggesting room for a clearly better experience, though SOCIAL has strong incumbents (42% of its apps exceed 1M installs). Do not chase a tiny download size; size and installs are only weakly related (correlation about 0.3).

## 12. Limitations

- Installs are buckets (lower bounds), so install-based numbers are approximate.
- The data is a single 2018 snapshot and may not reflect today's market.
- Revenue covers upfront prices only, and is dominated by a few apps.
- Sentiment covers about 800 of the 9,659 apps, and VADER is a lexicon-based method that can miss sarcasm and mixed opinions.
- All relationships reported are correlations, not proof of cause.
