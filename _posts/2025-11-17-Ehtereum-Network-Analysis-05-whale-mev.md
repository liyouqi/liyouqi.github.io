---
title: "Ehtereum Network Analysis — Part 05: Whale & MEV Methodology"
date: 2025-11-17
categories:
  - Data Mining
tags:
  - Reflection
  - Ethereum
  - MEV
  - Whale Analysis
  - Blockchain
  - Cryptocurrency
layout: single
author_profile: true
read_time: true
comments: false
share: true
---

# Whale & MEV Methodology

Finally, the fun part.  
This chapter covers how I detect **whales** and potential **MEV‑style behavior** in the network. Again, this is based on **one day of data**, so the goal is to demonstrate methods, not claim market‑level insights.

---

## 1. Whale Detection

A whale is defined here as:

- ETH transfer > **$10k**  
- Token transfer > **$10k USD equivalent**  

After filtering, I identify about 11 unique whale addresses in the dataset. Not a lot, but enough for research purposes.

### 1.1 Whale Ranking

Top value whale:  0x28c6c06298d514db089934071355e5743bf21d60
Top PageRank whale: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
Top degree whale: 0xc02aaa39b223fe8d0a0e5c4f27ead9083c756cc2
Top overlap whale: 0x28c6c06298d514db089934071355e5743bf21d60

### 1.2 Centrality-Based Whale Detection

```python
deg = nx.degree_centrality(G_hetero)
pagerank = nx.pagerank(G_hetero)
```

High PageRank addresses are often:

- exchanges  
- routers  
- settlement contracts  

---

## 2. Whale Ego Graphs

```python
ego = nx.ego_graph(G_eth, whale_addr, radius=1)
```

![Whale Ego Graph](/assets/images/ethereum_network_analysis/whale_ego.png)

You can clearly see multiple inbound and outbound flows.


Net Flow: Value vs Centrality Whales
![Net Flow vs Centrality](/assets/images/ethereum_network_analysis/Whale_NetFlow_Centrality.png)


---


TODO


## 3. MEV Detection

I look at patterns such as:

- **very short time intervals** between actions  
- **repeated interactions** with router contracts  
- **triangular swap patterns** in the heterogeneous graph  

### 3.1 Router Frequency


### 3.2 Time Interval Patterns



## 4. Mini Case Study

