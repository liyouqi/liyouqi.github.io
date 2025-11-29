---
title: "Market Fear Regime Identification - Part2: Data Pipeline & Feasibility Study"
date: 2025-11-28
categories:
- Data Mining
tags:
  - Cryptocurrency
  - Market Regimes
  - Unsupervised Learning
  - K-Means Clustering
  - Time-Series Analysis
  - Volatility
  - Fear & Greed Index
  - Returns and Sentiment
  - Data Pipeline

layout: single
author_profile: true
read_time: true
comments: false
share: true
mermaid: true
---

## Feature Engineering

### Design Philosophy

I'm starting simple and expanding later. This feasibility study uses just 3 features: BTC returns, BTC volatility, and Fear & Greed sentiment. I want to validate that the basic approach works before adding 10+ multi-asset features. This avoids overfitting and keeps the initial clustering interpretable.

### Feature Set (Feasibility Study)

#### 1. Daily Log Returns (ret_btc)

```python
df['ret_btc'] = np.log(df['BTC'] / df['BTC'].shift(1))
```

I use log returns because they're additive across time (easier for rolling calculations), symmetric (a +10% and -10% change have equal magnitude in log space), and standard practice in quantitative finance. The distribution shows a mean around 0.2% daily (indicating long-term uptrend) with high kurtosis.

What is the meaning of symetric here, 

#### 2. Annualized Volatility (vol_btc_7)
 **A really important feature for regime detection.**  
```python
df['vol_btc_7'] = df['ret_btc'].rolling(7).std() * np.sqrt(365)
```

Why vol_btc_7?

I chose a 7-day rolling window because it captures recent volatility spikes without over-smoothing (compared to a 30-day window). And if choosing 1-day volatility, it would be too noisy. 7 days is a good compromise.
It's responsive to regime changes — fear regimes show sudden volatility surges. This balances noise reduction with sensitivity.

The annualization factor √365 converts daily volatility to an annual percentage, which is industry standard. This lets us compare crypto volatility with traditional assets (S&P 500 is around 15-20% annual volatility). The formula is σ_annual = σ_daily × √(252 or 365). I use 365 since crypto trades 24/7, unlike stocks which take weekends off.

For example, if the 7-day raw standard deviation is 0.0312 (daily), the annualized value is 0.0312 × √365 ≈ 0.596, or 59.6% annual volatility. For crypto, this is typical during turbulent periods.

#### 3. Normalized Fear & Greed (`fg_norm`)

```python
df['fg_norm'] = df['fg_raw'] / 100  # Scale 0-100 to 0-1
```

**Why normalize?**
- Ensures all features have comparable scales for K-Means (distance-based algorithm)
- Alternative: could use StandardScaler, but [0,1] range preserves interpretability

**Distribution**:
- Mean ≈ 0.47 (market slightly below neutral on average)
- Bimodal: clusters around extreme fear (<0.3) and greed (>0.7) periods

### Feature Space Analysis

After computing features, we drop rows with NaN values (first 7 days lack rolling volatility) and standardize:

```python
from sklearn.preprocessing import StandardScaler

X = scaler.fit_transform(df_feat[['ret_btc', 'vol_btc_7', 'fg_norm']])
```

**Why StandardScaler?**
- Removes mean and scales to unit variance
- Critical for K-Means since it uses Euclidean distance
- Without scaling, volatility (range ≈ 0-2) would dominate over normalized F&G (range 0-1)

**Feature Space Characteristics**:
- **Volatility vs. Sentiment**: Shows clear separation between calm periods (low vol, high F&G) and panic (high vol, low F&G)
- **Returns**: More noise, but extreme positive/negative returns align with regime boundaries
- **Correlations**: Volatility and F&G have weak negative correlation (-0.3), confirming they capture different dimensions

### Future Feature Expansion (Phase 2)

Once feasibility is proven, we'll add:
- **Market Breadth**: % of coins with positive returns (captures systemic rallies/crashes)
- **Cross-Asset Correlation**: Rolling 30-day correlation matrix (detects contagion)
- **Volatility Spread**: Std of volatilities across all 20 assets (regime-specific dispersion)
- **Volume Anomalies**: Z-score of trading volume (extreme fear → volume spikes)

**Decision Point**: Use PCA only if feature count >10 and multicollinearity is high (will validate with correlation heatmap)

---

## **Feasibility Study Results**

### Clustering Setup

- **Algorithm**: K-Means (initialized with k-means++, random_state=42)
- **Number of Clusters**: k=5 (determined via Elbow Method + Silhouette Analysis)
- **Features**: Standardized `ret_btc`, `vol_btc_7`, `fg_norm`

### Why k=5 Instead of k=3?

I initially expected 3 clusters (Fear, Neutral, Greed), but the data told a different story. Running the Elbow Method and Silhouette analysis across k=2 to k=10 showed that k=5 gives the best balance.

![K-value selection using Elbow Method and Silhouette Score](/assets/images/market_fear_regime/7.png)

The Silhouette score peaks at k=5 (0.487) compared to k=3 (0.441), and the Davies-Bouldin Index is lowest at k=5 (0.712) versus k=3 (0.863). The elbow plot shows inertia drops about 60% by k=5 and then flattens, confirming this is where we get diminishing returns from adding more clusters.

The market actually has 5 distinct regimes, not just 3. Looking at the cluster sizes, we have:

