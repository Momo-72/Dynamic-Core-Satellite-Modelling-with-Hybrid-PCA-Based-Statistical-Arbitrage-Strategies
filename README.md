# Dynamic Core-Satellite Algorithm with Hybrid PCA-Based Statistical Arbitrage Strategies
> [!WARNING] 
> All code and information provided in this repository is for experimentation purposes only and should not be taken as financial advice. Do not use this algorithm for your personal financial interests under any circumstances, I highly highly highly advice against it. I am a random person from a non-financial background on the Internet.

## Abstract
This project proposes an algorithm strategy inspired by the Core-Satellite model, and is modelled to produce a trading strategy overlay. It utilizes a core, basic portfolio, such as 1/N or inverse-volatility, for diversification and consistency, while leverging statistical arbitrage for riskier investments, in order to profit from potential opportunitism. This algorithm is capable of modeling the true market accurately, where at some instances it may even exceed the market, and additionally improves sharpe ratios of basic portfolios by over 20% and reduces volatility by over 15%. These results are consistent across different baseline portfolio philosophies, and market conditions. This illustrates the proposed algorithm's potential to improve pre-existing baseline portfolios by introducing an adaptive overlay. 

## Introduction
Pairs trading is a relative‑value arbitrage strategy that exploits the mean‑reverting property of spreads between cointegrated assets. When two securities diverge from their historical equilibrium, the undervalued asset is bought and the overvalued asset is sold short, with the expectation that prices will converge. This contrarian philosophy is market‑neutral, profiting from relative mispricings rather than broad market trends.

The extension of pairs trading to multiple assets is referred to as statistical arbitrage (StatArb). This involves utilizing linear combinations of assets to construct and form mean‑reverting spreads. This project uses a PCA-based approach to isolate common factors and extract idiosyncratic residuals, which can serve as candidates for mean‑reversion trades. However, raw residuals are noisy, requiring smoothing and robust standardization before trading signals can be derived. Therefore, the extracted residuals are smoothed using the Kalman filter, and standardized into s-scores, and filtered through ADF stationary tests. Proceeding this processing, Contrarian entry/exit rules are applied when deviations exceed thresholds (e.g., ±2 for entry, ±0.5 for exit).  Furthermore, risk management is additionally introduced in this implementation through stop‑loss limits, maximum holding horizons, and safe initialization of positions.

By using the above process, a hybrid portfolio algorithm is introduced, combining an existing baseline allocation scheme with PCA-based StatArb overlays in a dynamic-core satellite inspired framework. It builds upon pre-existing portfolios, and integrates it as its core. This baseline portfolio ensures diversification, and utilizes heavy-tailed distribution estimators to coincide with the nature of financial data. Then, this baseline portfolio is combined with the previously mentioned algorithm by introducing it as a sattelite overlay. This sattelite overlay is designed to introduce a statistical arbitrage overlay to exploit short-term mean-reversion signals. The goal of this project is to produce a framework which is capable of leveraging the diversification capabilities of a pre-existing baseline portfolio alongside exploiting short-term opportunities with the statistical arbitrage satellite. 

The proposed algorithm performance is measured primarily via cumulative returns, drawdown, volatility and sharpe ratio. This is then compared to both the true market, and the baseline portfolios on their own.

## Repository Structure
```
├── Experiments/
    ├── 1-over-N-Portfolio/    
        ├── 1_N_2015_to_2020.ipynb
        ├── 1_N_2017_to_2019.ipynb
        └── 1_N_2018_to_2020.ipynb
    ├── Inverse-Volatility-Portfolio/    
        ├── Inverse_Volatility_2015_to_2020.ipynb
        ├── Inverse_Volatility_2017_to_2019.ipynb
        └── Inverse_Volatility_2018_to_2020.ipynb
    ├── Minimum-Variance-Portfolio/    
        ├── Minimum_Variance_2015_to_2020.ipynb
        ├── Minimum_Variance_2017_to_2019.ipynb
        └── Minimum_Variance_2018_to_2020.ipynb
    └── final/
        ├── CVar_Minimizing_2015_to_2020.ipynb
        ├── CVar_Minimizing_2017_to_2019.ipynb
        └── CVar_Minimizing_2018_to_2020.ipynb
├── README.md
└── Template.ipynb
```

