# Task 1 · Predicting House Prices with Linear Regression

Build and evaluate a linear regression model that predicts house sale prices from features such as area, neighbourhood, rooms and age, covering the full workflow from data cleaning to model interpretation.

## Results at a glance

| Model | Test RMSE | Test R² |
|---|---|---|
| Linear Regression | 31,405 | 0.871 |
| Ridge | 31,145 | 0.874 |
| Lasso | 30,972 | 0.875 |

The baseline model explains 87.1% of the variance in house prices on unseen data. Its typical error (RMSE) is about 17% of the average price (180,921). See [`report.md`](report.md) for the full write-up.

## Files

| File | Description |
|---|---|
| `house_price_prediction.ipynb` | The complete, executed Jupyter notebook (all outputs and plots included) |
| `report.md` | Written project report: method, results, interpretation, limitations |
| `req.txt` | Python libraries required to run the notebook |

## Dataset (not included; place it next to the notebook)

`train.csv`: 1,460 houses described by 80 features, plus the target `SalePrice`.

## How to run

1. Put `house_price_prediction.ipynb` and `train.csv` in the same folder.
2. Install the libraries:
   ```bash
   pip install -r req.txt
   ```
3. Open the notebook and choose **Run All**:
   ```bash
   jupyter notebook house_price_prediction.ipynb
   ```

> **Tip:** if you see a `ModuleNotFoundError`, the notebook is probably running in a different Python environment from the one you installed into. Run `%pip install -r req.txt` in a notebook cell, restart the kernel, and run again.

## Tech stack

Python, pandas, NumPy, scikit-learn, matplotlib, seaborn, Jupyter Notebook.

## Feature checklist coverage

- [x] Load dataset and EDA: null check, descriptive statistics, distribution of the target (price)
- [x] Feature selection discussion in a markdown cell
- [x] Handle missing values and one-hot encode categorical features
- [x] Correlation heatmap of the features most correlated with price
- [x] 80/20 train/test split
- [x] Linear Regression model with scikit-learn
- [x] Evaluation with MSE, RMSE and R²
- [x] Scatter plot of actual vs predicted prices
- [x] Residual plots
- [x] Coefficient analysis (highest positive and negative impact)
- [x] Bonus: comparison with Ridge and Lasso (alpha chosen by 5-fold cross-validation)

## Method summary

1. **Cleaning:** blanks in categorical columns such as `Garage*`, `Bsmt*`, `Fence` mean the feature is absent, so they are filled with a `None` category. `LotFrontage` is imputed with the training-set median inside the model pipeline (no test-set leakage).
2. **Features:** engineered `HouseAge`, `RemodAge` and `TotalBath`; dropped `Id`, columns that are exact sums of other columns (to avoid multicollinearity) and columns that are over 90% empty.
3. **Encoding:** numeric features are median-imputed and standardised; categorical features are one-hot encoded (`drop="first"`), with categories seen in fewer than 10 houses pooled into one group. This gives 194 model inputs.
4. **Modelling:** a single scikit-learn `Pipeline`, so all preprocessing is learned from the training data only.

## Project structure

```
task1_house_price/
├── house_price_prediction.ipynb
├── train.csv                 (you provide)
├── req.txt
├── report.md
└── README.md
```
