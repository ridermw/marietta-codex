# Crypto Trend-Following System – Repository Scaffolding Blueprint

## 1. Key Claims and Findings from Zarattini et al. (2025)

* **Ensemble Donchian Channel Strategy:** The paper introduces a trend-following strategy that combines multiple Donchian Channel breakout models with lookback windows ranging from 5 days to 360 days. These individual signals are aggregated into a single ensemble trend signal, paired with a volatility-targeted position sizing method. The position size is adjusted daily to target 25% annualized volatility using a 90-day rolling volatility, with leverage capped at 2× (200%) to prevent excessive exposure.

* **Data and Market Coverage:** The authors constructed a **survivorship bias-free** daily OHLCV dataset of **21,616 cryptocurrencies (2010–2025)** from CoinMarketCap. They exclude stablecoins, wrapped tokens, and NFT-collectible tokens to focus on investable assets. The strategy is first tested on Bitcoin, then extended to a **rotational portfolio** of the top 20 most liquid coins (selected by trading volume) to evaluate robustness across the broader crypto market.

* **Trade Entry/Exit Rules:** A long position is initiated whenever price closes above the *n*-day high (upper Donchian band), and the position is trailed by a stop set at the Donchian channel midpoint. The trailing stop moves up (never down) to the max of its prior value and the current channel midpoint. An exit triggers when price closes below the active trailing stop. This rule captures momentum upsides while cutting losses on trend reversals.

* **Volatility-Based Position Sizing:** Each strategy variant (for each lookback *n*) scales its daily position to target 25% annual volatility. In formula, `position_weight = 0.25 / sigma(90-day)` with `sigma` = annualized 90-day std. dev. of returns. If this computed weight exceeds 200%, it is capped at 200% (leveraged double exposure max). This dynamic sizing reduces position size during volatile periods and increases it during calmer periods.

* **Performance on Bitcoin:** On Bitcoin alone, the trend strategy delivers **\~30% annual CAGR** with substantially improved risk-adjusted metrics. Shorter lookback models (5–30 days) achieved the highest Sharpe ratios in single-asset tests. The combined **ensemble (“Combo”) model** on BTC yielded a **Sharpe ratio ≈1.58**, Sortino ratio \~2.0, **max drawdown \~19%** (vs. >80% drawdown for simple buy-and-hold BTC), and an **annualized alpha \~14% over passive Bitcoin**. Nearly all tested trend models (except the longest 150d/250d channels) produced statistically significant alpha (p < 0.025) versus a Bitcoin benchmark.

* **Multi-Asset Portfolio Results:** Expanding to a diversified portfolio of top cryptocurrencies greatly improved absolute and risk-adjusted returns. A portfolio of the **top 20 coins** (rebalanced monthly by volume) under the trend program achieved about **18% CAGR** with **Sharpe \~1.57** and **Sortino \~1.97**, while cutting maximum drawdown to \~11%. This 20-asset trend portfolio generated an **annualized alpha of \~10.8% over Bitcoin** (net of fees). Including more than \~20 assets yields diminishing marginal benefits – performance gains plateau and **Sharpe ratios stabilize around \~1.5** beyond 20 assets. Moderate diversification is sufficient to capture most benefits, as adding assets past a certain point mainly lowers volatility but doesn’t significantly raise risk-adjusted returns.

* **Transaction Costs & Mitigation:** The backtests account for realistic **transaction costs** (tested at 10, 25, 50 bps per trade). To combat performance drag from trading fees, the authors employ a **20% rebalancing threshold** – i.e. only rebalance position sizes when the volatility-targeted allocation deviates by >20% due to market moves. This simple rule reduces turnover and trading frequency, significantly mitigating cost impact without materially hurting performance. All reported results (e.g. Sharpe \~1.5 for top20 portfolio) are **net of a 0.1% fee per trade with the 20% rebalance band applied**.

* **Long-Only Focus (Practicality):** The study primarily examines a long-only strategy, noting most investors have a long bias in crypto and shorting many coins is infeasible or costly. **Short positions are not utilized** in the main results. However, the authors note that a symmetric long-short version of the model was tested and also showed favorable performance in simulations (results provided in an Appendix). They limited the main analysis to long-only to avoid uncertain shorting costs and make the strategy more executable for real-world crypto market participants.

