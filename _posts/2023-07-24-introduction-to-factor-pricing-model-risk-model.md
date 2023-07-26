---
layout: post
title: Can we build a factor pricing model at the enterprise level? (2)
subtitle: Unleashing the Power of Factor Pricing Models with Statistical Models
tags: [factormodel, riskmodel, portfoliomanagement, finance, quantdevelopment]
comments: true
---


Since my previous [post](https://gavincyi.github.io/2022-10-14-can-we-build-a-factor-pricing-model-at-the-enterprise-level-1/) about factor models, I have embarked on creating a new package named [factor-pricing-model-risk-model](https://github.com/factorpricingmodel/factor-pricing-model-risk-model). The aim of this open-source project is to generate factor models from specified market data. To better understand the technical aspects and explore the possibilities, I delved into several existing libraries. Now, I'd like to share an update on my progress through this checkpoint post.

## pyportfolioopt

One of the libraries I came across is [pyportfolioopt](https://github.com/robertmartin8/PyPortfolioOpt), an open-source creation by  [Robert Martin](https://github.com/robertmartin8) designed for portfolio optimization toolkits. The project stands out with its support for various essential portfolio optimization models and techniques, including the Markowitz optimizer and Black-Litterman allocation. While it also covers risk model generation from raw market data, it is currently limited to computing covariances from raw prices. I find the library to be among the top 3 open source libraries that closely resemble how institutional investors and asset managers manage their portfolio. However, there seems to be a missing piece in the risk model generation specifically related to factor models, and that's where my new library comes into play.

## qlib

Another library is Microsoft's [qlib](https://github.com/microsoft/qlib/tree/main), an AI-oriented quantitative investment library offering a wide range of models and examples, from point in time databases to reinforcement learning. The library sounds powerful, but, similar to my perspective on Microsoft's products, its features are fragmented and lack good documentation. I found a few modules in the source code about covariance estimators, including [structured covariance estimator](https://github.com/microsoft/qlib/blob/main/qlib/model/riskmodel/structured.py) and [covariance shrinkage](https://github.com/microsoft/qlib/blob/main/qlib/model/riskmodel/shrink.py), but I could not find their descriptions on the official documentation site. While I regard qlib as a great reference, I believe my new library can provide a better structure and a more coherent solution for factor model generation and risk analysis.

## Factor pricing model 

My primary focus is to create a library that centers on factor models themselves. Unlike existing libraries and scripts that mainly address risk model construction, my goal is to cater specifically to factor models. With this library, users will be able to:

- Choose from various models, e.g. statistical models, to build their risk models
- Compare and benchmark the forecasting accuracy of different risk models
- Easily scale up the size of their risk models without encountering significant challenges

In the following sections, I will walk you through how I approach these problems and outline the solutions.

## Data

Building and benchmarking risk models heavily relies on high-quality market data. Unfortunately, obtaining such historical data, especially for equities, can be challenging and often requires costly subscriptions to data providers. To demonstrate the models and allow everyone to try them out in Colab notebooks, I have chosen cryptocurrency data as a starting point. It doesn't mean that I haven't tested the library with equity data – I indeed have and compared it with a risk model from a prominent vendor. However, using cryptocurrency data provides a more accessible playground for anyone to start experimenting without incurring significant costs.

## Statistical model

In developing the `factor-pricing-model-risk-model` library, I decided to focus on statistical models as the starting point. Unlike fundamental models, which can be challenging to define factors beyond obvious ones like momentum, statistical models, such as Principal Component Analysis (PCA), offer a more straightforward approach and also serve as a natural benchmark for other statistical models.


## Forecast accuracy

To evaluate the forecast accuracy of different models, we need appropriate metrics. For this purpose, I have incorporated two metrics: 

1. **Bias test**: This metric assesses forecasting accuracy of volatility on a time series of observed portfolio returns.

2. **Value at risk (VaR)**: VaR is a risk metric that represents the maximum possible loss for a given holding horizon and specific level of confidence.

These metrics provide valuable insights into how well the models can forecast in general.

## Agility 

Scalability is a crucial consideration, especially as we increase the size of the universe. With the covariance matrix growing to N x N for an N-sized universe, the runtime complexity can become a concern. I have been exploring ways to manage runtime efficiently, even when scaling up the universe's size.

Additionally, researchers often face the challenge of comparing various models with benchmarks once they have defined the estimation and model universe. This comparison may involve their existing models retrieved from vendors. Finding an efficient and reliable way to perform such comparisons is an essential aspect of the library's design.

## Universe

The concept of universes plays a pivotal role in the factor-pricing-model-risk-model library. Two types of universes are defined:

1. **Estimation Universe**: This comprises the top market cap instruments in the market. It is used to construct factor returns.

2. **Model Universe**: This consists of instruments with sufficient trading liquidity, such as the top 50% based on average daily volume. The model universe derives its factor exposures and covariances from the computed factor returns.

Both universes include instruments actively trading in the past 6 months and exclude stablecoins and forked-like pairs (e.g., BTC vs. wrapped BTC). Similar to constructing equity universes, the process involves excluding ETFs and instruments listed on multiple exchanges.

By carefully defining and managing these universes, the library ensures robust and accurate factor model construction and risk analysis.

The following graph illustrates the number of valid instruments in the estimation universe

![estimation universe](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/d304dd1c-17b2-4d48-a4cb-b5bdcb408951)

and that in the model universe.

![model universe](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/6ecb1445-d611-48c8-a76d-d8c36c763dd0)

Comparing the number of valid instruments in the two universes in cryptocurrency class, we can see the ratio between the estimation and model universe is around 10:1.

## Factor Model

With the defined universes and market data in place, we can now proceed to build the factor model and compute the covariances for the model universe.

To begin, we create a DataFrame to represent the estimation universe, capturing its validity within the date index and instrument information across columns.

|index|algorand|apecoin|avalanche-2|axie-infinity|binance-usd|
|---|---|---|---|---|---|
|2022-09-11 00:00:00|false|false|true|true|false|
|2022-09-12 00:00:00|false|false|true|true|false|
|2022-09-13 00:00:00|true|true|true|true|false|
|2022-09-14 00:00:00|false|true|true|true|false|
|2022-09-15 00:00:00|false|true|true|true|false|

Next, we generate a risk model using statistical factors obtained through PCA, which serves as our benchmark. The parameter `n_components` is set to 0.9, indicating that we select the number of components needed to explain more than 90% of the total variance. This model operates on a rolling basis of 180 days, using daily returns from the estimation universe. To enhance stability, the daily returns are windorized by +/-20%.

```
from fpm_risk_model.rolling_factor_risk_model import RollingFactorRiskModel
from fpm_risk_model.statistical import PCA

model = PCA(n_components=0.9)
rolling_risk_model = RollingFactorRiskModel(
    window=180,
    model=model,
    show_progress=True,
)
rolling_risk_model.fit(
    X=est_universe_returns,
    validity=est_universe_validity,
)
```

Similarly, we define the model universe in the same format as the estimation universe. With a larger set of returns in the model universe, we can then transform the risk model from the estimation universe to the model universe, enabling us to effectively analyse and evaluate the covariances for the entire model universe.

```
rolling_risk_model.transform(
    X=model_universe_returns,
    validity=model_universe_validity,
)
```

## Covariance estimation

The rolling risk model allows us to derive covariances from any subsets of the model universe.

For instance, we can retrieve the unadjusted covariance on a specified date (2023-05-31)

```
rolling_risk_model.get("2023-05-31").cov()
```

or the unadjusted correlation between two specified pairs (BTC v.s. ETH)

```
rolling_risk_model.get("2023-04-30").corr().loc[["BTC_Bitcoin", "ETH_Ethereum"], ["BTC_Bitcoin", "ETH_Ethereum"]]
```

|              |   BTC_Bitcoin |   ETH_Ethereum |
|:-------------|--------------:|---------------:|
| **BTC_Bitcoin**  |      1        |       0.892195 |
| **ETH_Ethereum** |      0.892195 |       1        |


However, for better covariance estimation, we can employ two techniques. Firstly, we can adjust the diagonal entries using alternative volatility estimation methods. For example, replacing the existing volatility estimation, derived from longer-term observations, with GARCH volatility estimation can better capture the volatility structure.

The second technique involves "coverage shrinkage". Covariance estimation typically requires a large sample of data to converge towards the true population values. However, due to the curse of dimensionality, smaller observation sets often lead to noisy sample covariances and even noisier inverses. This phenomenon is especially common in financial data with extreme and potentially noisy returns. 

Coverage shrinkage is a powerful method to address this issue. By introducing a parameter $\delta$, we can suppress the influence of off-diagonal elements in the covariance (or correlation) matrix, achieving a better signal-to-noise ratio. This technique yields more reliable results, particularly in scenarios with smaller datasets or when dealing with extreme and noisy financial data.

The covariance shrinkage parameter $\delta$ can be specified during covariance estimator creation using the following formula:

$$
\hat{Q} = (1 - \delta) * Q + \delta * \mu * I
$$

Here, $\hat{Q}$ is the adjusted covariance matrix, $Q$ is the sample covariance matrix, $\mu$ is the mean variance, and $I$ is the identity matrix.

creation. Additionally, volatility adjustments can be made using the cov method. For instance, to specify a constant shrinkage with $\delta$ equal to 0.2 and adjust the volatility using a GARCH estimation named garch_est, you can follow the specified procedure.

```
from fpm_risk_model import RollingCovarianceEstimator

estimator = RollingCovarianceEstimator(
  rolling_risk_model,
  shrinkage_method="constant",
  delta=0.2
)
estimator.cov(volatility=garch_est)
```

## Result

The results presented below are derived from the analysis conducted on the crypto universe dataset. The factor model is initially trained using statistical models with an estimation universe and subsequently transformed with the model universe. The covariances, both unadjusted and adjusted, are compared using accuracy estimators to evaluate their performance.

The following types of covariances are compared

1. Unadjusted covariances derived by PCA factor model
2. Unadjusted covariances derived by APCA factor model
3. Covariances adjusted with EWMA volatilities derived by PCA factor model
4. Covariances adjusted with EWMA volatilities and constant shrinkage (0.2) derived by PCA factor model

The observed results are as follows:

![XKL8YfM-9AOmI-HoT9_1k84Q_DjmtOfzNNNA7AEer3k_l4sPSnQqpMp9puqzvO54c_L-ZHDds--Lwl28qycyoe-rC09joGFWAMP2U3qUbv0I21mJ6coFmZZ_P71Q](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/ba4b003e-d673-4a77-8f93-36384de7bc6e)

![Dv3NpGwyL-nrjsCH7msCZIzqh36MNTtOMPiJ_GNaWeHJK0iVXjx_SAc24-LHES3it4OhjIbw1ZJ7S6EIZ85eeK0Onhe0ylQK2oj5SEu7L98DP7LSm5OsVxsBgDsM](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/46bd2696-5448-443b-99f9-239e6889f032)


1. PCA is the stable benchmark for comparison
2. APCA does not work better even theoretically it should give better short term variance predictions
3. PCA adjusted with short term EWMA volatility gives a slightly better result than the benchmark
4. Constant shrinkage delta does not give a sufficiently stable result. More advanced shrinkage technique is needed to get optimal shrinkage delta.


## What's next

In conclusion, as the factor-pricing-model-risk-model library is still in its beta stage, the results obtained so far serve as an encouraging glimpse of its potential. With each iteration, we are committed to refining and expanding the library's capabilities, ensuring it becomes even more powerful and versatile. The comparisons between covariance estimation techniques have provided valuable insights that will guide our ongoing development efforts.

While the library is a work in progress, it is open to contributions and welcomes ideas from the community. As more features are added and improvements made, the goal is to provide users with a comprehensive and powerful tool for factor pricing models across various asset classes. Feedback, suggestions, and contributions from the community will play a crucial role in shaping the library's future and ensuring it becomes a valuable resource for researchers, institutional investors, and asset managers in their pursuit of sophisticated risk analysis and data-driven decision-making.

All kinds of contributions and ideas from the community are welcomed, as we continue to refine the library. Please do not hesitate to drop me PMs or emails if you want to understand more about the library.
