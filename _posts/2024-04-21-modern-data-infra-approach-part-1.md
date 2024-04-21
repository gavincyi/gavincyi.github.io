---
layout: post
title: Modern Data Infrastructure Approach in Quant Finance - Part 1
subtitle: Business requirements, challenges and competitions
cover-img: https://github.com/gavincyi/gavincyi.github.io/assets/10500805/812f4afe-b79b-4e7f-b751-baccd9b73147
tags: [data, quantfinance]
comments: true
---

In 1997, Joshua Levine developed the first equity electronic matching engine, called Island, and the engine executed the first trade. Before that, the market relied on human traders, or "specialists", to match the buy and sell orders. Now It sounds natural that this human routine can be easily automated by machines. Yet, in hindsight, it was a significant breakthrough in the financial market that the order execution can be totally unbiased by the brokers and traders, but only three aspects - price, quantity and time.

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/cf51b746-3667-4b84-8752-88b03a8cc673)


Island Exchange offered a lower transaction fee while the data feed was accessible for free. In a year, it became the second largest electronic communication network (ECN), just after the Reuter owned ECN Instinet. Finally, it was acquired by Instinet in 2002, while Instinet was then acquired by NASDAQ. 

The infrastructure on Island Exchange wasn't the massive clusters of mainframes typical today. Instead, the matching engine was written in FoxPro and ran on Dell PCs. You can still access its source code on Joshua's website, where it's mentioned that an error message from the matching engine persisted in the NASDAQ system after the acquisitions.

Joshua's infrastructure design for the matching engine was remarkable, considering both data storage and processing for high order and execution traffic. This aroused my interest about how vital we can build a resilient system (meaning the application lasts for a longer than average lifespan), and how adapting modernised infrastructure keeps us at the right pace. In this post, I will outline the technology challenges in quantitative finance, especially on the investment / buy-side, and in the next post, I will delve into choosing the right technology for these challenges.


## 1. Data storage

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/48de14ab-4177-44ed-93d7-cad703613a57)

In the 1960s, Gordon Moore, co-founder of Intel, predicted that computing power would double every two years, based on his observation that the number of transistors on an integrated circuit doubled in a similar pattern. This prediction was later called "Moore's Law." Subsequently, in 2005, Mark Kryder derived a version of this law for storage capacity, suggesting that disk density doubles every 13 months, known as Kryder's Law.

Generally, the growth of storage capability, computing power and also transmission speed mirrors the surge of both dimensions and depth in data storage. Over time, images have more pixels, and videos are in better resolutions, and text sourced from various media and social platforms grows exponentially. It is natural to assume that the need for bigger storage capability in the financial industry aligns with the general trend.

<img width="718" alt="image" src="https://github.com/gavincyi/gavincyi.github.io/assets/10500805/5b581b21-eade-4c98-af3a-dd09451dbce3">

Interestingly, the core fundamental and economic data have not grown at the same pace. Governments still retain the cadence to publish economic data, like CPI, on a quarterly or monthly basis. Company filings are only required annually, while most companies announce the financial performance quarterly. Though the number of stocks are surging, the rate is well below the exponential rate as technology improves. Handling these limited sizes of data has now become a more trivial task for most analysts than a decade ago.

In the trading side, not only the trading volumes and executions grow, but also the higher cadence and deeper market book requires a consistent improvement in data infrastructure. Exchanges stream the market data with all the executions, market book snapshot, and sometimes the order changes to all market participants. I recalled, in my first engineering job in a boutique HFT firm, due to the limited computing power and flash storage, most strategies employed only the L2 order book, updated in every 500 ms, and executions. We often needed to estimate how many ring buffers and arrays we could store in the RAM, based on the number of trading instruments and expected execution arrival times, to avoid the out-of-memory in the application and system. Now, engineers may not need to worry too much about the storage constraint, but more about the access and write performance.

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/b40996ad-ba6e-4884-940a-3db1f2fa055c)