* **Low Correlation with Traditional Markets:** Crypto trend-following exhibited **very low correlation (\~0.07 on average)** with traditional trend-following programs. The paper compares the crypto trend strategy’s returns to the SG Trend Index (an index of managed futures funds) and finds correlations oscillating between slightly negative and +0.3 at most, with long stretches near zero or negative. This indicates crypto trend-following returns are largely orthogonal to those of traditional asset trend strategies, offering strong **diversification benefits** for multi-strategy portfolios. In fact, during major crypto bull runs (2017, 2020–21, late 2023) the crypto trend strategy dramatically outperformed the flat-to-modest gains of the SG Trend Index, underlining its potential to boost portfolio returns.

* **Deployment Considerations:** The paper concludes with practical guidance on implementing the strategy in real markets, including **on-chain execution** (using non-custodial wallets and decentralized exchanges for trading) versus **off-chain** (centralized exchanges). This highlights the operational nuances of crypto trading, though these aspects are beyond the scope of the backtest code. The key takeaway is that the trend model can be executed via either route, but each comes with different custody and liquidity risks that must be managed (e.g. smart contract risks on DEX vs. counterparty risk on CEX).

## 2. Data Ingestion Module (`data/`)

This module handles fetching daily OHLCV price data for Bitcoin and altcoins from various sources, falling back gracefully on alternatives. All raw data is saved as CSV files (in a git-ignored `data/raw/` directory) for caching and offline access.

* **Data Sources Supported:** We implement connectors for multiple APIs:

  * **CoinGecko API** – free and no API key required (good fallback for daily data).
  * **CryptoCompare API** – offers historical data (API key support via environment variables if needed).
  * **Yahoo Finance (yfinance)** – for convenience when a symbol is available (e.g. BTC-USD).
  * **Local CSV** – ability to read from a local file if provided (for custom datasets or testing).

* **Module Structure:**

  * `data/`

    * **`fetcher.py`** – Primary interface with a function like `get_ohlcv(symbol, source, start_date, end_date)` that orchestrates data retrieval. It reads from local CSV cache if available; otherwise it dispatches to the specified API client. If one source fails, it can optionally try a secondary source. After fetching, it saves the data to `data/raw/{symbol}.csv` (timestamp, OHLCV) for future use.
    * **`coingecko_client.py`** – Functions to call CoinGecko’s public API for historical market data. Example pseudocode:

      ```python
      from tenacity import retry, wait_exponential, stop_after_attempt
      @retry(wait=wait_exponential(min=1, max=60), stop=stop_after_attempt(5))
      def fetch_coingecko(symbol: str, days: int) -> DataFrame:
          url = f"https://api.coingecko.com/api/v3/coins/{symbol}/ohlc?vs_currency=usd&days={days}"
          resp = requests.get(url, timeout=10)
          if resp.status_code != 200:
              raise Exception("API error")
          return parse_to_dataframe(resp.json())
      ```

      This uses **Tenacity** to automatically retry transient failures with exponential backoff (e.g. doubling wait time up to 60s) and stops after a fixed number of attempts, logging each retry attempt. Similar patterns are used for other API clients.
    * **`cryptocompare_client.py`** – Fetches data from CryptoCompare’s API. If an API key is required, it is read from a config or environment variable. This client also employs `@retry` decorators for robustness. It might handle rate-limit errors by pausing appropriately.
    * **`yfinance_client.py`** – Uses the `yfinance` library to download daily OHLCV for symbols that Yahoo Finance recognizes (e.g. “BTC-USD” for Bitcoin). This is mainly for convenience and cross-checking data.
    * **`local_csv.py`** – Utility to load data from a local CSV (for users who provide their own data dumps). Ensures the DataFrame is in the same format as API outputs.
    * **`__init__.py`** – (Optional) could expose a unified `get_data` function or classes for external import.

