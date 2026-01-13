This repository contains the code for a backtest of a trading algorithm based on two novel indicators:
  - a volatility indicator (PCI)
  - a directional indicator (ASI)
The project began with the creation and testing of PCI. PCI was designed to incorporate intraday price movement and trading volume in order to capture market sentiment and future volatility. After testing PCI, it was confirmed that it did not contain directional information, so ASI was introduced to develop a functioning trading strategy.

PCI is computed as follows:
  - μ = (Shares Outstanding + Volume) × High
  - λ = (Shares Outstanding − Volume) × Low
  - PCI = (High − Low) / (μ − λ)
Mu and Lambda represent theoretical limits to a stock’s daily movement given observed volume and intraday price action. PCI measures the fraction of realised price movement relative to these limits.
High PCI values indicate higher expected future volatility, while low PCI values indicate lower expected future volatility.
This behaviour follows from the following theoretical scenarios:
  - Low volume, large price range → High PCI
    (fragile pricing, high future volatility)
  - High volume, large price range → High or moderate PCI
    (price discovery phase)
  - Low volume, small price range → Potentially high PCI
    (theoretical weakness of PCI)
  - High volume, small price range → Low PCI
    (strong market price confidence)

PCI was evaluated through correlation with:
  - future absolute returns
  - standard deviation
  - standard deviation calculated only from top K largest market movements
Several smoothing techniques and future horizons were tested. Bootstrapping was used to ensure statistical robustness, partial correlation was measured to determine the information added beyond standard volatility measures, and random PCI baselines were included to identify spurious correlations.

The key metrics considered were:
  - mean and median bootstrapped Spearman correlation
  - share of bootstraps with positive correlation
  - lift at extreme PCI deciles

The most robust configuration of PCI was found to be a 15-period weighted moving average, evaluated over a 10–15 day horizon.

Directional Indicator: ASI

Monte Carlo simulations confirmed that PCI does not contain directional information, as expected. To introduce directionality, ASI was developed.
ASI is calculated as follows:
  - Count the number of positive-return periods for each sub-period length (1 to N)
  - Divide by the total number of possible periods
  - Aggregate across all sub-period lengths
ASI measures the persistence of positive returns across multiple horizons, hence the name Asset Safety Index (ASI).
The version used in the strategy, referred to as ASI2 (or RASI), is defined as:
  - 15-day rolling ASI / 50-day rolling ASI
This configuration was selected based on correlation with future returns.

Strategy Construction
For each ticker, signal weights are calculated as:
  - ASIΩ = ASI^α / Σ(ASI^α)
  - PCIΩ = PCI^β / Σ(PCI^β)
  - Ω = ASIΩ + PCIΩ
Portfolio weights are proportional to Ω and can optionally be restricted to top-percentile ASI baskets.
The strategy supports both long-short and long-only implementations. Long-only was found to be the strongest configuration over the long term.

Findings
  - PCI meaningfully predicts future volatility
  - PCI and ASI together form an effective trading system
  - Performance is strongest during bullish market regimes
  - Rebalancing specifically on Fridays leads to statistically significant outperformance
      - A hypothesis for the Friday effect was higher trading volume, which could enhance PCI’s signal, or option expiration and            maximum pain dynamics. Trading volume was found to have a negligible effect, and option expiration effects were not                 explored further due to data limitations.
   
Open Questions & Future Work
  - Further investigation of option expiration effects
  - Testing the strategy across larger and international universes
  - Exploring higher-frequency ASI calculations (e.g. Hourly data)

For a more detailed overview with ouputs and results see the file named PCI+ASI Research Doc FV.2.pdf
