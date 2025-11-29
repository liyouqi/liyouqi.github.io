---
title: "Market Fear Regime Identification - Part 3: Multi-Asset Feature Engineering"
date: 2025-11-29
categories:
- Data Mining
tags:
  - Cryptocurrency
  - Market Regimes
  - K-Means Clustering
  - Feature Engineering
  - Multi-Asset Analysis
  - Market Breadth
  - Correlation Analysis
  - Contagion Risk

layout: single
author_profile: true
read_time: true
comments: false
share: true
---

## Introduction

Part 1 established a baseline for market regime detection using three features: BTC daily returns, BTC 7-day volatility, and the normalized Fear & Greed Index. The clustering worked (Silhouette Score of 0.276 with k=5 clusters), but it had limitations. Using only BTC means we're ignoring what's happening across the broader market. If panic is spreading through multiple assets, we can't detect it. If only a few coins are rising while others decline, we miss that too.

This post expands the feature set to capture system-wide dynamics. I'm adding features that measure market breadth (how many coins are moving together), cross-asset correlations (market coupling), and volatility dispersion (contagion effects). The goal is to improve clustering quality by incorporating multi-asset signals.

## Baseline Review

Before adding new features, I reproduced the Phase 1 results as a benchmark. Using BTC daily log returns, BTC 7-day annualized volatility, and normalized Fear & Greed Index, the K-Means clustering with k=5 achieved:

| Metric | Value | Interpretation |
|--------|-------|----------------|
| Silhouette Score | 0.2756 | Moderate cluster quality |
| Davies-Bouldin Index | 1.1197 | Acceptable separation (lower is better) |
| Calinski-Harabasz | 914.69 | Solid baseline (higher is better) |

The Silhouette Score of 0.276 indicates moderate cluster quality. Clusters are reasonably separated but with some overlap. The Davies-Bouldin Index (lower is better) at 1.12 suggests acceptable separation. The Calinski-Harabasz score of 915 (higher is better) provides a solid baseline.

These metrics establish the benchmark. The question is whether multi-asset features can capture systemic risk patterns that BTC alone misses.

## Multi-Asset Feature Engineering

I engineered nine new features across three categories:

### Market Breadth Indicators

Market breadth measures how many assets participate in market movements. I calculated:

- `pct_positive`: Percentage of the 20 cryptocurrencies with positive daily returns
- `pct_above_ma50`: Percentage of assets trading above their 50-day moving average

The rationale is simple. During fear regimes, few coins rise together (low breadth). During greed regimes, widespread participation happens (high breadth). These features capture market-wide sentiment that individual asset features can't.

```python
ret_cols = [f'ret_{coin}' for coin in coin_cols]
df['pct_positive'] = (df[ret_cols] > 0).sum(axis=1) / len(coin_cols)

for coin in coin_cols:
    df[f'ma50_{coin}'] = df[coin].rolling(50).mean()
ma_cols = [f'ma50_{coin}' for coin in coin_cols]
df['pct_above_ma50'] = (df[coin_cols].values > df[ma_cols].values).sum(axis=1) / len(coin_cols)
```

### Cross-Asset Correlation Features

Correlation features measure market coupling and systemic risk:

- `btc_eth_corr_30d`: 30-day rolling correlation between BTC and ETH
- `mean_pairwise_corr`: Average correlation across all 20 coins

Financial theory suggests correlations spike during market stress as all assets move together in flight-to-safety behavior. In calm periods, correlations decrease as investors differentiate between assets. These features help distinguish coupled (systemic) from decoupled (idiosyncratic) market states.

```python
df['btc_eth_corr_30d'] = df['ret_BTC'].rolling(30).corr(df['ret_ETH'])

def calc_mean_pairwise_corr(window=30):
    corr_values = []
    for i in range(window-1, len(df)):
        window_data = df[ret_cols].iloc[i-window+1:i+1]
        corr_matrix = window_data.corr()
        upper_tri = corr_matrix.where(np.triu(np.ones(corr_matrix.shape), k=1).astype(bool))
        mean_corr = upper_tri.stack().mean()
        corr_values.append(mean_corr)
    return [np.nan] * (window-1) + corr_values

df['mean_pairwise_corr'] = calc_mean_pairwise_corr(30)
```

### Volatility Contagion Indicators

These features measure dispersion of volatility and returns across the 20 cryptocurrencies:

