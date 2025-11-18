---
title: "Ehtereum Network Analysis — Part 03: Basic Statistics"
date: 2025-11-14
categories:
  - Data Mining
tags:
  - Reflection
  - Cryptocurrency
  - Ethereum
  - Blockchain
  - Data pareparation
  - Data Science
layout: single
author_profile: true
read_time: true
comments: false
share: true
---

## Data Cleaning 
After downloading the raw CSV files from Google BigQuery, I performed some data cleaning and preprocessing steps, code in the 
src/data/preprocessing.py file. The main tasks included:
 - nomalizing timestamp formats
 - handling missing values
 - converting data types for easier analysis
 - removing duplicates and some irrelevant columns

after this, the cleaned data files are stored in the data/cleaned/ directory, ready for analysis.


## Basic Statistics
Overview of Transactions and Token Transfers, Just a preliminary analysis.

We will get some basic statistics on Ethereum network activities and the treands.

#### Time Distribution
Because I limited the data to one day (2025-11-11), I can analyze the hourly distribution of transactions and token transfers within that day.

![Hourly Distribution of ETH Transactions](/assets/images/ethereum_network_analysis/1.png)

![Hourly Distribution of Token Transfers](/assets/images/ethereum_network_analysis/2.png)


#### Top Active Addresses
Top 5 Senders in Token Transfers and Their **Counts**:
- 0xfbd4cdb413e45a52e2c8312f670e9ce67e794c37    **-- 3681**
- 0x51c72848c68a965f66fa7a88855f9f7784502a7f    **-- 3344**
- 0xbbbbbbbbbb9cc5e90e3b3af64bdaf62c37eeffcb    **-- 2238**
- 0x9008d19f58aabd9ed0d60971565aa8510560ab41    **-- 2153**
- 0x000000000004444c5dc75cb358380d2e3de08a90    **-- 2124**

Top 5 Receivers in Token Transfers and Their **Counts**:
- 0xfbd4cdb413e45a52e2c8312f670e9ce67e794c37    **-- 3621**
- 0x51c72848c68a965f66fa7a88855f9f7784502a7f    **-- 3614**
- 0xa69babef1ca67a37ffaf7a485dfff3382056e78c    **-- 2489**
- 0x9008d19f58aabd9ed0d60971565aa8510560ab41    **-- 2306**
- 0xbbbbbbbbbb9cc5e90e3b3af64bdaf62c37eeffcb    **-- 2152**





#### Value Distribution Analysis
=== ETH Transactions ===
Rows: 13268
Unique from: 6368
Unique to: 5123
Unique addresses: 7796

=== Token Transfers ===
Rows: 103796
Unique from: 19555
Unique to: 19393
Unique addresses: 24661


![Value Distribution of ETH Transactions](/assets/images/ethereum_network_analysis/3.png)

## Whale Transactions

Top Whale Addresses - Transfers Over $10,000 USD Equivalent:
We can clearly see some exchange addresses and smart contract addresses here. 
![Whale Transactions Over $10,000](/assets/images/ethereum_network_analysis/4.png)