* **Logging and Failover:** The data module logs each download attempt and outcome. If one source fails (network error or data missing), the error is caught and logged (via the common logging module described later), and the module can try an alternate source or re-use cached data. For example, if CoinGecko is unresponsive, the `fetcher` might automatically attempt CryptoCompare next. All API calls include error handling to not crash the program during data refresh.

* **Usage & Integration:** Other parts of the system (e.g. backtest engine or scripts) will call `data.fetcher.get_ohlcv("bitcoin", source="coingecko", start="2015-01-01")` to obtain a DataFrame of daily prices and volumes. The data module ensures the data is saved to disk (`data/raw/bitcoin.csv`) for reuse. The fetch functions also enforce consistent formatting (timestamp as index, numeric columns) so that downstream components can directly use the DataFrame without additional cleaning.

## 3. Backtest Engine (`backtest/`)

The backtesting engine is responsible for simulating the strategy on historical data and evaluating performance. We design it to either leverage an existing framework like **Backtrader** (for robust event-loop, accounting, etc.) or to use custom-built classes for flexibility. Key features of the engine include implementing the Donchian Channel strategy logic, volatility-based position sizing, dynamic portfolio rotation, and Monte Carlo analysis of results.

* **Module Structure:**

  * `backtest/`

    * **`strategy.py`** – Defines the **DonchianTrendStrategy** class, which encapsulates the trading rules (entry, exit, position size). If using Backtrader, this class could subclass `bt.Strategy`. In pseudo-code form:

      ```python
      class DonchianTrendStrategy:
          def __init__(self, lookbacks=[5,10,20,30,60,90,150,250,360], vol_target=0.25, max_leverage=2.0):
              self.lookbacks = lookbacks
              self.vol_target = vol_target  # 25% target vol per position
              self.max_leverage = max_leverage
              # internal state: trailing stops, signals for each lookback window
              self.trailing_stops = {n: None for n in lookbacks}
              self.positions = {n: 0 for n in lookbacks}
          def on_new_day(self, data):
              price = data['Close'][-1]
              for n in self.lookbacks:
                  window = data['Close'][-n:]
                  donchian_high = max(window); donchian_low = min(window)
                  mid = 0.5 * (donchian_high + donchian_low)
                  # Update trailing stop for n-day channel
                  if self.positions[n] != 0:
                      prev_stop = self.trailing_stops[n]
                      self.trailing_stops[n] = max(prev_stop, mid)  # (never decrease for longs)
                  # Entry signal: breakout above n-day high
                  if price >= donchian_high and self.positions[n] == 0:
                      self.positions[n] = 1  # enter long
                      self.trailing_stops[n] = mid  # set initial stop
                  # Exit signal: price falls below trailing stop
                  if self.positions[n] == 1 and price < self.trailing_stops[n]:
                      self.positions[n] = 0  # exit position
              # Determine ensemble position and size
              active_signals = sum(self.positions[n] for n in self.lookbacks)  # number of active longs
              # Average weight per active signal (equal-weight ensemble)
              w = (1.0/len(self.lookbacks)) * active_signals  
              # Volatility scaling for the asset
              vol = data['Close'][-90:].pct_change().std() * (252**0.5)
              target_w = min(self.vol_target/vol if vol > 0 else 0, self.max_leverage)
              return w * target_w  # final portfolio weight for this asset
      ```

      This pseudocode illustrates the daily signal generation: each lookback model sets its position (1 or 0) and trailing stop; the ensemble combines them (equal weight across active signals) and scales by volatility targeting. In practice, this strategy class would interface with a **Portfolio** or **Broker** to place orders (if using Backtrader, via `buy()`/`sell()` calls, adjusting size).
    * **`portfolio.py`** – Manages multi-asset portfolios and rebalancing logic. It tracks positions across multiple assets and applies the **rotational strategy**: each day, it decides which assets are eligible (e.g. top 20 by volume) and ensures capital is allocated according to strategy signals. It also enforces the **20% rebalance threshold** for volatility-target adjustments: small deviations in position size (due to price moves) under 20% are ignored to reduce churn, whereas larger deviations trigger an adjustment trade. Key responsibilities:

      * Selecting the universe of assets (e.g. at each month-end, pick top N assets by liquidity for next period).
      * Normalizing the strategy’s weight outputs to an **equal-weight across assets** (e.g. if 5 assets have non-zero signals, allocate 20% of capital to each, scaled by each asset’s volatility target weight).
      * Handling transaction costs by deducting fees when trades occur. For example, if reallocating, reduce the trade size by 0.1% of volume to simulate a 10 bps cost.
      * Keeping track of performance metrics (PnL, returns) for each asset and the total portfolio.
    * **`engine.py`** – The core backtest loop and execution engine. If using Backtrader, this sets up the Cerebro, feeds, and runs the strategy. For a custom engine, this module contains code to:

      * Load historical price data (via the data module) for each asset in the test universe.
      * Initialize the Portfolio and Strategy instances.
      * Loop through each trading day in chronological order:

        * For each asset, call `strategy.on_new_day(data[asset])` to get that asset’s desired weight.
        * Aggregate weights and enforce portfolio constraints (e.g. sum of weights ≤ 100% unless leverage allowed).
        * If any rebalance threshold is breached (asset weight drift >20%), adjust the position towards the target weight, logging the trade and cost.
        * Record daily portfolio value and other stats.
      * After the loop, calculate performance metrics (CAGR, volatility, Sharpe, Sortino, max drawdown, etc.) for the period.
      * The engine should support **parameterization** to easily run different scenarios (e.g. different lookback sets, different cost levels, etc.).
    * **`analysis.py`** (optional, or part of `engine.py`) – Utilities for analyzing backtest results, including generating **Monte Carlo performance bands**. To account for uncertainty in outcomes, we can simulate random variations of the strategy’s performance:

      * For example, run the strategy multiple times on the historical data but with slight random noise added to returns or with randomized start dates. Alternatively, bootstrap the daily returns of the strategy and generate an ensemble of equity curves.
      * Compute the envelope of these simulated equity curves to get **±10% performance bands**. (For instance, show the range in which 80% of simulated outcomes fall, and highlight ±10% deviation from the actual CAGR as a simple margin.)
      * This helps visualize best-case and worst-case trajectories around the realized performance. The code could output these bands for plotting (e.g. an area chart around the actual equity curve).
    * **`indicators.py`** (optional) – Functions for technical indicator calculations (e.g. a dedicated `donchian(highs, lows, n)` that returns the bands, or volatility calculators). This separates analytical logic from the Strategy class.