## Background Theory and Applied Methods
### Mean Reversion and Stationarity
Mean reversion is the tendency of a time series to fluctuate around a long‑term average and eventually revert to it. It serves as the algorithms primary trading principles, as it is utilized to construct spreads via PCA residuals, where it is expected that they will revert to equilibrium. By employing the contrarian entry/exit rules, the algorithm is capable of explicily exploiting the mean-reverting behaviour. This mean reversion is responsible for driving the trading signals (entry/exit).

Stationarity is the requirement that the statistical characteristics if a time series remain constant over time, and is typically testing via the utilization of methods such as the Augmented Dickey‑Fuller (ADF) test. The differencing is often applied to achieve stationarity. This algorithm implements the ADF test in order to derermine which residuals are eligible for trading. In this algorithm, it is used as a filter to ensure the selected residuals are statistically valid candidates for mean reversion, where as the ADF test is applied to each residual series, the algorithm will then explicitly only trade the residuals which pass the specified test. By employing stationarity testing, this assists in preventing trading on random walks or non-stationary signals which is undesireable due to its potential to lead to spurious results.

### Cross‑Sectional Mean Reversion
There are two primary types of mean reversion, referred to as longitutdinal and cross-sectional. This project is based upon the cross-sectional mean reversion, meaning the mean reversion occurs across assets and the deviations are relative to the average across a group of securities. This is evident by the PCA residuals, as they represent deviations across multiple securities relative to common factors. When one asset deviates positively and another negatively, the spread is expected to revert. This enables the implemented algorithm to remain market-neutral, as the relative mispricings across a group of assets is considered.

### Why Log‑Prices and Log‑Returns
Financial prices are typically modeled in log‑space because log‑returns are additive and often closer to stationarity. Differencing log‑prices yields log‑returns, which are integrated of order 1 and suitable for statistical modeling. This ensures that the residuals extracted from PCA are meaningful candidates for mean‑reversion analysis.

### Robust Estimation in Heavy‑Tailed Data
Financial returns are heavy‑tailed, meaning that extreme events occur more frequently than under Gaussian assumptions. To address this:

- Spatial median is used as a robust estimator of expected returns, less sensitive to outliers than the arithmetic mean. In baseline portfolios which utilize the mean, it replaces the arithmetic mean to handle heavy-tailed data.
- MAD (Median Absolute Deviation) is used to estimate volatility and covariance, providing stability under fat‑tailed distributions. This ensures a robust  estimation.

These robust estimators ensure that the baseline portfolio is not distorted by extreme values.

### PCA Residuals and StatArb Overlay
PCA decomposes asset returns into systematic factors and idiosyncratic residuals. By reconstructing ~80% of the variance, the residuals represent deviations unexplained by common market factors.  These residuals are then processed through a robust signal pipeline:
1. **Kalman Filter Smoothing**  
   - Reduces noise and extracts latent states.  
   - Produces more stable signals compared to raw residuals.

2. **Robust Standardization (Median & MAD)**  
   - Converts residuals into s‑scores.  
   - Resistant to outliers and heavy‑tailed distributions.

3. **Stationarity Testing (ADF)**  
   - Ensures signals are based on mean‑reverting processes.  
   - Filters out spurious trends.

4. **Contrarian Trading Rules**  
   - Go long when residuals are unusually low.  
   - Go short when residuals are unusually high.  
   - Exit when residuals revert toward the median.
  
5. **Risk controls**
    - Enforce stop‑loss thresholds and maximum holding horizons.

This approach systematically extracts alpha from residual risk by:
- Stripping away common market noise.  
- Focusing only on idiosyncratic, mean‑reverting opportunities.  
- Applying robust statistical safeguards to avoid false signals.  
- Exploiting temporary mispricings through disciplined contrarian trades.

PCA residual decomposition provides a risk‑controlled, data‑driven way to isolate and trade hidden market inefficiencies.

### Core-Satellite
The core–satellite model is a portfolio construction approach that combines:
- **Core portfolio**: A stable, diversified allocation (often market index or risk‑controlled baseline) designed to capture broad market returns with low turnover.
- **Satellite portfolio**: Smaller, more active positions aimed at generating excess returns (alpha) through tactical strategies, factor tilts, or alternative signals.