Finally, the unlimited storage capability we now seem to have (for example, look into our Gmail mailbox storage now) has led to a new stream in quantitative finance - alternative data. Conventional market data is typically time-series based and combined with financial reports and official publications, while the alternative data is composed of various types of unstructured data, retrieved from various API sources and web scraping methods. The examples of alternative data includes retail market transaction data (e-commence), social media posts and trends, weather forecasts, images and videos, and geographical data. These sets of data can give a great insight about the sales and revenue of individual companies, demand and supply of macro products, and country economies. The questions mainly surround the format and platform to store the gigantic size of unconventional data.

## 2. Data processing

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/f33d9b10-8bfa-4ab8-bd5c-2e2c0690f1cc)

In most cases, orchestrating data pull and pipelines is crucial in quant finance data infrastructure. Pulling data from various data sources is just the beginning of the data management journey. We then normalise the data into the data storage, clean it and send notifications if any hiccups occur. Exposing the data to downstream applications and users for data analysis is crucial, with data usage dictating storage format in data processing, and its location to live.

Previously, data orchestration mainly surrounded around the parallel / asychornized job runs in a single instance or limited computing power. Now, the discussion has been shifted to the resilience and scalability in multi-region clusters, and running jobs in various computing instances in CPU and GPU with careful considerations in FinOps.

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/97faa091-f297-4efd-a298-a3a1f53c3c4b)

In the meantime, data streaming is another related area to process the continuous data, while it is mostly used in the event driven system. It composes the critical part in the trading infrastructure in handling market data and events. Raw market data is streamed directly from the exchanges or data vendors through TCP/IP sockets, UDP multicast, or websockets. Institutional market data is normally streamed in low-latency communication protocols. For example, FAST for market data and FIX for order / market access. The specific protocol ensures the optimal transmission rate across the internet. 

Then the data is ingested with parsing and validation. Usually the ingestion speed is measured by 90th / 95th / 99th percentiles to ensure the consistency. High frequency trading firms / market makers invest significantly on both hardware and software to ensure that the data received and ingestion speed is on a par with competitors. The competition opens up the potential of FPGA and wave transmission technologies.This requires a deep understanding in computing architecture in both hardware and operational systems.

Finally, the raw market data is conveyed into derived data. For example, the trade prices are derived into volume and time average prices (VWAP and TWAP). Orders sitting in the book are processed into the order flow and imbalance analysis to detect the iceberg orders and institutional flows. The process power determines the synchronisation of primary and secondary level data.  

## 3. Data Validation and Privacy

![image](https://github.com/gavincyi/gavincyi.github.io/assets/10500805/60dccafa-5cef-4a05-8906-247601c76131)

Data validation is not only about avoiding stale data interrupting the current data infrastructure, but also handling outlier data in with statistical approaches in quantitative areas. It is often that the financial instrument prices are scaled before transmission. The examples are the stocks in London Stock Exchange (LSE) are listed in pence. The convention and scaling factor of the same instrument may vary across different data providers. One of the famous coding horrors was users thought to place the orders in pences but actually in pounds. So the data validation is vital before and after the raw data is conveyed to data pipelines. 

In the meantime, though the buy-side firms are mostly data consumers, data privacy is still key in protecting the business. Implementing role-based access controls (RBAC) and permissions restricts data access among different functionalities. For example, the trading entities are first masked in the trading activities before being processed by the research team for the market impact study. Modernized data protection schemes efficiently set up data infrastructure while ensuring privacy and compliance.

## Coming next

In conclusion, modern data infrastructure in quantitative finance faces evolving challenges in storage, processing, validation, and privacy. Addressing these challenges requires leveraging advanced technologies, such as cloud storage, stream processing, and data validation tools, while also ensuring robust privacy measures. I will walk through how the modern data infrastructure helps build holistic solutions on these problems.