* **Monte Carlo & Stress Testing:** (As noted above in `analysis.py`) the engine can produce Monte Carlo scenarios to assess strategy robustness. For example, after a backtest run, the system can generate 100 random trials where each trial modifies daily returns by a random factor within ±10% or shuffles the sequence of some trades. The output could be an array of CAGRs or Sharpe ratios, from which we derive a **confidence band** (e.g. the strategy might have a base CAGR of 18%, with a ±10% band indicating most outcomes fall between 16.2% and 19.8% CAGR given small perturbations). This feature is useful for understanding sensitivity to luck or slight market changes.

* **Backtester Output:** The engine will produce:

  * A detailed **performance report** (which can be written to console or a file): including CAGR, annualized volatility, Sharpe, Sortino, max drawdown, MAR (CAGR/MDD), alpha and beta vs Bitcoin, and number of trades, win rate, average trade profit, etc. (many of these metrics were used in the paper’s tables).
  * Optionally, equity curve time series for the portfolio and for a benchmark (e.g. buy-and-hold BTC) to compare visually.
  * If enabled, Monte Carlo simulation results (e.g. plot data for performance bands).

* **Integration:** The backtest engine is configured via the config files (see Config section). For example, the `config.yml` may specify which assets to include, the lookback periods to use, initial capital, transaction fee, rebalance threshold, and whether to use backtrader or the custom engine. The engine module will read these settings and set up the run accordingly. Results can be printed to console, saved as JSON/CSV, and plotted (if run in an interactive environment).

## 4. Project Structure and Documentation