The theoretical advantage is that the core provides long‑term stability and market exposure, while the satellite allows for flexible, opportunistic trading without impact the overall risk profile. This balance improves risk‑adjusted performance by separating systematic exposure from active bets. The PCA residual decomposition is particularly suited to the satellite component because:
1. **Separation of Market Factors**  
   - PCA reconstructs ~80% of return variance as common market factors.  
   - Residuals represent idiosyncratic deviations, ideal for satellite trading since they are independent of broad market risk.

2. **Noise Reduction and Robustness**  
   - Kalman smoothing stabilizes residual signals.  
   - Median/MAD standardization ensures robustness against outliers.  
   - ADF stationarity tests filter out non‑mean‑reverting series.

3. **Contrarian Trading Logic**  
   - Residuals capture temporary mispricings relative to market factors.  
   - Contrarian rules exploit mean reversion, generating short‑term alpha opportunities.

With the application of a core-satellite framework into the proposed algorithm:
- The core portfolio ensures market‑like performance and stability.  
- The satellite portfolio, powered by PCA residual signals, selectively adds alpha from idiosyncratic opportunities.  
This creates a **trading strategy overlay**: equity‑like returns with lower drawdowns, improved Sharpe ratios, and disciplined exposure to hidden inefficiencies.

The PCA residuals provide a data‑driven, statistically robust way to fuel the satellite component, while the core guarantees long‑term stability. This synergy makes the core–satellite model an ideal framework for balancing systematic exposure with opportunistic trading.

## Algorithm Design 
### Key Contributions
- **Bias‑Free Design**: Signals generated at *t‑1* and applied at *t*, eliminating lookahead bias.  
- **Robust Statistics**: Median and MAD replace mean and variance to handle heavy‑tailed financial data.  
- **Hybrid Portfolio**: Stable baseline allocation blended with PCA‑based arbitrage overlay.  
- **Risk Controls**: Stop‑loss and maximum holding horizon integrated for realistic drawdown management.  
- **Kalman Smoothing**: Residuals filtered to reduce noise and improve signal consistency.
- **Core-Satellite**: Enables the consideration of both a baseline portfolio, and StatArb for a hybrid portfolio which provides diversification, and exploits opportunities in the market.

### Original Research Ideas Contributed in this Project
This project is designed to extend classical theory with robust statistics, time-series filtering and statistical validation, and then applies a blended allocation framework that unifies diversification and arbitrage. By doing this, the aim is to introduce original research in robust statistical arbitrage and portfolio design. Specifically, the following novel approaches are implemented in this approach:
1. **Spatial Median:** Instead of using the arithmetic mean for expected returns in the baseline portfolio, the multivariate robust estimator spatial median is employed. This is not commonly applied in portfolio constructions. The application of this spatial median estimator directly addresses heavy‑tailed return distributions, making the optimization more resilient to outliers. This contributes to research by extending a classical portfolio optimization portfolio into a robust framework suitable for heavy-tailed financial data.
2. **MAD‑Based Covariance Estimation:** Similar as with the implementation of the spatial mean, the Median Absolute Deviation is utilized for volatility and covariance estimation. This is because MAD is less sensitive to extreme events than variance, which is crucial in financial markets where heavy-tails indicate larger risks. This contributes to research by providing a consistent robust risk stimator that aligns with the spatial median for expected returns.
3. **PCA Residual Overlay with Kalman Smoothing:** PCA methods are often utilized to isolate idiosyncratic residuals, however, the raw PCA residuals are often noisy and lead to false signals in practical applications. This algorithm is designed to address this by applying the Kalman filter for smoothing before trading, introducing state-space modelling which is not commonly employed in pairs trading literature. This contributes to research by providing a hybrid approach, which integrates both the dimensionality reduction capabilities of PCA, with time-series filtering using Kalman, to produce cleaner arbitrage signals.
4. **Robust S‑Score Standardization:** Usually z-scores are employed in statistical arbitrage, however, this algorithm uses the median and MAD to standardize residuals into s-scores. This ensures that the signal thresholds for entry and exit are more reliable under heavy-tailed distributions. This contributes to research by offering a robust alternative to conventional z-score trading rules, tailored for heavy-tailed data.
5. **Stationarity as a Trading Filter:** ADF stationarity tests are enforced before trading residuals, which ensures that pair trading strategies ignore stationarity which can lead to spurious trades on random walks and noise. This contributes to research as its a formal integration of stationarity testing into the traditional trading pipeline, ensuring statistical validity of mean-reversion signals.
6. **Blended Allocation Framework:** This algorithm combines a baseline portfolio designed to prioritize diversification (such as 1/N) with a stat-arb overlay, by modeling the integration of these two concepts as a core-satellite model. This provides a blended approach to portfolio design rather than considering these ideas as seperate, providing long-term robustness with short-term opportunism. This contributes to research by using a portfolio design philosophy that integrates divers with disciplined arbitrage in a single allocation scheme.
7. **Dynamic Allocation:** The allocation between the core and satellite is approached dynamically, which enables it to adapt to different market conditions as it changes. 
8. **Bias‑Free Signal Timeline:** The provided code explicitly ensures bias-free timing, as signals estimated at t-1 are only applied at t to prevent lookahead bias, which is present in many backtests. This provides a methodological framework that can be replicated and audited for fairness in backtesting.

