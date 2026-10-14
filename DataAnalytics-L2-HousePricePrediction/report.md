# Report: Predicting House Prices with Linear Regression

## 1. Objective

Build and evaluate a linear regression model that predicts house sale prices (`SalePrice`) from property features (area, location, rooms, age and quality), and interpret which features drive price.

## 2. Dataset

`train.csv` contains **1,460 houses** described by **80 features** (numeric and categorical) plus the target `SalePrice`.

| Statistic | SalePrice |
|---|---|
| Mean | 180,921 |
| Median | 163,000 |
| Standard deviation | 79,442 |
| Minimum / Maximum | 34,900 / 755,000 |
| Skewness | 1.88 |

The distribution is **right-skewed**: a minority of expensive houses pulls the tail out. This matters later, because a linear model struggles at the extreme high end.

## 3. Data preparation

### 3.1 Missing values
19 columns contain nulls. Most are not errors: a blank in `PoolQC`, `Alley`, `Fence`, `FireplaceQu`, `Garage*`, `Bsmt*` or `MasVnrType` means the house does not have that feature. These were filled with a `None` category. Genuine gaps were handled as follows:

| Column | Treatment |
|---|---|
| `Electrical` (1 gap) | Filled with the most common value |
| `MasVnrArea` | Filled with 0 (no veneer) |
| `LotFrontage` (259 gaps) | Median imputation, fitted on the training set only inside the model pipeline |

### 3.2 Feature engineering and clean-up
- Created `HouseAge` (sale year minus year built), `RemodAge` (sale year minus remodel year) and `TotalBath` (full baths plus half of half-baths, basement baths included).
- Dropped `Id`, the original year and bath columns that were replaced, and `GarageYrBlt`.
- Dropped columns that are exact sums of other columns (`1stFlrSF`, `2ndFlrSF`, `LowQualFinSF`, `BsmtFinSF1`, `BsmtFinSF2`, `BsmtUnfSF`) because perfect multicollinearity makes coefficients unstable.
- Dropped `PoolQC`, `MiscFeature` and `Alley` (over 90% "feature absent").

### 3.3 Encoding and scaling
- **Numeric (25 features):** median imputation, then standardisation. This makes coefficients comparable and is required for Ridge and Lasso.
- **Categorical (40 features):** one-hot encoding with `drop="first"`. Categories shared by fewer than 10 training houses are pooled into one "infrequent" group.
- Result: **194 model inputs**.

> **Why pool rare categories?** A first run without pooling gave a test R² of about 0.68, because a single house with a unique roof material (`Membran`, one house in the whole dataset) was predicted at a negative price. Pooling rare categories fixed this and lifted test R² to 0.87.

## 4. Exploratory findings

Top numeric correlations with `SalePrice`:

| Feature | Correlation |
|---|---|
| OverallQual | 0.791 |
| GrLivArea | 0.709 |
| GarageCars | 0.640 |
| TotalBath | 0.632 |
| GarageArea | 0.623 |
| TotalBsmtSF | 0.614 |
| TotRmsAbvGrd | 0.534 |
| HouseAge | -0.523 |
| RemodAge | -0.509 |
| MasVnrArea | 0.473 |

Build quality and living area are the strongest predictors; older and less recently renovated houses sell for less. `GarageCars` and `GarageArea` are correlated with each other, which is one source of multicollinearity.

## 5. Modelling

The data was split **80/20** (1,168 training rows, 292 test rows, `random_state=42`). A scikit-learn `Pipeline` combines preprocessing and a `LinearRegression` model, so every transformation is learned from the training data only.

## 6. Results

| Model | Set | MSE | RMSE | R² |
|---|---|---|---|---|
| Linear Regression | Train | 615,237,072 | 24,804 | 0.8969 |
| Linear Regression | **Test** | 986,285,680 | **31,405** | **0.8714** |
| Ridge | Train | 686,646,408 | 26,204 | 0.8849 |
| Ridge | Test | 969,983,965 | 31,145 | 0.8735 |
| Lasso | Train | 706,194,522 | 26,574 | 0.8816 |
| Lasso | Test | 959,274,463 | 30,972 | 0.8749 |

- The baseline explains **87.1%** of the variance on unseen houses. Its RMSE of 31,405 is about 17% of the average price.
- Training R² (0.897) is higher than test R² (0.871), a small gap that indicates mild overfitting from the large number of correlated inputs.
- **Ridge** (alpha 16.04) and **Lasso** (alpha 142.51), both chosen by 5-fold cross-validation on the training data, improve test RMSE slightly. Lasso performs best and keeps only **98 of 194** features (96 set to exactly zero), giving a simpler model for slightly better accuracy.

## 7. Diagnostics

- **Actual vs predicted:** points follow the diagonal closely for typical houses (roughly 100k to 300k). Houses above about 400k are clearly **under-predicted**.
- **Residuals:** centred near zero (mean 2,028), mostly randomly scattered. The spread widens at higher predicted prices (mild heteroscedasticity) and there are a few large positive outliers, mostly expensive houses.

## 8. Coefficient analysis

Numeric features were standardised, so a numeric coefficient is the price change for a one standard deviation increase, holding other features constant.

| Strongest numeric effects | Coefficient |
|---|---|
| GrLivArea | +26,366 |
| OverallQual | +10,900 |
| GarageCars | +9,802 |
| TotalBath | +5,537 |
| HouseAge | -4,296 |

Categorical coefficients are premiums or discounts relative to the dropped reference level of each column:

- **Largest positive:** `Neighborhood_StoneBr` (+43,530), `Neighborhood_NoRidge` (+38,768), `Neighborhood_NridgHt` (+29,352) and `SaleCondition_Partial` (new builds, +22,362). Location is the biggest non-size driver.
- **Largest negative:** `BldgType_Twnhs` (-31,373), `BsmtQual_Gd` and `BsmtQual_TA` (about -29,000), `KitchenQual_TA` (-26,888) and `Neighborhood_Edwards`, `Mitchel` and `OldTown` (about -21,000 to -24,000). The quality-level negatives mean "this level sells for less than the reference (excellent) level", not that good kitchens lower prices.
- `TotalBsmtSF` has a small negative coefficient (-3,057) only because it overlaps with `GrLivArea` and other size features; it should not be read as "bigger basements lower the price".

Pooled "infrequent" groups are excluded from this ranking because each is a mix of unrelated rare categories.

## 9. Limitations and next steps

- All three models under-predict the most expensive houses. Modelling `log(SalePrice)` or adding interaction terms are natural improvements.
- Multicollinearity between related features makes individual coefficients less reliable than the overall predictions.
- Results come from a single 80/20 split; cross-validation of the final model would give a more stable estimate.

## 10. Conclusion

A linear regression model built on cleaned, encoded and standardised features explains **87.1%** of house price variance on unseen data, with a typical error of about 31,400. The main drivers of price are living area, overall quality, garage capacity, neighbourhood and house age. Regularisation (Ridge, and especially Lasso) gives a small accuracy gain, and Lasso also halves the number of features used.