Below is an outline of the repository’s file structure, along with a description of each component and how they interrelate:

* **Root Directory** (project top-level):

  * **`README.md`** – *Project overview and usage guide.* Provides a high-level description of the strategy and repository. It explains how to set up the environment, how to fetch data and run backtests, and gives example commands. It also outlines the directory structure and directs the user to further documentation (like CODEX and TODO). Example sections: *Introduction, Installation, Quickstart, Repository Layout, Example Usage*. (This is the first point of contact for new users.)
  * **`TODO.md`** – *Development task list.* An exhaustive list of planned improvements and issues, written as actionable, atomic items (suitable for an AI agent or contributors to pick up). Each TODO entry is a bullet or checkbox item describing the task, expected behavior, edge cases, and tests. For example: *“Add `lookback` parameter to Donchian strategy class – allow easy adjustment of channel lengths via config. **Expected:** Strategy reads list from config; **Edge cases:** ensure no duplicate periods; **Tests:** verify that modifying the list changes strategy signals.”*  This file guides contributors on next steps and ensures the project’s future work is well-defined.
  * **`CODEX.md`** – *Technical design and API reference.* This document acts as a developer guide, detailing the architecture of the code. It includes class diagrams or descriptions (how `Strategy`, `Portfolio`, and data modules interact), API method signatures (what functions are available and their parameters), and pseudocode for core algorithms (e.g. the daily backtest loop and signal generation logic). For instance, CODEX.md will show the pseudocode for the Donchian Channel breakout logic and how volatility targeting is implemented (similar to the snippets in the strategy section above), as well as how the Monte Carlo analysis is conducted. The purpose is to help new developers quickly grasp how the system works internally.
  * **`CONTRIBUTING.md`** – *Contribution guidelines.* Explains how to fork the repo, create branches, run tests, and submit pull requests. It includes coding style conventions (linting rules, type hints, docstrings), the requirement to write unit tests for new features, and how to update the CHANGELOG. This ensures consistency in contributions.
  * **`CODE_OF_CONDUCT.md`** – *Community standards.* Standard code of conduct for contributors (e.g. adapted from Contributor Covenant) outlining expected behavior in communications and project participation.
  * **`.gitignore`** – Configured to ignore sensitive or large files: e.g. `data/raw/*.csv` (fetched price data), `config.yml`, `.env`, any secrets or outputs. This keeps the repo lean and secure.
  * **`requirements.txt`** – List of Python dependencies (exact versions or constraints). Likely includes: `pandas`, `numpy`, `yfinance`, `requests` (or a specific HTTP client), `tenacity` for retries, `backtrader` (if used), and possibly plotting libraries for analysis. This file enables users to install all requirements with `pip`.
  * **`setup.py`** or **`pyproject.toml`** (optional) – If we package the project, these define the package info. Not strictly required if running as scripts, but could be included for completeness.