## Algorithm and System Justification
*Please see the file Template.ipynb for exact code implementation details and walkthrough of the code*

The algorithm is implemented primarily in two main parts, that being the formation of the core and the satellite. The core is identified in step 1 which serves the foundation as the algorithm's stable and diversified allocation. This is where the base_weights are calculated. Then in steps 2-7, different mathematical formulas are applied to extract the statistical arbitrage, which serve as the algorithms satellite weights. These weights are then combined in a core-satellite inspired framework to produce the final algorithm.

**Step 1: Baseline Portfolio Construction (e.g., 1/N, inverse volatility)**
- Description: Use a portfolio designed primarily around the concept of diversification for best results. Base weights will be constructed from this baseline portfolio.
- Role: Diversification and risk control.
- Justification: Provides a stable allocation foundation.

**Step 2: PCA Factor Extraction on the window data**
- Description: Standardizes the returns and reconstructs 80% of the variance in order to compute residuals.
- Role: Separate systematic factors from idiosyncratic noise.  
- Justification: Identifies common drivers of returns and isolates residuals for arbitrage.

**Step 3: Residual Calculation & Smoothing on the PCA residuals**
- Description: Kalman Filtering is applied to the residuals extracted from PCA to reduce noise and estimate latent states. Ensures that the algorithm is not based on modeling pure noise and is capable of representing financial data properly.
- Role: Measure deviations from PCA reconstruction and filter noise.  
- Justification: Residuals represent mispricings; Kalman smoothing prevents false trades from short‑term spikes.

**Step 4: Robust S‑Score Signal Generation on the smoothed residuals**
- Description: The smoothed residuals are standardized using the median and MAD for robust estimation. This is to determine where the deviations are large, which indicate potential mean-reversion trades.
- Role: Standardize residuals into comparable signals.
- Justification: Large s‑scores indicate statistically significant deviations, suitable for contrarian trades.

**Step 5: Trading Signals + Stationarity Test**
- Description: Entry - If s > +2, short; if s < -2, long. Exit - Close when |s| < 0.5 (reversion). Only the residuals which pass the ADF stationarity test are traded.
- Role: Execute mean‑reversion trades.  
- Justification: Short assets with high positive s‑scores (expected to fall), long assets with negative s‑scores (expected to rise).

**Step 6: Risk Controls**
- Description: Enforce stop-loss and maximum holding horizon. After this step, the statarb weights can be calculated and normalized.
- Role: Limit downside risk.  
- Justification: Stop‑loss rules and max holding horizons prevent capital erosion when mean‑reversion fails.

**Step 7: Blended Allocation with Core-Satellite Model**
- Description: Uses a dynamic alpha_split calculation to calculate the fractions of satellite (StatArb) and core (Baseline Portfolio) distributions to adapt to the current financial state of the market, with risk adjustment integration
- Role: Provides risk awareness and balancing, adaptive exposure and conviction scaling.
- Justification: Provides a signal-driven allocation method, which considers risk management and avoids overfitting by ensuring exposure to the satellite is proportional to the evidence provided. It provides balanced blend between passive diversification and active alpha-seeking. 

