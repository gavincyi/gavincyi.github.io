---
layout: post
title: Navigating Alpha Sizing: Insights from a Factor Risk Model and Cryptocurrency Experiment
subtitle: Alpha Sizing, Factor Risk Modeling, and the Quest for Portfolio Perfection
cover-img: https://github.com/gavincyi/gavincyi.github.io/assets/10500805/e731c30b-e663-47ae-a156-74425a5e46c1
tags: [alphageneration, portfolio, alphasizing, quantfinance]
comments: true
---

## Background

It was a perplexing puzzle to me when I first read Giuseppe A. Paleologo's book "Advanced Portfolio Management" in 2022. The book (in Chapter 6) introduces the concept of alpha sizing, a technique for allocating positions based on expected returns of alphas and risk. After an in-depth empirical analysis of various sizing rules, the author provides a concise insight:

> Proportional sizing method beats other common methods, e.g. mean-variance portfolio

In the realm of quantitative finance, I was taught that the Mean-Variance (MV) portfolio, also known as the Markowitz portfolio, is considered the standard, the golden rule, and the best approach for asset allocation, especially when considering target portfolio risk. Given this, I couldn't help but wonder: if mean-variance isn't the optimal approach for alpha generation, why do we invest so much effort and resources in its development? I've been patiently awaiting an opportunity to explore this question further.

## Risk-Based Sizing

The concept of risk-based sizing revolves around the idea of allocating assets to maximize the portfolio Sharpe Ratio rather than focusing solely on gross/net returns. With the right approach, portfolio managers can adjust portfolio exposure by target volatility or leverage. It also plays a crucial role in alpha generation, allowing for a comparison of each alpha's performance before their combination into a portfolio.
In the book, the following sizing approaches are suggested:
1. Proportional: This method involves allocating assets in the same direction and magnitude as the alpha, with adjustments for dollar neutrality and leverage.

2. Proportional Signed (1/N rule): In this approach, only the sign (positive or negative) of the alpha is considered, and the allocation is divided equally between the long and short sides.

3. Risk Parity: This method divides the alpha by the asset's volatility. The rationale behind this is to normalize the alpha with respect to volatility, resulting in dollar exposure per 1% of volatility.

4. Mean Variance: Alpha is divided by the asset's variance

5. Mean Variance (Shrink): Alpha divided by the shrunken asset variance `(1 - p) x variance + p x benchmark variance`. For instance, 75% shrink means 25% asset variance and 75% benchmark variance)