* **Source Code Packages:**

  * **`data/`** – *(Data ingestion package as described in Section 2.)* This contains the modules for fetching and caching data.

    * Files: `fetcher.py`, `coingecko_client.py`, `cryptocompare_client.py`, `yfinance_client.py`, `local_csv.py`, and possibly `__init__.py`.
    * **Interdependencies:** The data module might use the global config (from `config.yml`) to decide which source to prioritize or to retrieve API keys. Other modules (like backtest) call functions from `data.fetcher` to get the historical prices. The data files produced are saved in `data/raw/` subdirectory.
  * **`backtest/`** – *(Backtesting engine package, see Section 3.)* Contains the logic to simulate the strategy.

    * Files: `strategy.py`, `portfolio.py`, `engine.py`, `analysis.py` (or `monte_carlo.py`), `indicators.py`, and `__init__.py`.
    * **Interdependencies:** `strategy.py` might import indicator functions or use numpy/pandas for rolling window calcs. It likely reads parameters from a central config (e.g. list of lookbacks, risk target). `engine.py` uses `data/` to load time series, uses `strategy.Portfolio` to manage positions, and logs via the logging module. The backtest package will also rely on config settings (like transaction cost level, rebalance threshold from `config.yml`). Results produced by `engine.py` (e.g. performance metrics, equity curves) might be passed to `analysis.py` for further processing (like Monte Carlo).
  * **`logging/`** – *(Telemetry and logging utilities.)* This folder centralizes logging configuration so that all parts of the project use a consistent logging approach.

    * **`logging_conf.yml`** (or `.json`) – Configuration file for Python’s logging module. Defines log format (e.g. JSON structure with timestamp, module, level, message), log levels, and handlers (console, file). For instance, debug logs can go to a rotating file `logs/debug.log` and important info/errors to console.
    * **`telemetry.py`** – Sets up custom loggers and metrics collection. It might define a logger for data fetch retries, another for trade executions, etc., each with context-rich output. Also, this module could integrate **Prometheus metrics** export: e.g. maintain counters for number of API calls made, number of trades executed, total PnL, etc., which can be exposed if this were running as a service. In backtesting context, it could simply accumulate these metrics and perhaps output them at the end or save to a JSON.
    * **Interdependencies:** All other modules import the logging/telemetry module to emit logs. For example, the data clients log any retries or failures, the strategy logs every entry/exit signal, and the engine logs summary performance. By configuring this in one place, we ensure uniform log formatting. If integrated with a monitoring system, this module would handle the details.
  * **`config/`** – *(Configuration files and templates.)* Handles configuration management so that sensitive or environment-specific info is not hard-coded.

    * **`config.sample.yml`** – A sample configuration file with all possible settings and documentation comments. For example:

      ```yaml
      data_source: "coingecko"        # default data source [coingecko|cryptocompare|yfinance]
      assets: ["BTC", "ETH", "BNB", ...]   # list of assets to include (e.g. top 20 by vol)
      lookbacks: [5, 10, 20, 30, 60, 90, 150, 250, 360]
      target_volatility: 0.25        # 25% annual target vol
      rebalance_threshold: 0.2       # 20% drift threshold for rebalancing
      transaction_cost: 0.001        # 0.1% per trade
      use_backtrader: false          # whether to use Backtrader engine
      monte_carlo_trials: 100        # number of simulations for performance bands
      ```

      Users can copy this to `config.yml` and adjust as needed. The app will load `config.yml` (which is git-ignored).
    * **`config.yml`** – Actual configuration in use (git-ignored for privacy). This may also include API keys if needed (e.g. `cryptocompare_api_key: "XYZ"`), or those can be supplied via environment.
    * **`.env`** – Environment variable definitions (git-ignored). Could store secrets like `CRYPTOCOMPARE_API_KEY=...` which the code reads at runtime. This separation allows sharing the sample config publicly while keeping credentials private.
    * **Interdependencies:** The backtest engine and data modules read from the config to get parameters (we will have a small config loader that parses YAML at startup and makes settings available as a singleton or passed object). For instance, `data.fetcher` might check `config["data_source"]` to decide which API to call, and `backtest.engine` will use `config["transaction_cost"]`, etc.

* **DevOps & CI:**

  * **`.github/`** – GitHub configuration for continuous integration and community health.

    * **`workflows/`** – CI/CD pipeline definitions:

      * **`ci.yml`** – Continuous Integration workflow. This uses GitHub Actions to run tests and linters on each push/PR. It may have jobs like:

        * *Install & Lint:* set up Python, install requirements, run a linter (e.g. flake8 or pylint) and a type checker (mypy).
        * *Unit Tests:* run the test suite (perhaps with `pytest`) on a matrix of Python versions.
        * *Coverage:* measure test coverage; if coverage < 50%, the job can fail (ensuring at least 50% coverage as specified).
        * Jobs can be conditional (for example, only run certain heavy tests on main branch or when a special label is set on the PR).
      * **`docker.yml`** – Docker build and publish workflow. This defines a multi-stage Docker build (e.g. a base stage with Python and requirements, then copy code, then a smaller runtime image). The workflow might be triggered on new tags or manual dispatch, and is configured to be easily turned off or on (for instance, it could check a repository secret or variable like `BUILD_DOCKER=true`). This avoids using CI minutes by default on every push. When enabled, it builds the Docker image of the project and (optionally) pushes to a registry.
    * **`ISSUE_TEMPLATE/`** – Templates for GitHub issues:

      * **`bug_report.md`**, **`feature_request.md`** – Markdown templates that prefill issue forms with headings (e.g. Steps to Reproduce, Expected vs Actual behavior for bugs; or Use Case, Proposed Solution for features). This ensures contributors provide necessary info.
    * **`PULL_REQUEST_TEMPLATE.md`** – A template that every PR description will start with. It can include a checklist (tests added, docs updated, lint passed) and prompt for a summary of changes. This helps maintain quality and consistency for PRs.