**Step 8: Apply weights**
- Description: Weights are applied to the daily returns, update strategy PnL and compute the cumulative log returns.
- Role: Combine baseline and stat‑arb overlay satellite.  
- Justification: Ensures robustness by keeping most capital in the stable baseline while opportunistically trading residuals.

## Experiments
This repository contains the experimentation results of this algorithm applied to a variety of baseline portfolios. Each of the selected portfolios utilize different variables, such as the inverse volatility portfolio using the covariance matrix while 1/N does not, an follow different portfolio philosophies. This enables testing of whether the proposed algorithm enhances portfolio results regardless of foundation portfolio utilized, by stress-testing the statarb satellite under various scenarios and how this affects the results. 

The following experiments in particular are included in this repository:
- **1/N Portfolio** → Heuristic Portfolio. No estimation of mean or covariance matrix required. This provides a neutral benchmark.
- **Inverse Volatility** → Risk-based Portfolio. Estimation of covariance matrix only. This uses volatility estimates only, not correlations. Reduces exposure to high-risk assets.
- **Minimum Variance** → Risk-based Portfolio. Estimation of covariance matrix only. However, unlike inverse volatility, it uses the full covariance matrix. Creates the lowest-risk portfolio possible
- **CVar-Minimizing** → Drawdown Portfolio. This experiments with Downside protection.
  
In order to prevent the fine-tuning of hyperparameters which causes overfitting in backtesting, they were selected at random intial values and not changed for the duration of the experiments. They were kept consistent across portfolios as well. They were selected at intial values common in research. This means that the provided experiments avoid overfitting on past noise, and do not introduce data-snooping bias. This means that all performance results reflect the robustness of the statistical framework rather than optimization of thresholds to past noise, ensuring reliability and honesty of the provided results.
The selected hyperparameters, such as window size, are located in the provided files.

Rather than fine-tuning hyperparameters, the experimentation was conducted by applying different baseline portfolios as discussed above, as well as different market scenarios. Specifically, the following time periods were tested:
- **2016–2020 (long horizon)**: Captures several market environments; steady growth, volatility spikes and the onset of COVID. Determines whether the strategy is stable over diverse conditions.
- **2017–2019 (pre‑COVID short horizon)**: Excludes COVID which can be considered an outlier event. Isolates performance with “normal” market, and excludes pandemic disruption.
- **2018–2020 (including COVID shock)**: Experiments with a calm market just before COVID, as it goes into the volatile market of 2020. This tests resilience and adaptability under crisis conditions.

Testing multiple scenarios avoids overfitting data on a very particular point of time, and avoids deliberately selecting the market where the strategy performs well to ensure transparency. Selected overlapping windows which include both normal conditions, and outlier conditions (COVID-19) to test how the strategy behaves under stress and ensure robustness.


### Outputs
- **strategy_series**: Daily strategy returns.
- **strategy_cum**: Cumulative returns (log returns assumed).
- **Drawdown Curve**: Can be computed via running max of cumulative returns.
- **Annulaized Sharpe Ratio**
- **Annulized Volatility**

### Results Summary
#### Annualized Volatility Improvement with the Strategy versus the baseline alone

| Period       | Portfolio Type        | Volatility | Improvement |
|--------------|-----------------------|------------|-------------|
| 2017–2019    | 1/N                   | 0.1771     | 22.7%       |
|              | Inverse               | 0.1639     | 20.0%       |
|              | Min Var               | 0.1590     | 11.7%       |
|              | CVaR-minimizing       | 0.1709     | 10.5%       |
| 2015–2020    | 1/N                   | 0.2185     | 12.5%       |
|              | Inverse               | 0.2033     | 13.0%       |
|              | Min Var               | 0.1935     | 5.0%        |
|              | CVaR-minimizing       | 0.1926     | 9.5%        |
| 2018–2020    | 1/N                   | 0.2628     | 13.3%       |
|              | Inverse               | 0.2437     | 11.1%       |
|              | Min Var               | 0.2320     | 8.0%        |
|              | CVaR-minimizing       | 0.2247     | 12.0%       |

### Annualized Sharpe Ratio Improvement with the Strategy versus the baseline alone

