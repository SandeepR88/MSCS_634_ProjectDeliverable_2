# MSCS 634 – Project Deliverable 2
## Regression Modeling and Performance Evaluation

### Project overview
This deliverable continues Deliverable 1 using the **UCI Online Retail** dataset. Deliverable 1 identified 541,909 transaction-line records and eight source attributes, documented data cleaning, retained returns/cancellations for separate analysis, and created a positive-sales subset for sales analysis.

This phase builds invoice-level regression models to estimate total invoice revenue in GBP from transaction-basket, time, and country features.

### Dataset
- **Source:** Chen (2015), UCI Machine Learning Repository
- **Dataset:** Online Retail
- **Original records:** 541,909 transaction lines
- **Original attributes:** InvoiceNo, StockCode, Description, Quantity, InvoiceDate, UnitPrice, CustomerID, Country
- **Historical period:** December 1, 2010 to December 9, 2011
- **File:** `data/Online Retail.xlsx`

The notebook follows Deliverable 1's cleaning decisions: exact duplicate lines and unusable date/numeric rows are removed; cancellation invoices and non-positive quantities/prices are excluded from the positive-sales modeling subset. The original workbook is retained in `data/`.

### Target and feature engineering
The modeling unit is one invoice. The target, `InvoiceRevenue`, is the sum of `Quantity × UnitPrice` across valid sales lines belonging to that invoice.

Engineered predictors include:
- Basket size: number of invoice lines, unique products, total quantity.
- Price profile: mean, median, maximum, and standard deviation of unit price.
- Basket ratios: quantity per line and products per line.
- Log-transformed size and price features.
- Time features: hour, weekday, month, day of month, weekend indicator.
- Country, one-hot encoded.

Invoice number and customer ID are excluded because they are identifiers, not generalizable behavioral features. Missing unit-price standard deviations for single-line invoices are set to zero.

**Important scope note:** Quantity and price summaries are observed basket attributes that are directly related to invoice revenue. This is an invoice-value estimation task once basket details are available, not a forecast of customer demand before shopping or checkout.

### Modeling and evaluation
Two models are compared:
1. **Ridge Regression** – standardized numeric features, one-hot encoded country, `alpha=100`.
2. **Random Forest Regression** – nonlinear ensemble with 60 trees, maximum depth 16, minimum leaf size 3, and one-hot encoded country.

A reproducible 80/20 train/test split (random state 42) is used. Three-fold shuffled cross-validation is applied to the training data. Reported metrics include R-squared, MSE, RMSE, and MAE. The notebook also generates:
- Holdout RMSE and MAE comparison
- Actual-versus-predicted plots
- Residual plots
- Random Forest feature-importance chart

### Evaluation results
The following values were obtained by running the notebook workflow against the included UCI workbook. Minor differences may occur with different library versions.

| Model | CV R² mean | CV RMSE mean (£) | CV MAE mean (£) | Test R² | Test MSE (£²) | Test RMSE (£) | Test MAE (£) |
|---|---:|---:|---:|---:|---:|---:|---:|
| Ridge Regression | 0.6366 | 1,068.13 | 201.53 | 0.7907 | 154,087.89 | 392.54 | 172.88 |
| Random Forest | 0.7647 | 934.33 | 107.94 | 0.9190 | 59,634.19 | 244.20 | 79.89 |

The notebook prints full-precision metrics; README values are rounded to two or four decimal places.

**Interpretation:** Random Forest performed better on both the mean cross-validation RMSE and the held-out test metrics (higher R² and lower RMSE/MAE). Ridge had a lower cross-validation RMSE standard deviation, indicating less fold-to-fold variation in this run. Random Forest is the stronger overall candidate based on its lower average cross-validation RMSE and substantially stronger test performance, but its larger cross-validation RMSE standard deviation warrants caution.

### Insights and observations
- Invoice revenue is right-skewed; a small number of large invoices can substantially affect RMSE and R².
- MAE and RMSE tell different stories: RMSE penalizes large misses more strongly, while MAE describes typical absolute error more directly.
- Random Forest's stronger test performance suggests nonlinear relationships and feature interactions may matter.
- Cross-validation results are less favorable for Random Forest than its single holdout result, so the holdout score should not be treated as guaranteed future performance.
- A time-ordered test split is a useful next step because the data span a historical period rather than a stationary current market.

### Challenges and how they were addressed
- **Transaction-line versus invoice-level prediction:** Revenue is naturally an invoice-level business outcome, so transaction lines were aggregated by invoice.
- **Returns and cancellations:** These records can distort ordinary sales values, so the modeling subset follows Deliverable 1 and uses positive, non-cancelled sales only.
- **Skewed invoice values and outliers:** Plausible high-value invoices were retained rather than deleted automatically. Both RMSE and MAE are reported to show sensitivity to large errors.
- **Categorical country field:** Country is one-hot encoded within each model pipeline.
- **Potential leakage and overstated claims:** InvoiceNo and CustomerID are excluded, and target-derived invoice revenue per line is not used as a predictor. Basket quantity and price features are observed basket attributes, so the model is not presented as pre-purchase forecasting.


### References
Chen, D. (2015). *Online Retail* [Dataset]. UCI Machine Learning Repository. https://doi.org/10.24432/C5BW33

Chen, D., Sain, S. L., & Guo, K. (2012). Data mining for the online retail industry: A case study of RFM model-based customer segmentation using data mining. *Journal of Database Marketing & Customer Strategy Management, 19*, 197–208. https://doi.org/10.1057/dbm.2012.17