- `vol_dispersion`: Standard deviation of 7-day volatilities across all assets
- `ret_dispersion`: Standard deviation of daily returns across all assets

The contagion hypothesis is that during fear regimes with high contagion, dispersion is lower (all assets experiencing similar volatility/returns). During greed regimes with differentiated performance, dispersion is higher. These features quantify whether market stress is spreading uniformly or remaining isolated.

```python
for coin in coin_cols:
    df[f'vol_7d_{coin}'] = df[f'ret_{coin}'].rolling(7).std() * np.sqrt(365)
vol_cols = [f'vol_7d_{coin}' for coin in coin_cols]
df['vol_dispersion'] = df[vol_cols].std(axis=1)
df['ret_dispersion'] = df[ret_cols].std(axis=1)
```

## Feature Selection Through Testing

I now had 3 baseline features plus 9 multi-asset features (12 total). Rather than using all of them, I tested different combinations to find which features actually improved clustering quality.

After multiple experiments, the best configuration used only 4 features:
- `ret_btc`: BTC daily log return
- `vol_btc_7`: BTC 7-day volatility
- `fg_norm`: Normalized Fear & Greed Index
- `pct_positive`: Percentage of coins with positive returns

The other multi-asset features either introduced noise or were redundant. ETH features were highly correlated with BTC. Correlation features and contagion features didn't significantly improve the clustering. Only `pct_positive` (market breadth) added meaningful signal.

This was an important finding. Not all multi-asset features are useful. Market breadth captures system-wide risk appetite in a way that correlation or dispersion metrics don't.

## Optimal k Selection

Before clustering with the enhanced features, I needed to determine the optimal number of clusters. I tested k from 2 to 10, evaluating each with three metrics:

| k | Silhouette | Davies-Bouldin | WCSS |
|---|------------|----------------|------|
| 2 | **0.2861** | 1.4039 | 8179.06 |
| 3 | 0.2445 | 1.3604 | 6883.81 |
| 4 | 0.2743 | 1.2942 | 5799.25 |
| 5 | 0.2566 | 1.2173 | 5023.51 |
| 6 | 0.2516 | 1.1797 | 4491.19 |
| 7 | 0.2527 | 1.0789 | 4155.07 |
| 8 | 0.2491 | 1.1315 | 3830.54 |
| 9 | 0.2411 | 1.1507 | 3536.24 |
| 10 | 0.2360 | 1.1170 | 3335.86 |

**Optimal k (by Silhouette): 2 (Score: 0.2861)**

![Optimal k selection plots](/assets/images/market_fear_regime/11.png)

The Silhouette Score peaks at k=2 with 0.286, then drops significantly for k=3 (0.245). The Elbow plot shows WCSS decreasing steadily, but the "elbow" appears around k=2-3. The Davies-Bouldin Index improves with higher k, but this must be balanced against interpretability.

The key insight is that the market naturally forms two distinct regimes rather than five. This binary classification aligns with the Fear & Greed Index conceptual framework. The market operates in two fundamental states: Risk-Off (Fear) and Risk-On (Greed).

I decided to use k=2 for the enhanced clustering, even though this differs from the baseline's k=5. The data is telling me that two clusters provide better quality and clearer interpretation.

## Feature Correlation Check

Before finalizing the features, I checked for multicollinearity. All pairwise correlations were below 0.8, indicating the four features provide complementary information without redundancy. BTC return and volatility showed expected low correlation. Fear & Greed Index showed moderate correlation with market breadth, confirming they capture related but distinct signals. No need for PCA or dimensionality reduction.

![Feature correlation heatmap](/assets/images/market_fear_regime/12.png)

## Enhanced Clustering Results

Using 4 features with k=2, the enhanced model achieved:

| Metric | Baseline (3 feat, k=5) | Enhanced (4 feat, k=2) | Change |
|--------|------------------------|------------------------|--------|
| Silhouette Score | 0.2756 | 0.2861 | +3.8% ✅ |
| Davies-Bouldin Index | 1.1197 | 1.4039 | +25.4% |
| Calinski-Harabasz | 914.69 | 1132.30 | +23.8% ✅ |

The Davies-Bouldin increase is primarily due to the k-value change (k=5 → k=2) rather than feature quality degradation. When using fewer clusters, this metric naturally increases. However, the Silhouette Score (which fairly compares different k values) improved, indicating better-defined, more meaningful clusters.

The enhanced model successfully improves clustering quality despite using fewer clusters. This suggests the market naturally exhibits two dominant regimes rather than five, and the `pct_positive` feature adds meaningful signal.

