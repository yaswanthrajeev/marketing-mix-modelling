

 # Marketing Mix Modeling – Revenue Prediction

This repository contains a reproducible notebook (`maeketing.ipynb`) and supporting code for building and evaluating machine learning models to explain and predict weekly **revenue** based on paid media, direct response levers, price, promotions, and other features.
---

## 1. Data Preparation
- **Weekly granularity:** Data is structured at the weekly level across ~2 years.  
- **Trend & seasonality:** Differencing and lag features were introduced to capture temporal dynamics without leaking future information.  
- **Zero-spend periods:** Weeks with no paid media spend were preserved, as they carry informative variation for model learning.  
- **Scaling & transformations:**  
  - Continuous predictors (e.g., spend, price) scaled with `StandardScaler`.  
  - Mediator and revenue models scaled separately to maintain causal clarity.  

---

## 2. Modeling Approach
- **Two-stage setup:**
  1. **Mediator model** → Predicts intermediate digital engagement (e.g., Google spend proxy) from marketing inputs.  
  2. **Revenue model** → Uses both original levers and predicted mediator to estimate revenue.  

- **Models used:**  
  - **Regularized regression (Ridge, Lasso):** Provides interpretability and feature selection.  
  - **XGBoost:** Captures nonlinearities and interactions for stronger predictive accuracy.  

- **Hyperparameters:**  
  - Ridge/Lasso → tuned via `alpha` grid search with cross-validation.  
  - XGBoost → learning rate, depth, and regularization tuned on validation folds.  

- **Validation plan:**  
  - Rolling/blocked cross-validation to respect time ordering.  
  - Final out-of-sample evaluation on holdout weeks.  

---

## 3. Causal Framing
- **Mediator assumption:**  
  - Modeled explicitly by separating the prediction of `google` (mediator) from final revenue regression.  
  - Prevents leakage where future mediator values could bias results.  

- **Back-door paths:** Controlled by including relevant confounders (price, promotions, followers) to isolate media impact.  
- **Feature structure:** Mediator (`pred_google`) included only as predicted, not actual, during revenue estimation.  

---

## 4. Diagnostics
- **Out-of-sample performance:**  
  - XGBoost achieved the lowest RMSE and highest R² (~73% variance explained).  
  - Ridge/Lasso underperformed, with negative R² (worse than baseline).  

- **Residual analysis:**  
  - Residuals checked for randomness and approximate normality.  
  - Ridge/Lasso showed systematic bias; XGBoost residuals more stable.  

- **Stability checks:**  
  - Blocked time-series CV used to avoid lookahead bias.  
  - Sensitivity analysis conducted on price and promotion levers.  

- **Sensitivity plots:**  
  - Simulated revenue changes under ±10% price adjustments and promo toggling.  
  - Promotions generally shifted revenue upwards; higher prices showed nonlinear effects.  

---

## 5. Insights & Recommendations
- **Key drivers:**  
  - Promotions have a consistent, positive revenue effect.  
  - Average price shows an inverted-U response (too high suppresses demand).  
  - Paid media effectiveness partly mediated through digital engagement (`google`).  

- **Best model:**  
  - XGBoost recommended for forecasting and scenario planning (better accuracy).  
  - Ridge regression still useful for causal interpretability.  

- **Risks & caveats:**  
  - **Collinearity:** Spend channels are correlated; coefficients in Ridge/Lasso must be interpreted with caution.  
  - **Mediator validity:** Assumes predicted `google` captures the correct pathway; misspecification could bias effects.  
  - **Non-stationarity:** Effects may drift over time; recommend retraining with recent data.  

---

## How to Reproduce
1. Clone this repo and open `maeketing.ipynb`.  
2. Install dependencies:
   ```bash
   pip install -r requirements.txt
