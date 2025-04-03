## OLS on Nelson Siegel parametrization of US treasury yield curve

**Motivation** </br>

One challenge with modeling the yield curve is that it consists of many maturities (1m, 3m...30y). The high dimensionality makes it somewhat cumbersome to make predictions on it. One may decide to select a set of features and attempt to fit a model for each of the maturities, but some issues are:

- Separate predictions do not account for the structure of the yield curve. It is clear that the yields of different maturities are correlated in some way
- Predicted yields might not make sense (negative values, weird yield curve structures)

**Possible solution**

The Nelson Siegel model parametrizes the yield curve using 3 parameters: $\beta_1$, $\beta_2$, $\beta_3$. You may think of them as the level, slope, and "twist/curvature" of the yield curve. Now, instead of having to predict over 10 maturities, we only need to predict 3 values. The hyperparameter $\lambda$ controls the curvature/shape of the parametrized curve

$$y(\tau) = \beta_0 + \beta_1 \left( \frac{1 - e^{-\tau/\lambda}}{\tau/\lambda} \right) + \beta_2 \left( \frac{1 - e^{-\tau/\lambda}}{\tau/\lambda} - e^{-\tau/\lambda} \right)
$$

**Explanation of variables**:
- $y(\tau)$: The yield (interest rate) at a given maturity $ \tau $.
- $ \tau $: The time to maturity of the bond (years)
- $ \beta_0 $: The long-term level of interest rates (as $ \tau \to \infty $).
- $ \beta_1 $: The short-term component, influencing the curve's initial level.
- $ \beta_2 $: The medium-term component, controlling the hump or curvature.
- $ \lambda $: The decay factor that determines how quickly the effects of $ \beta_1 $ and $ \beta_2 $ fade over time.

**Intuition**:
- $ \beta_0 $ shifts the entire curve up or down.
- $ \beta_1 $ controls the slope of the curve at shorter maturities.
- $ \beta_2 $ determines the convexity or "hump" shape of the yield curve.
- $ \lambda $ controls how quickly the short-term and medium-term effects decay.


Of course, by reducing the dimensionality of the problem, we also lose some accuracy/information on the original yield curve. We will explore if such a tradeoff is worth it
