---
layout: post
title: Can we build a factor pricing model at the enterprise level?
subtitle: How far is it now to get rid of GIL in the future Python?
cover-img: https://user-images.githubusercontent.com/10500805/195588121-e4ab2d01-548e-49ea-b946-c236d2818947.png
tags: [factormodel, riskmodel, portfoliomanagement, finance]
comments: true
---

Multifactor models describe the return on an asset in terms of the risk of the asset with respect to a set of factors. The formulation can be represented as

$$ r_f = Bf + u $$

where \\\(r\\\) is the vector of asset returns, \\\(f\\\) is the factor returns, and \\\(u\\\) is the idiosyncratic returns. \\\(B\\\) is the factor loading matrix with a number of rows equal to the number of assets.

## Background

The multifactor model is critical in portfolio management and it can be used to

- examine the instrument and portfolio exposure to the factors

- construct the instrument-wise covariance matrix on a portfolio consisting of a basket / large number of instruments

- break down the performance attribution in portfolio

In the market, there are already credible vendors providing factor models or risk models based on multi-factor models. For example, MSCI (previously Barra) is famous in [providing](https://www.msci.com/our-solutions/analytics/multi-asset-class-factor-models) multi-asset class and equity factor models.

## Factors

Basically there are different types of factors, and the factors vary among asset classes. The types are composed of, but not limited to, active / style, regional / industry and macroeconomics.

In equity, the major active / style factors are

|Name|Description|
|---|---|
|Size|Small-cap stocks exhibit greater returns than large-cap stocks|
|Momentum|Return persistence in short or medium term horizon|
|Value|Stock prices relative to their fundamental values, e.g. price-to-book and price-to-earnings|
|Volatility|Low volatility stocks exhibit greater returns than high volatility stocks|
|Quality|Return associated with companies resilience, e.g. low debt and stable incomes|

And the vendors may have distinctive definitions on the factors. For example, Axioma defines the momentum factor simply as the cumulative returns on the recent months, while Factset as "normalized slope of an ordinary least squares regression of last 14 closing prices at four-weekly intervals"


## Can we build an enterprise level factor model?

First question is who is "we". "We" can be researchers, boutique asset management funds / family offices, or even finance enthusiasts.

It comes to the next question - why do we want to build a factor model by ourselves?

Subscribing the data for enterprise level of factor models and risk models can be expensive, and even very expensive if multiple asset classes of them are subscribed. Cost can be one factor.

Also, selecting the right package is not straightforward. For example, if the universe is America blue-chip stocks and the reallocation period is monthly, it is a no-brainer to subscribe to an America medium-horizon factor model for any providers in the market. But what if your portfolio is a daily reallocated portfolio of Chile and Argentine stocks, is there any available package in the market? How good can the vendor provide factor models of an emerging market basket? Customisation can be another factor.

Finally, as mentioned in the above section, vendors may have slightly different definitions on the factors, and more factors are found out in the academic world. Researchers need a simple tool to test on the new factors, find out the factor loading and idiosyncratic returns from customised factors, or even generate factor models in exotic asset classes, e.g. crypto. Standardisation is also a critical factor.

But before concluding whether it is feasible, let me outline the models, the estimation approaches, and not least the problems we need to tackle.

## Modelling and estimation

In general, there are three models to derive the factor returns and idiosyncratic returns

1. Fundamental model
2. Time series model
3. Statistical model

Historically, the fundamental model is regarded as the most reliable method in performance and interpretability, while the statistical model is catching up rapidly. As a whole, a complete factor model is a combination of active / style factors, derived by fundamental model, and statistical factors, derived by statistical model.

The time series model is mostly used to derive the factor exposures given the factor returns, especially the macroeconomic factors, for example, changes in bond yields, or unanticipated inflation.

### Fundamental model

The fundamental model derives the factor returns \\\(f\\\) and the residual returns \\\(u\\\), given the asset returns and exposure matrix.

The cross sectional regression can be computed with the following steps

1. Before the market opens, retrieve an updated list of features of each instrument from data and process the data, e.g. normalise each loading.

2. The features are allocated into the loading matrix \\\(B\\\) in a shape of (m, n), where m is the number of instruments and n is the number of features.

3. Compute the factor returns \\\(f\\\) and idiosyncratic returns \\\(u\\\) by regression between the instrument returns and factor loadings.

4. WIth the latest factor and idiosyncratic returns, derive the factor covariance matrix and the idiosyncratic volatilities with the existing values.

### Time series model

The time series model derives the factor loading \\\(B\\\) and idiosyncratic returns \\\(u\\\), given the time series of asset returns and factor returns.

Similar to cross section regression, the time series regression can be computed with the following steps

1. Retrieve the time series data of the features and the instrument returns.

2. The returns of the features are computed and allocated into the factor returns \\\(f\\\) in a shape of (m, n), where m is the number of rolling days and n is the number of features

3. For each instrument, compute the factor loading \\\(B\\\) and idiosyncratic returns \\\(u\\\) by regression between the time series of instrument returns and factor returns.

4. Combine the factor loading \\\(B\\\) and idiosyncratic returns \\\(u\\\) for all instruments

### Statistical model

The most prevailing statistical model is PCA (Principal components analysis). The method determines the specified number of factors (denoted as m) by eigendecomposition of the observed asset returns covariance matrix. Given the observed asset returns covariance matrix \\\(Q\\\),

$$ Q = UDU^T $$

select the largest \\\(m\\\) eigenvalues and eigenvectors from \\\(D\\\) and \\\(u\\\)

The factor loading matrix \\\(B\\\) is taken to be \\\(U_m D^{0.5}_{m}\\\) where \\\(U_m\\\) and \\\(D_m\\\) are the matrix chosen from the largest \\\(m\\\) eigenvalues, and the factor return matrix \\\(f\\\) can be derived by regression.

The procedure to retrieve the factor loading and idiosyncratic return is

1. Compute the observed asset returns covariance matrix \\\(Q\\\) from a small basket of instruments. For example, only the blue-chip stocks in equity.

2. Get the factor loading matrix \\\(B\\\) from PCA and the specified number of features.

3. Derive the factor returns \\\(f\\\) by running the time-series regression on the instrument returns in the basket and their factor loading matrix \\\(B\\\)

4. Use the factor returns \\\(f\\\) to compute the factor loadings and idiosyncratic returns for the whole portfolio.

## Risk model

With the time series of factor returns and idiosyncratic returns, the covariance matrix can be derived by

$$ Q = B \Sigma B^T + \Delta ^2 $$

where the factor return covariance matrix \\\(\Sigma\\\) can be computed by the time series of factor returns, while the idiosyncratic volatilities \\\(\Delta\\\) by the time series of idiosyncratic returns.

To compute the factor return covariance matrix \\\(\Sigma\\\) from a time series of factor returns \\\(FF^T\\\), where \\\(f\\\) contains rows of factor returns

$$
\begin{bmatrix}
f_{0,0} & f_{0,-1} & ... & f_{1,-T} \\
f_{1,0} & f_{1,-1} & ... & f_{2,-T} \\
f_{n,0} & f_{n,-1} & ... & f_{n,-T} \\
\end{bmatrix}
$$

The estimation of the factor covariance matrix can be produced by weighting the factor return matrix on recent history using an exponential weighting scheme.

For example, with a specific halflife \\\(\lambda\\\), the weight is \\\(w_{i} = 2^{\frac{-i}{\lambda}}\\\), while \\\(i\\\) is between \\\(0\\\) and \\\(-T\\\), and the factor returns are adjusted as \\\(FW^{\frac{1}{2}}\\\)

## Challenges

The above parts have outlined the high level implementation to model the relationship between the instrument returns and the factors, and the procedures to derive the result step by step. In the meantime, the challenges will be explained below, so that it is comprehensive for everyone to understand why the vendored factor models are still prevailing in the status quo.


### Modelling

Basically there are two types of regressions in the three models - cross sectional and time series regressions. OLS (Ordinary least-squares) regression is an intuitive approach; however its assumption of homoscedasticity is unlikely to be satisfied, i.e. the variances of the idiosyncratic returns among the instruments are doubtfully in synchronisation.

Moreover, outlines in regression can disrupt the quality of the results. The objective of OLS regression is to minimise the total squared error, and minimising the error in fat tailed returns can significantly impact the beta and shrink the R-square values.

Finally, tracing the modelling error is not trivial. Part of the problem is how to quantify the modelling error, regarding the large number of instruments and factors. The remaining is how to distinguish between the estimation error, for example due to sampling error and outliers, and the specification error, for example incorrect forecast horizon.

### Computation

Updating the factor model online is manageable, but running the factor model in full history can be an exhausting process. Currently lots of statistical libraries in Python are assumed to run only on CPUs. Running the pricing model in full history may take at least several days of run time.

### Data

Generating active / style factors in a fundamental model requires a vast amount of raw data. For example, in equity, to generate the value factor, it requires the stock fundamental data, like price-to-book. These depend on a reliable data source and require passing the raw data into pipelines to convert them into time series of factor loadings.

## Summary

Though the challenges are never five-finger exercises, I would like to dive into the procedures and implementations to tackle them. While I am still in faith in vendored factor models and their derived risk models, I reckon that building a factor model from scratch is manageable, and the result may be close as the enterprise level ones. I believe the following is archivable and deliverable

- Standardise the major modelling procedures, e.g. regression

- Run the computation in different instance types, i.e. CPU, GPU or even TPU

- Measure the risk model forecasting accuracy

and will outline the details in the future.

## Reference

[Axioma, Axioma RobustTM Risk Model Version 4 Handbook, October 2017](https://www.ibm.com/support/pages/sites/default/files/inline-files/$FILE/Axioma_ModelUpdate-WW4-Jan2018-2.pdf)

[Factset, Multi-Asset Class (MAC) II Risk Model](https://advantage.factset.com/multi-asset-class-risk-model-ii-white-paper)

[Factset, FactSet Equity Model, Global Mid-Horizon (Daily), 2022](https://advantage.factset.com/factset-equity-model-daily)

[Giuseppe A. Paleologo, Advanced Portfolio Management, Wiley, 2021](https://www.amazon.co.uk/Advanced-Portfolio-Management-Fundamental-Investors/dp/1119789796)