* **Testing:**

  * **`tests/`** – (If included) Unit test modules for each component. E.g. `test_data_fetcher.py` to simulate API calls (perhaps using VCR or stub data), `test_strategy.py` for verifying signal logic on known price series, etc. The CI will run these. Each new feature in TODO is expected to come with a test in this directory.

All components are structured to maximize modularity. For example, you could swap out the data source (say, use CSV instead of API) by changing the config, without modifying the backtest code. The logging and config are centralized to avoid scattering those concerns throughout the code. The **interdependencies** are managed via clear interfaces: the strategy doesn’t need to know *where* data comes from, just that it receives a DataFrame; the engine doesn’t need to know *how* signals are generated internally, just that it can call `strategy` and get target weights. This separation of concerns makes the system robust and easy to extend (for instance, adding a new indicator or a new data source in the future).

## 5. Example Usage

Below is a minimal example demonstrating how one might use the data and backtest modules to fetch data for a single asset and run a simple Donchian breakout test, then output the generated signal:

```python
# Minimal example: fetch BTC data and run a one-day signal check for a 20-day Donchian breakout.

from config import config  # assume config is a module that loads config.yml
from data.fetcher import get_ohlcv
from backtest.strategy import DonchianTrendStrategy

# 1. Load historical data for one ticker (e.g., Bitcoin).
btc_df = get_ohlcv(symbol="bitcoin", source="coingecko", start_date="2020-01-01", end_date="2021-01-01")
print(f"Fetched {len(btc_df)} days of data for BTC-USD")

# 2. Initialize the trading strategy (single Donchian channel for simplicity).
strategy = DonchianTrendStrategy(lookbacks=[20], vol_target=0.25)  # using only a 20-day channel

# 3. Simulate a simple backtest loop (here just checking the latest signal).
for day in range(len(btc_df)):
    today_data = btc_df.iloc[:day+1]  # data up to current day
    strategy_weight = strategy.on_new_day(today_data)
# After loop, strategy.positions and trailing_stops contain the final state.

# 4. Output the final signal for today (e.g., 1 for long, 0 for no position).
latest_price = btc_df['Close'].iloc[-1]
position_active = strategy.positions[20]  # for 20-day channel
if position_active:
    print(f"Signal: BUY (price broke out, currently long). Trailing stop at {strategy.trailing_stops[20]:.2f}")
else:
    # If not long, it means either no breakout occurred or the position was exited.
    last_high = btc_df['Close'].iloc[-20:].max()
    if latest_price >= last_high:
        print("Signal: BUY (just triggered a breakout to new 20-day high today!)")
    else:
        print("Signal: HOLD/NO POSITION (no breakout signal)")
```

In this snippet:

* We fetch one year of Bitcoin daily prices.
* We create a DonchianTrendStrategy focusing only on a 20-day channel for clarity.
* We loop through the days (as a lightweight backtest) and update the strategy each day.
* Finally, we check if the strategy ended with a long position and print the appropriate signal and the trailing stop. In a real run, the backtest engine would handle looping and would record all trades and PnL, but here we demonstrate usage in a simplified manner.

**Output (example):** The printout might be:

```text
Fetched 366 days of data for BTC-USD  
Signal: BUY (just triggered a breakout to new 20-day high today!)
```

This indicates that on the last day of the dataset, the price exceeded the prior 20-day high, generating a buy signal. The scaffolding above can be expanded to run full backtests and analyze performance using the modules we designed.
