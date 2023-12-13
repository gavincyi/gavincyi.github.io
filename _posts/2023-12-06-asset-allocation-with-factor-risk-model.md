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