## Regime Characteristics

The cluster statistics reveal distinct regime characteristics:

| Regime | Count (Days) | Mean Return | Volatility | Fear & Greed | Market Breadth |
|--------|--------------|-------------|------------|--------------|----------------|
| 0 (Fear) | 1,587 (55.6%) | -1.8% | 0.545 | 0.465 | 22.1% |
| 1 (Greed) | 1,269 (44.4%) | +2.4% | 0.567 | 0.486 | 68.3% |

![Regime statistics table](/assets/images/market_fear_regime/13.png)

Regime 0 (Fear) covers 1,587 days (55.6% of the dataset):
- Mean daily return: -1.8% (negative performance)
- Volatility: 0.545 (elevated)
- Fear & Greed: 0.465 (below neutral)
- Market breadth: 22.1% (only about 1 in 5 coins rising)

Regime 1 (Greed) covers 1,269 days (44.4%):
- Mean daily return: +2.4% (positive performance)
- Volatility: 0.567 (slightly higher)
- Fear & Greed: 0.486 (near neutral)
- Market breadth: 68.3% (strong participation, about 2 in 3 coins rising)

Market breadth is the key differentiator. The difference between 22% and 68% is dramatic, validating the choice to include this feature. It captures system-wide risk appetite better than sentiment indices alone.

Regime 0 is larger, meaning fear states persist longer (56% of days). Markets spend more time in risk-averse conditions than exuberant ones. There's also a volatility paradox: Regime 1 (Greed) shows slightly higher volatility (0.567 vs 0.545), possibly because active markets experience both up and down moves, while fear regimes see more unidirectional decline.

The fairly balanced distribution (56% vs 44%) indicates both regimes are persistent market states, not rare anomalies.

## PCA Visualization

I used PCA to project the 4-dimensional feature space into 2D for visualization. The first two principal components capture about 70-80% of the total variance, meaning most information can be visualized in 2D.

![PCA scatter plot](/assets/images/market_fear_regime/14.png)

The visualization shows clear separation between Regime 0 (red) and Regime 1 (turquoise). Some overlap exists in the boundary region, which is expected due to transitional periods. The separation validates the k=2 choice — the data naturally forms two clusters. This provides confidence that the clustering represents genuine structural differences rather than arbitrary partitioning.

## Regime Timeline

The timeline visualization shows how regimes evolve over time. Clear alternation occurs between Regime 0 (fear) and Regime 1 (greed).

![Regime timeline plot](/assets/images/market_fear_regime/15.png)

Major fear periods are visible during:
- Early 2020 (COVID crash)
- Mid-2022 (Terra/Luna collapse, crypto winter)
- Late 2022 (FTX bankruptcy)

Sustained greed periods appear during:
- Late 2020 through early 2021 (bull run)
- Q4 2023 into 2024 (recovery rally)

The bottom panel shows volatility over time. Extreme volatility spikes align with fear regimes. Regime transitions often occur during volatility surges. Both fear and greed regimes can exhibit elevated volatility, confirming the earlier observation.

Market regimes tend to persist for weeks to months before transitioning, suggesting they represent genuine market states rather than noise. Major crypto market events (COVID, Luna, FTX) clearly trigger fear regime classifications. Transitions from fear to greed often occur gradually rather than instantly.

## Key Takeaways

The enhanced model with k=2 and 4 features successfully improves upon the baseline. The addition of `pct_positive` (market breadth) captures system-wide participation patterns that the BTC-only baseline missed. The shift to k=2 reveals that the market fundamentally operates in two regimes (Fear vs Greed) rather than five.

Testing all nine multi-asset features revealed that most don't add value. ETH features are redundant with BTC. Correlation features and contagion features introduce more noise than signal. Only market breadth meaningfully improves clustering. This validates the importance of testing rather than assuming all theoretically-motivated features will work.

The clustering quality improved by 3.8% in Silhouette Score and 23.8% in Calinski-Harabasz Score. More importantly, the two-regime model provides clearer actionable insights for risk management. You can see when the market is in a fear state (low breadth, negative returns) versus a greed state (high breadth, positive returns).

These persistent regimes with clear structural differences make them ideal candidates for network analysis. The next phase will examine how correlation networks differ between these two distinct market states — whether fear regimes show higher market coupling and BTC centrality, while greed regimes show more decoupled, idiosyncratic movements.
