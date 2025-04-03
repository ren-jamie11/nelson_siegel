## OLS on Nelson Siegel parametrization of US treasury yield curve

**Motivation: How would you predict a 10-dimensional object?** </br>

INSERT PICTURE OF YIELD CURVE

One challenge with modeling the yield curve is that it consists of many maturities (1m, 3m...30y). The high dimensionality makes it somewhat cumbersome to make predictions. One may decide to select a set of features and attempt to fit a model for each of the maturities, but some issues are:

- Separate predictions do not account for the structure of the yield curve. It is clear that the yields of different maturities are correlated in some way
- Predicted yields might not make sense (negative values, weird yield curve structures)

**Possible solution: Dimension reduction**

The Nelson Siegel model parametrizes the yield curve using 3 parameters: $\beta_1$, $\beta_2$, $\beta_3$. You may think of them as the level, slope, and "twist/curvature" of the yield curve. Now, instead of having to predict over 10 maturities, we only need to predict 3 values. The hyperparameter $\lambda$ controls the curvature/shape of the parametrized curve

$$y(\tau) = \beta_0 + \beta_1 \left( \frac{1 - e^{-\tau/\lambda}}{\tau/\lambda} \right) + \beta_2 \left( \frac{1 - e^{-\tau/\lambda}}{\tau/\lambda} - e^{-\tau/\lambda} \right)
$$

**Explanation of variables**:

- $y(\tau)$: The yield (interest rate) at a given maturity $\tau$.
- $\tau$: The time to maturity of the bond (years)
- $\beta_0$: The long-term level of interest rates (as $\tau \to \infty$).
- $\beta_1$: The short-term component, influencing the curve's initial level.
- $\beta_2$: The medium-term component, controlling the hump or curvature.
- $\lambda$: The decay factor that determines how quickly the effects of $\beta_1$ and $\beta_2$ fade over time.

Of course, by reducing the dimensionality of the problem, we also lose some accuracy/information on the original yield curve. We will explore if such a tradeoff is worth it.

### Problem setup

#### 1. Load data

1. Gather yield curve data
2. Back out Nelson Siegel parameters by finding $\beta_1$, $\beta_2$, $\beta_3$ that minimizes least squares (set $\lambda = 1.6729$)*
3. Gather macroeconomic predictors (I chose PCE, consumer sentiment, unemployment rate)
4. Resample to 1 months and then take the difference (to stationarize data)
5. Shift responses forward 1 month (Use last month's change in macro variables to predict next month's change in response variables)

*Data sources*

| Data                     | Source                                                               |
|--------------------------|----------------------------------------------------------------------|
| Nelson Siegel Method    | https://www.diva-portal.org/smash/get/diva2:1306445/FULLTEXT01.pdf |
| US Treasury Yield Curve | https://home.treasury.gov/interest-rates-data-csv-archive  |
| PCE (Inflation)         | https://fred.stlouisfed.org/series/PCEPI                    |
| Consumer Sentiment      | https://fred.stlouisfed.org/series/UMCSENT                  |
| Unemployment Rate      | https://fred.stlouisfed.org/series/UNRATE                    |


Fortunately, the data was already quite clean. All I needed to do was minor date format changes and resampling (for this project at least!)

#### 2. OLS model: Train and predict

**Training period**: 2010-01 to 2022-12 </br> 
**Test period**: 2023-01 to 2024-12

**Method 1:** Fit and predict for each of the 10 yield curve maturities directly </br>
**Method 2:** Fit and predict for 3 Nelson Siegel parameters then transform back to yield curve

Note: We are regressing the *monthly change* in our macro predictors on the *next-monthly change* in the responses

#### 3. Compare methods

Compare the sum of the absolute error of the predicted vs actual yield curve values for each maturity (then take mean over each of the 24 months in test period). If we let $T = \{\tau_1, \tau_2...\tau_n \}$ be the set of maturities in the yield curve in years ($1/12, 3/12...30$), then we are comparing

$$L_{t} = \sum_{\tau \in T} | y_{\tau} - \hat{y}_{\tau}|$$ 

and then 

$$L = \frac{1}{n}\sum_{i=1}^{n}L_{t_i}$$ 

for both methods where $n$ is the number of test samples. We can perform a 2-sample t-test to see if there is a statistical significant difference in the 2 methods. We also will qualitatively examine the visual plot of the true yield curve vs predicted yield curve for both methods

### Discussion and Results

**Questions**
- Was OLS linear regression a good choice here? (check residual vs fitted values)
- Was there a statistically significant relationship between chosen macro predictors and responses? (t stats)
- Did the Nelson siegel OLS perform better than direct OLS on each of the 10 maturities? (statistical test + visual plots)
- If so, why? If not, why?

#### Residuals vs fitted values

<img src="assets/img/resid_vs_fitted_yc.png" alt="Residuals vs fitted values for each yield curve" width="500">

#### OLS t stats (training)

#### Compare 2 methods

### Conclusion