In fact, there are several more approaches in the market, e.g. the [Kelly](https://arxiv.org/abs/1710.00431#:~:text=Kelly's%20Criterion%20is%20well%20known,the%20financial%20and%20automation%20literature.) and the [Hierarchical Clustering](https://hudsonthames.org/an-introduction-to-the-hierarchical-risk-parity-algorithm/#:~:text=Hierarchical%20Risk%20Parity%20uses%20single,are%20closest%20to%20each%20other.) method. However, two of the main criteria considered when choosing the sizing candidates in the book are simplicity and practical relevance, meaning that the rules must be widely in use in both academia and industry. Intrigued by this, I rolled up my sleeves and embarked on an experiment in a new market - cryptocurrency.

## What did I do?

In the book, simulated alphas were generated from the lookahead one-month returns using universes in the Russell 3000 (with ADV over the previous month of $2M+). In the meantime, I opted to use historical data from the cryptocurrency market as my starting point for several reasons:

- Availability of Free Market Data: Cryptocurrency market data, with full history, is publicly available at no cost.

- Open Market with Good Liquidity: The cryptocurrency market is open and generally exhibits good liquidity, making it a suitable choice for experimentation.

- Extreme Volatility and Skewness: Cryptocurrencies are known for their extreme price volatility and skewed returns, which can present unique challenges and opportunities for alpha generation.

The cryptocurrency market shares some similarities with the equity market and even exhibits correlations with it (source). Both markets feature a large number of instruments, requiring the construction of universes over time. While the author constructed portfolios consisting of 50, 100, and 200 stocks from RAY, I selected up to 600 instruments from the top 50% trading volume in the crypto universe.
Furthermore, in my experiment, I not only generated a lookahead alpha (referred to as the "magical alpha") but also explored a few academic factors.


The cryptocurrency market shares some similarities with the equity market and even exhibits [correlations](https://www.institutionalinvestor.com/article/2bstqvrdta4idnp83c6ps/portfolio/crypto-is-becoming-more-correlated-to-stocks-and-its-your-fault) with it. Both markets feature a large number of instruments, requiring the construction of universes over time. While the author constructed portfolios consisting of 50, 100 and 200 stocks from RAY, I selected up to 600 instruments from the top 50% trading volume in the crypto universe.

Finally, I generated not only a lookahead alpha (named as "magical alpha") but also a few more academic factors in the experiment.

## Academic Factors

Academic factors represent specific drivers of asset returns and are identified as positive contributors to market returns in the long run. In the equity markets, for example, the momentum factor has been shown to generate positive returns across global indices and developed markets. The HML (High-minus-Low) factor compares companies with high book-to-market ratios to those with low ones, serving as a standard representation of the value factor. These factors often exhibit positive gross Sharpe ratios (excluding transaction costs) but slightly negative net Sharpe ratios (factoring in transaction costs).
Given the availability of market data (only daily prices and volumes), these signals were used in the experiment:
1. Magical Alpha: Calculated as the average of daily returns on the next month's returns.
2. Momentum: Computed as the average of 3-month, 6-month, 9-month, and 1-year annualized returns. To account for the extreme volatility of the market, annualized returns are clipped at +/-100%. The average annualized returns are then smoothed using a 14-day rolling window. 
3. Reversal: Reflects mean reversion in short-term movements. It is defined as the reverse normalized z-score of daily returns, with mean and standard deviation values derived from a period ranging from 3 months to 1 year. The resulting scores are further smoothed using a 14-day rolling window to reduce overall turnover.
4. Volatility: This factor suggests that assets with lower volatility tend to generate higher risk-adjusted returns than those with higher volatility. The signal is defined as the 6-month rolling volatility difference from the cross-sectional mean.
5. Liquidity: This factor suggests that less liquid assets may offer higher expected returns as compensation for their lower liquidity. The signal is defined as the inverse of volume (in USD) logarithmically transformed.

All signals are assumed to trade in a dollar-neutral portfolio (net position = 0) without any leverage (sum of absolute positions = 1). They are consistently normalized (recursively) on a cross-sectional level, with a winsorization of +/-3.0 applied.

Next, several performance metrics, such as the Sharpe ratio, are computed for these signals using the five sizing approaches mentioned earlier.

## You need a stronger stomach for MV portfolios

First, let's examine the Sharpe ratio. The table below presents results that align with the book's suggestions—generally, the proportional rule outperforms other approaches. The MV (Mean-Variance) portfolio turns a profitable liquidity alpha signal into a negative Sharpe ratio. The only exception for the MV portfolio is the momentum signal, and even its relative improvement is only modest.

 


Furthermore, the MV portfolio experiences the most significant maximum drawdown among all signals. Its maximum drawdown is 3-5 times higher than that of the proportional approaches. This implies that running an MV portfolio requires a stronger stomach, especially when the market takes a downturn.



Consistent with the book's findings, the 75% shrunken MV rule emerges as the best alternative to the original MV rule. Notably, it also significantly reduces the maximum drawdown.

Smart readers may have raised a critical question in an earlier section—why does the MV rule divide only the signal by variances? Why is the pairwise correlation between the instruments ignored?

## Factor risk model

The original experiment described in the book generated instruments with returns that are independent of each other, making it logical to consider only the instrument variances in the MV (Mean-Variance) rule. However, in practice, market instruments are often correlated, and for a large universe (as many as 600 instruments in our example), estimating empirical covariances becomes a challenging task.


To estimate an N x N covariance matrix, N x (N+1) / 2 relationships need to be estimated. Achieving a sufficiently converged covariance matrix in the financial market requires an enormous number of data samples, which is often impractical to obtain. Additionally, financial time series data often exhibits autocorrelation characteristics. This is where dimensionality reduction techniques come into play.

Creating a factor risk model from a smaller universe generates factors and their returns that have a significant influence on the market. The risk model then extends these factor exposures to a larger universe to estimate the covariances.

As mentioned in a previous post, the process begins by constructing an estimation universe to generate a factor risk model. This universe is selected based on the top 10% trading volume and involves the identification of the ten most significant technical factors using Principal Component Analysis (PCA). The number of instruments in the model and estimation universe is as follows:


Model Universe: Top 50% trading volume
Estimation Universe: Top 10% trading volume


Subsequently, the risk model is transformed using the returns of the model universe to derive the covariances within the model universe. Although I estimated both the volatilities and correlations of the model universe using 180-day returns, the framework supports estimating correlations with longer rolling windows and volatilities with shorter windows for a more responsive volatility estimation.

The estimated covariances are then used to derive the MV optimal portfolio using the formula:

$$
w^* = \Sigma w
$$ 

Here, \\\(w^*\\\) is the MV optimal portfolio, \\\(\Sigma\\\) is the covariance matrix, and \\\(w\\\) is the original signal.

## Corrected version of MV portfolio

Similar to previous rules, the corrected version of the MV (Mean-Variance) portfolio aims for dollar neutrality and unit leverage. While it is possible to derive a closed-form solution for the [dollar-neutral](https://quant.stackexchange.com/questions/59202/derivation-of-mean-variance-portfolio-weights-as-closed-form-analytical-solution) MV portfolio, for simplicity, we choose to standardize the MV portfolio by demeaning it and then adjusting its leverage at the end.

With covariances estimated from the factor risk model, the Sharpe ratio of the MV portfolio exhibits significant improvement, particularly for the shrunken MV portfolios (e.g., 75%). Notably, the MV portfolio based on the liquidity signal restores the Sharpe ratio to positive territory. In several signals, such as reversal, the 75% shrunken MV portfolio produces the best Sharpe ratio.



A similar improvement is observed in terms of maximum drawdown. The 75% MV portfolio experiences the lowest maximum drawdown in the momentum and reversal signals.



These results underscore the critical role of covariance estimation in the MV allocation rule. Even with the same variance estimation, accurate correlation estimation makes a substantial difference in alpha sizing.

## The bottom line

The conclusion drawn in the book states:

> It is not the volatility estimation error that matters, but rather the estimation error of the expected returns, combined with the fact there is variation in volatilities across the stock universe

In addition to the above conclusion, based on the empirical results from the cryptocurrency market, my conclusion is that volatility, correlation, and expected return estimation all carry significant importance, likely in equal measures. It appears that the 75% shrunken MV (Mean-Variance) portfolio strikes a balanced approach to address these three estimation errors.

For practitioners of the Kelly Criterion, this result aligns with conventional wisdom that fractional Kelly (such as quarter Kelly or even 1/8 Kelly) is the optimal capital growth strategy, as explained by [MacLean, L., E. O. Thorp, and W. T. Ziemba (2010)](https://www.researchgate.net/publication/227623956_Long-term_capital_growth_the_good_and_bad_properties_of_the_Kelly_and_fractional_Kelly_capital_growth_criteria).

IMeanwhile, it has become a standard [practice](https://ideas.repec.org/p/bge/wpaper/92.html) in the industry to shrink estimated covariances. If you are already employing covariance shrinkage techniques, you may find it beneficial to reduce the shrinkage parameter in the MV sizing rule or potentially eliminate the need for shrinkage altogether.

Finally, despite some academic literature suggesting the use of the 1/N (proportional-sized) rule in large universes and the MV rule in small universes, based on the results presented above, I believe that the 75% shrunken MV rule is the most suitable rule to apply across all universe sizes, especially when faced with uncertainty in choosing between the proportional and MV portfolio.
