## OLS on Nelson Siegel parametrization of US treasury yield curve

**Motivation: How would you predict a 10-dimensional object?** </br>

<img src="https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/example_yc.png" alt="Alt text" width="500">

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

### Residuals vs fitted values

![alt text](https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/resid_vs_fitted_yc_train.png)
![alt text](https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/resid_vs_fitted_yc.png)

![alt text](https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/resid_vs_fitted_betas_train.png)
![alt text](https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/resid_vs_fitted_betas_test.png)

#### Interpretation:

In general, the residual vs fitted values on training do not exhibit any particularly strong patterns, suggesting that linear regression is an appropriate rough approximation of the relationship between features and responses. Additionally, the patterns exhibited in training and test are not too different, suggesting the model did not overfit. One note:

Lower maturities seem to have more outliers than higher ones. This makes sense, since the variance of short-term rate changes is higher than long-term rate changes (since long-term rates are closely related to the average of short-term rates over that period)

### OLS training stats summary

![alt text](https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/ols_summary.png)

*Correlation matrix* </br>
<img src="https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/corrs.png" alt="Alt text" width="300">

#### Interpretation

Looking at the p-values, we can conclude that there is some positive relationship between PCE (inflation)/consumer sentiment and yields. Conceptually, an increase in inflation/consumer sentiment in the previous month generally leads to rises in yields in the next month. This aligns with economic intuition, as the Fed will tend to raise rates to dampen inflation/overheated economy. Meanwhile, we do not find a statistically significant relationship between yields and unemployment (holding other variables constant). </br>

Because the correlation between unemployment and the other 2 features are somewhat high, we could have chosen to remove it to have a stronger interpretation of linear regression results. But since our primary goal is prediction, we will keep it for now 

Meanwhile, macro predictors seem to have meaningful explanatory power for change in level, but not change in slope and twist. This is likely because our predictors are very short-term and backward looking (1m), while the slope/twist of the yield curve requires economic forecasts 5-10 years into the future. Therefore, we would need to collect more relevant features to accurately model level/twist (such as Fed announcements, recession estimates etc). 


### Compare 2 methods

![alt text](https://github.com/ren-jamie11/nelson_siegel/blob/main/assets/prediction_vs_true.png)

**Statistical results**

| Metric                          | Value   |
|---------------------------------|---------|
| Mean total abs error for direct OLS  | 2.197   |
| Mean total abs error for Nelson Siegel OLS | 2.420   |
| 2-sample t-stat                 | -0.5620 |
| P-value                         | 0.5769  |


#### Observations

- Nelson siegel (orange) curves are generally flatter than the direct OLS predictions (less variance in shape)
- It is not visually obvious which one has more accurate predictions

Both seem to be particularly bad at predicting changes in long-term yields. This makes sense, as there is much more information that goes into long-term rates than simply looking at the 1-month change in inflation/consumer sentiment. Long-term rates consist of all available information on economic policy forecasts for the next 5-10 years. Just because inflation went up 0.6% last month doesn't mean we expect that to continue happening over the next 10 years.

### Conclusion: 

Overall, there was a *lack of statistical evidence to suggest there is a meaningful difference in predictive accuracy between directly performing OLS on the yield curve itself vs. its Nelson Siegel parameters*. This conclusion is made under the context of using 1-month change in inflation, consumer sentiment, and unemployment to predict next-month yield curve changes. </br>

The *main weakness of the OLS on Nelson Siegel method is that the features did not have meaningful predictive power on the shape of the yield curve* - only its value. Without features that can predict the shape/structure of the yield curve, the model loses its power. In that case, we might as well perform OLS directly on the yield curve itself, since at least we get a more precise prediction of the level of each maturity (rather than a single $\beta_1$ feature representing the average level of the entire curve) 

Therefore, to *capture the benefits of Nelson Siegel parametrization, we would need to find features that capture the change in steepness/curvature of the yield curve*, which necessarily requires features that capture long-term economic expectations (rather than 1m backward-looking changes). 

*TLDR: Model choice cannot help you if you do not have good features!*