- Cluster 0 (Neutral): 294 days — low volatility (38%), medium sentiment (50%)
- Cluster 1 (Mild Fear): 1026 days — medium volatility (52%), low sentiment (35%) 
- Cluster 2 (Extreme Fear): 313 days — high volatility (75%), very low sentiment (28%)
- Cluster 3 (Optimism): 241 days — low volatility (35%), high sentiment (68%)
- Cluster 4 (Extreme Greed): 982 days — medium volatility (45%), very high sentiment (82%)

The distribution shows that markets spend most time in either Mild Fear (1026 days) or Extreme Greed (982 days), while Optimism is relatively rare (241 days). This asymmetry makes sense — crypto markets tend to oscillate between cautious pessimism and irrational exuberance, with brief periods of balanced optimism.

### Cluster Quality Metrics

| Metric | Value | Interpretation |
|--------|-------|----------------|
| **Silhouette Score** | 0.487 | Good separation (>0.4 threshold) |
| **Davies-Bouldin Index** | 0.712 | Low overlap between clusters (closer to 0 is better) |
| **Calinski-Harabasz Score** | 1847.3 | High inter-cluster variance (indicates well-defined groups) |

### Visualization Insights

The feature space plot shows how the clusters separate across the volatility and sentiment dimensions. Each regime occupies a distinct region, with clear boundaries between them.

![Feature space clustering showing 5 distinct regimes](/assets/images/market_fear_regime/3.png)

The feature space plot uses viridis colormap (0=purple to 4=yellow) to show the 5 clusters. Looking at the scatter:
- **Low volatility (<0.5) + High sentiment (>0.7)**: Yellow/cyan clusters (Greed regimes)
- **High volatility (>1.5) + Low sentiment (<0.3)**: Purple clusters (Extreme Fear)
- **Medium volatility (0.5-1.5) + Low-mid sentiment (0.2-0.5)**: Blue/green clusters (Mild Fear and Neutral)
- **Notice**: Very few points in the "high vol + high sentiment" region — markets don't exhibit confident chaos.

Mapping these regimes over time reveals that transitions align well with known market events:

![Regime evolution timeline from 2018 to present](/assets/images/market_fear_regime/4.png)

This is a dual-panel plot:
- **Top panel**: Fear & Greed Index over time, color-coded by detected regime
- **Bottom panel**: BTC 7-day volatility over time, also color-coded by regime

Key observations:
- **March 2020 (COVID)**: Sharp volatility spike to 3.5+ in bottom panel, with purple dots (Extreme Fear) dominating
- **2020-2021 Bull Run**: Top panel shows sustained high sentiment (0.7-0.9), cyan/yellow colors (Greed)
- **May 2022 (Terra/Luna)**: Volatility jumps to 1.5-2.0, sentiment crashes to 0.1-0.3 (purple cluster)
- **Nov 2022 (FTX)**: Another volatility spike with low sentiment
- **2023-2024**: Alternating between green (Mild Fear, 0.2-0.4 sentiment) and brown clusters (mid-range regimes)
- **Late 2024-2025**: Return to higher sentiment (0.6-0.8) with low volatility

Looking at the detailed cluster characteristics:

![Cluster feature distributions and statistics](/assets/images/market_fear_regime/6.png)

This heatmap confirms that each cluster has distinct feature signatures. The transition matrix reveals interesting regime dynamics:

**Regime Persistence** (diagonal values):
- Cluster 1 (Mild Fear): 0.81 — **most stable**, tends to stay in this state
- Cluster 2 (Extreme Fear): 0.76 — also persistent during crisis periods
- Cluster 4 (Extreme Greed): 0.42 — less stable, frequently transitions out
- Cluster 3 (Optimism): 0.18 — **most transitional**, rarely persists
- Cluster 0 (Neutral): 0.34 — moderate stability

**Key Transitions**:
- **From Neutral (0)**: Equal probability to Optimism (0.33) or Mild Fear (0.27) — balanced crossroads
- **From Optimism (3)**: 46% jump to Neutral, 22% to Mild Fear — cooling down from optimism
- **From Extreme Greed (4)**: 49% crash to Extreme Fear (2) — **greed directly flips to panic**
- **From Extreme Fear (2)**: 76% stays fearful, but 15% jumps to Extreme Greed (4) — V-shaped recovery
- **From Mild Fear (1)**: 81% persistence — the market's "default state" is sticky

This explains why Mild Fear dominates (1026 days): once you enter it, there's an 81% chance you stay there tomorrow. Meanwhile, Extreme Greed is unstable (42% persistence) and often crashes directly to Extreme Fear (49% probability), skipping intermediate states.

### Key Findings

The feasibility study validates that unsupervised clustering can successfully identify meaningful market regimes. Three simple features capture the regime structure without needing complex transformations. Using k=5 instead of k=3 improves cluster quality and reveals finer-grained market states. Most importantly, the detected regimes align with major market events, providing external validation of the approach.

---

## **Conclusion**

This feasibility study confirms that **unsupervised learning can effectively identify cryptocurrency market regimes** using a minimal feature set. The validated approach now serves as a foundation for:

- Scaling to multi-asset features (Phase 2)
- Network analysis to study systemic risk (Phase 3)

Feasibility is proven, and the validated approach is ready for Phase 2 expansion.

---

*Notebook Files: `01_data_preparation.ipynb`, `02_feasible_study.ipynb`*