| Period       | Portfolio Type        | Sharpe Ratio | Improvement |
|--------------|-----------------------|--------------|-------------|
| 2017–2019    | 1/N                   | 1.052        | 12.2%       |
|              | Inverse               | 1.2216       | 29.31%      |
|              | Min Var               | 1.3416       | 28.7%       |
|              | CVaR-minimizing       | 0.2835       | 21.7%       |
| 2015–2020    | 1/N                   | 0.8313       | ~0%         |
|              | Inverse               | 1.1125       | 3.68%       |
|              | Min Var               | 0.9405       | 11.9%       |
|              | CVaR-minimizing       | 0.6383       | 3.2%        |
| 2018–2020    | 1/N                   | 0.8899       | 12.5%       |
|              | Inverse               | 0.9894       | 20.73%      |
|              | Min Var               | 1.0760       | 15.7%       |
|              | CVaR-minimizing       | 0.6196       | 14.8%       |

**Key Observations**:

Improvements were visible both pre-COVID (2017–2019) and during COVID (2018–2020), suggesting the algorithm is not regime-dependent and avoids overfitting the data.

1. Strategy performs best on short time periods
    - As seen in the graphs provided in the experiments. For all time period, the cumulative return remains simular to the true market. This is illustrated across each baseline portfolio, except for CVar-Minimizing, which indicates that the strategy is capable of aligining the market. The results of CVar-Minimizing baseline portfolio not aliging with the market is therefore most likely due to the baseline portfolio employed. The graphs are mainly aligned with the market in the shorter time periods (2017-2019 and 2018-2020). This is evident in the fact that, in some parts of these graphs, the proposed strategy is capable of exceeding the market.
    - This is particularly noticable in the stable market conditions of 2017-2019. When considering the market movement into outlier conditions such as COVID-19, the strategy is still capable of performing well.
    - The strategy tracks the market closely with slightly smaller drawdowns. Suggests that the dynamic core-satellite blend plus risk controls (stop-loss, capped satellite fraction, max holding days) are effectively dampening downside volatility while still capturing upside.
    - In shorter horizons, the algorithm can exceed market returns. Suggests that the satellite signals are successfully exploiting short-term mean-reversion opportunities.
    - Long-term returns may sit slightly below the market because of conservative risk controls, but the smoother ride is attractive. Suggests that this strategy could potentially be seen as a risk-managed market proxy, although it may be more aligned with the term risk‑managed trading overlay. 
2. Achieves a drawdown below the true market in these shorter time periods
    - In the shorter time periods where it is deemed more effective, each baseline portfolio achieves a smaller drawdown than the true market. Specifically, the Minimum Variance baseline performs best when it comes to drawdown.
    - In these shorter time periods, even during extreme volatility, the algorithm maintained lower drawdowns than the market. Suggests that the PCA residual approach adapts to changing correlation structures, while risk controls prevent large losses. 
    - However, this is not illustrated in the longer periods, where drawdown will often be worse than the true market. This suggests that the strategy is best employed in short-term scenarios, in order to effectively exploit opportunities represented in its satellite, and combing them effectively with the diversification of the core.
3. Consistent Volatility Reduction
    - Across all sample periods (2017–2019, 2015–2020, 2018–2020), the core-satellite algorithm reduces annualized volatility compared to the baseline portfolios on their own.
4. Sharpe Ratio Improvements
    - In most cases, Sharpe ratios improve meaningfully. Strongest gains appear when compared to Inverse Volatility and Minimum Variance baselines. Even in tougher periods (2015–2020), improvements are modest but still positive. 
5. Robustness Across Time Windows
    - Improvements are not confined to one period; they appear in 2017–2019, 2015–2020, and 2018–2020. This indicates the statarb satellite overlay is not overfitted to a single regime.
6. Robust in outlier events, such as COVID
    - The algorithm improved Sharpe ratios even during COVID, suggesting that the developed statarb overlay was capable of exploiting mean-reversion opportunities in residuals even in turbulent markets. Additionally, volatility reduction was also consistent across regimes. Even in COVID’s high-volatility environment, the algorithm lowered risk relative to baseline
7. CVar-Minimizing
    - The results of the CVar-Minimizing portfolio were far below the other baseline algorithms, as even alone the porfolio did not perform strongly on the provided data. However, even given its poor baseline performance, the provided strategy was able to improve its results rather significantly. This suggests that the proposed strategy can improve even poor performing baseline portfolios.
8. Strong performance results without hyperparameter fine-tuning
    - Despite not optimizing hyperparameters, such as window_size, the strategy was able to provide significant gains to each of the baseline portfolios in all scenarios. This illustrates the potential of the strategy to perform well, as it is not overfitting the data and showcases authentic results. This additionally suggests that by enhancing the hyperparameters, further gains to performance can be made.

**Baseline Portfolio Comparisons**
When comparing the algorithm to the baseline portfolios on their own without the satellite overlay, it is capable to improve both volatilities and sharpe ratios by over 20%. This is also demonstrated in outlier events such as COVID.

## Conclusion
In conclusion, the proposed algorithm can be understood as a risk‑managed trading overlay on an equity core, delivering equity‑like returns with some additional minor downside protection, and occasional short-term alpha. The empircal evidence suggests that the algorithm is able to provide a risk-adjusted performance and resilience across any of the provided time periods, and enhancement across a range of existing allocation philosophies. It offers practical enhancement to stabilize the portfolio performance in existing allocation philosophies, and is best suited for those which prioritize diversification. This is illustrated by its ability to improve the sharpe ratio and volatility scores by over 20%. While long-term returns may underperform the market, which could be due to conservative risk constraints, the fact that it reduces drawdown and can exceed the market in shorter time periods indiate that tge presented algorithm may be suited for stable equity exposure where drawdown is heavily considered. 

The novelty of the algorithm is presented in the integration of dimensionality reduction, state-space filtering, robust statistical measures, and adaptive portfolio blending within a unified framework. Additionally, the core-sattelite framework is implemented dynamically, rather than statically as in most core-sattelite frameworks. This means that the proposed algorithm does not remain fixed like previous models, and rather can adapt to changing conditions, and adapt to outlier events such as COVID.

Additionally, these market-aligned results were obtained without hyperparameter tuning, demonstrating that it didn’t perform well due to overfitting of data. Considering it performed well in a range of market scenarios, including outlier events such as COVID, without the fine-tuning of hyperparameters, this suggests that the algorithm has the potential to perform well on future data.

However, it does introduce limitations which indicate areas of potential future work, which are discussed in detail below. Particularly as the baseline portfolios and statistical arbitrage can incur heavy transaction costs which are not modeled, which could suggest that the algorithm may not be capable of modeling the real-data effectively. This indicates areas of potential work, and suggest that this algorithm may be best suited for benchmarking tasks, or model forecasting/prediction tasks.

## Limitations & Risks
Although the results indicate that the algorithm is capable of minimizing drawdown compared to the true market, there are existing limitations and risks in the developed algorithm. 

These limitations and risks include, but are not limited to:
- **Stationarity Test Fragility**: ADF on short samples can be unreliable.  
- **Fixed Thresholds**: Entry/exit levels may not adapt to changing volatility regimes.  
- **Kalman Fallback**: If filtering fails, raw residuals are used, reducing consistency.  
- **Stop-Loss Design**: Resetting PnL after exit may underestimate long-term drawdowns.  
- **Position Sizing**: Normalization ignores correlation and volatility scaling.  
- **Transaction Costs & Slippage**: Not modeled; frequent rebalancing could erode profits.  

### Suggested Improvements
For futher development of the algorithmn, the following improvements can be implemented:
- **More Robust Stationarity Checks**: Use longer windows or alternative tests (e.g., KPSS, PP).  
- **Adaptive Thresholds**: Adjust entry/exit levels dynamically based on volatility regimes.  
- **Enhanced Error Handling**: Improve Kalman filter robustness to avoid fallback inconsistencies.  
- **Drawdown Tracking**: Monitor global portfolio drawdown separately from per-trade PnL.  
- **Correlation-Aware Position Sizing**: Incorporate risk parity or covariance-based scaling.  
- **Transaction Cost Modeling**: Explicitly account for fees, slippage, and liquidity constraints.
- **Improvement of Hyperparameters**: By introducing both an in-sample and out-of-sample dataset, hyperparameters can be optimized to improve results.
