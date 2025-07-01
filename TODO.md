# TODO

A comprehensive, phased list of atomic tasks for building the Crypto Trend–Following System.  
Each item is formatted for easy copy/paste into a GitHub Issue and tagged with:

- **Phase**: Iteration number  
- **Actor**: Human / Codex (agentic AI)  
- **Tags**: One or more of {Setup, Data, Backtest, CoreFeature, Enhancement, Test, Config, DevOps, Documentation, Bug}

---

## Phase 1: Project Initialization & Scaffolding

1. **Initialize repository skeleton**  
   - **Phase:** 1  
   - **Actor:** Codex  
   - **Tags:** Setup, CoreFeature  
   - **Description:**  
     - Create the following top-level structure:  

       ```text
       /
       ├── src/
       ├── data/
       │   └── scripts/
       ├── tests/
       ├── logging/
       ├── config/
       ├── .github/
       ├── README.md
       ├── TODO.md
       ├── CODEX.md
       ├── CONTRIBUTING.md
       ├── CODE_OF_CONDUCT.md
       ├── requirements.txt
       └── .gitignore
       ```  
     - Add placeholder Dockerfile.  
     - Ensure `.gitignore` excludes `data/*.csv`, `src/config/config.yml`, `.env`, `/logging/*`.

2. **Add sample configuration files**  
   - **Phase:** 1  
   - **Actor:** Codex  
   - **Tags:** Config, CoreFeature  
   - **Description:**  
     - Create `src/config/config.sample.yml` with all required keys (data_source, assets, lookbacks, vol_target, rebalance_threshold, transaction_cost, use_backtrader, monte_carlo_trials).  
     - Create `src/config/.env.example` listing environment variables (e.g., CRYPTOCOMPARE_API_KEY).

3. **Implement configuration loader**  
   - **Phase:** 1  
   - **Actor:** Codex  
   - **Tags:** CoreFeature, Config  
   - **Description:**  
     - In `src/config/config_loader.py`, write a singleton that:  
       1. Reads `config/config.yml` (fallback to sample).  
       2. Loads `.env` via python-dotenv.  
       3. Exposes settings as a Python object/dict.

4. **Set up logging scaffolding**  
   - **Phase:** 1  
   - **Actor:** Codex  
   - **Tags:** CoreFeature, DevOps  
   - **Description:**  
     - Add `logging/logging_conf.yml` defining JSON formatter, console & file handlers.  
     - Add `src/logging/telemetry.py` to initialize Python `logging` from config and expose a `get_logger(name)` helper.

---

## Phase 2: Data Ingestion Module

5. **Implement data fetcher interface**  
   - **Phase:** 2  
   - **Actor:** Codex  
   - **Tags:** Data, CoreFeature  
   - **Description:**  
     - Create `src/data/fetcher.py` with `get_ohlcv(symbol, source, start_date, end_date)` that:  
       1. Checks for cached `data/{symbol}-{source}-OHLCV-{start}-{end}.csv`.  
       2. If missing or stale, dispatches to the appropriate client.  
       3. Saves raw CSV to `data/` following file-naming convention.  
       4. Returns a Pandas DataFrame.

6. **Implement CoinGecko client**  
   - **Phase:** 2  
   - **Actor:** Codex  
   - **Tags:** Data, CoreFeature  
   - **Description:**  
     - In `src/data/coingecko_client.py`, implement `fetch_coingecko(symbol, start_date, end_date)`.  
     - Use `@tenacity.retry` with exponential backoff.  
     - Parse API response to DataFrame with columns `[Timestamp, Open, High, Low, Close, Volume, NotValid]` (NotValid defaults to False).

7. **Implement CryptoCompare client**  
   - **Phase:** 2  
   - **Actor:** Codex  
   - **Tags:** Data, CoreFeature  
   - **Description:**  
     - In `src/data/cryptocompare_client.py`, implement `fetch_cryptocompare(symbol, start_date, end_date)`.  
     - Read API key from environment.  
     - Apply retry logic and format DataFrame as above.

8. **Implement yfinance client**  
   - **Phase:** 2  
   - **Actor:** Codex  
   - **Tags:** Data, CoreFeature  
   - **Description:**  
     - In `src/data/yfinance_client.py`, implement `fetch_yfinance(symbol, start_date, end_date)` using the `yfinance` library.  
     - Format DataFrame with required header and default NotValid=False.

9. **Implement local CSV loader**  
   - **Phase:** 2  
   - **Actor:** Codex  
   - **Tags:** Data, Enhancement  
   - **Description:**  
     - In `src/data/local_csv.py`, write `load_local_csv(path)` to read a CSV, validate header, and return DataFrame.

10. **Create data validation script**  
    - **Phase:** 2  
    - **Actor:** Codex  
    - **Tags:** Data, Test  
    - **Description:**  
      - In `data/scripts/validate_data.py`, iterate over all `data/*.csv`:  
        - Check for missing dates or NaNs.  
        - If issues found, set `NotValid=True` for affected rows and rewrite file.  
        - Log summary of validations.

11. **Write unit tests for data module**  
    - **Phase:** 2  
    - **Actor:** Codex  
    - **Tags:** Test, Data  
    - **Description:**  
      - Under `tests/`, add `test_coingecko_client.py`, `test_cryptocompare_client.py`, `test_yfinance_client.py`, and `test_fetcher.py`.  
      - Use small sample JSON/CSV fixtures and monkeypatch HTTP responses.  
      - Assert DataFrame shape, columns, and NotValid default.

---

## Phase 3: Backtest Engine & Core Strategy

12. **Implement DonchianTrendStrategy class**  
    - **Phase:** 3  
    - **Actor:** Codex  
    - **Tags:** Backtest, CoreFeature  
    - **Description:**  
      - In `src/backtest/strategy.py`, implement `DonchianTrendStrategy(lookbacks, vol_target, max_leverage)`.  
      - Follow pseudocode in CODEX.md: entry/exit rules, trailing stops, ensemble weight, volatility sizing.

13. **Implement Portfolio manager**  
    - **Phase:** 3  
    - **Actor:** Codex  
    - **Tags:** Backtest, CoreFeature  
    - **Description:**  
      - In `src/backtest/portfolio.py`, implement `Portfolio` class to:  
        1. Select top-20 assets by volume from config.  
        2. Allocate capital based on strategy weights.  
        3. Enforce 20% rebalance threshold and transaction costs.  
        4. Track PnL and positions.

14. **Implement backtest engine loop**  
    - **Phase:** 3  
    - **Actor:** Codex  
    - **Tags:** Backtest, CoreFeature  
    - **Description:**  
      - In `src/backtest/engine.py`, write the main loop:  
        1. Load OHLCV for each asset via data.fetcher.  
        2. Instantiate `Portfolio` and `DonchianTrendStrategy`.  
        3. Iterate trading days: update signals, rebalance if threshold breached, log trades.  
        4. Record daily portfolio value.

15. **Implement technical indicators utilities**  
    - **Phase:** 3  
    - **Actor:** Codex  
    - **Tags:** Backtest, Enhancement  
    - **Description:**  
      - In `src/backtest/indicators.py`, add functions: `donchian_channel(highs, lows, n)` and `rolling_volatility(returns, window)`.

16. **Write backtest unit tests**  
    - **Phase:** 3  
    - **Actor:** Codex  
    - **Tags:** Test, Backtest  
    - **Description:**  
      - Under `tests/`, add `test_strategy.py`, `test_portfolio.py`, `test_engine.py`.  
      - Use synthetic price series to verify correct signal generation, position sizing, and portfolio returns.

---

## Phase 4: Performance Analysis & Monte Carlo

17. **Implement Monte Carlo simulation**  
    - **Phase:** 4  
    - **Actor:** Codex  
    - **Tags:** Backtest, Enhancement  
    - **Description:**  
      - In `src/backtest/analysis.py`, add `run_monte_carlo(equity_curve, trials, error_margin)` that:  
        - Bootstraps or perturbs returns.  
        - Generates performance bands at ±10%.  
        - Returns band data for plotting.

18. **Write Monte Carlo tests**  
    - **Phase:** 4  
    - **Actor:** Codex  
    - **Tags:** Test, Backtest  
    - **Description:**  
      - Add `tests/test_analysis.py` to verify simulation outputs consistent shapes and band boundaries.

19. **Generate summary performance report**  
    - **Phase:** 4  
    - **Actor:** Codex  
    - **Tags:** Backtest, CoreFeature  
    - **Description:**  
      - Extend `engine.py` or `analysis.py` to compute and print CAGR, volatility, Sharpe, Sortino, max drawdown, alpha vs BTC, and summary trade stats.  
      - Format output as console table or JSON file.

---

## Phase 5: Live Signal Generation

20. **Implement daily signal generator**  
    - **Phase:** 5  
    - **Actor:** Codex  
    - **Tags:** CoreFeature, Backtest  
    - **Description:**  
      - Create `src/signals/daily_signal.py` that:  
        1. Loads latest OHLCV for each asset.  
        2. Applies `DonchianTrendStrategy` to generate today’s weights/signals.  
        3. Writes signals to `signals/YYYY-MM-DD.json`.

21. **Schedule signal job**  
    - **Phase:** 5  
    - **Actor:** Human  
    - **Tags:** CoreFeature, DevOps  
    - **Description:**  
      - Document how to run `daily_signal.py` daily (e.g. cron job inside Docker container).  
      - Provide example `docker-compose.yml` snippet.

---

## Phase 6: CI/CD & Docker

22. **Create CI workflow**  
    - **Phase:** 6  
    - **Actor:** Codex  
    - **Tags:** DevOps, Test  
    - **Description:**  
      - In `.github/workflows/ci.yml`, define jobs to:  
        1. Set up Python.  
        2. Install dependencies.  
        3. Run lint (`flake8`), type check (`mypy`).  
        4. Run tests with coverage; fail if <50%.

23. **Create Docker build workflow**  
    - **Phase:** 6  
    - **Actor:** Codex  
    - **Tags:** DevOps, Enhancement  
    - **Description:**  
      - In `.github/workflows/docker.yml`, implement multi-stage Docker build.  
      - Guard with `if: env.BUILD_DOCKER == 'true'` to respect free-tier limits.

24. **Write Dockerfile with multi-stage build**  
    - **Phase:** 6  
    - **Actor:** Codex  
    - **Tags:** DevOps, Enhancement  
    - **Description:**  
      - In project root, add Dockerfile:  
        - **Stage 1:** Install dependencies.  
        - **Stage 2:** Copy source, set entrypoint to run backtest or signal script.

25. **Add issue & PR templates**  
    - **Phase:** 6  
    - **Actor:** Codex  
    - **Tags:** Documentation, DevOps  
    - **Description:**  
      - In `.github/ISSUE_TEMPLATE/bug_report.md` and `feature_request.md`.  
      - Add `PULL_REQUEST_TEMPLATE.md` with checklist (lint, tests, docs).

---

## Phase 7: Documentation & Governance

26. **Populate README.md**  
    - **Phase:** 7  
    - **Actor:** Codex  
    - **Tags:** Documentation  
    - **Description:**  
      - Use scaffold details to write sections: Introduction, Features, Installation, Data, Usage, Docker, Contributing, License.

27. **Finalize CODEX.md**  
    - **Phase:** 7  
    - **Actor:** Human  
    - **Tags:** Documentation  
    - **Description:**  
      - Review and adjust `CODEX.md` to ensure all agent instructions are accurate and up to date with code structure.

28. **Finalize CONTRIBUTING.md**  
    - **Phase:** 7  
    - **Actor:** Human  
    - **Tags:** Documentation  
    - **Description:**  
      - Confirm guidelines match implemented workflows and hooks.

29. **Publish CODE_OF_CONDUCT.md**  
    - **Phase:** 7  
    - **Actor:** Human  
    - **Tags:** Documentation  
    - **Description:**  
      - Add standard Contributor Covenant text and adjust contact information.

---

## Phase 8: Testing & Quality Assurance

30. **Configure pre-commit hooks**  
    - **Phase:** 8  
    - **Actor:** Codex  
    - **Tags:** DevOps, Test  
    - **Description:**  
      - Add `.pre-commit-config.yaml` to enforce `black`, `isort`, and `flake8` locally before each commit.

31. **Add MyPy configuration**  
    - **Phase:** 8  
    - **Actor:** Codex  
    - **Tags:** DevOps, Test  
    - **Description:**  
      - Create `mypy.ini` with strict settings and integrate into CI.

32. **Verify 50% coverage enforcement**  
    - **Phase:** 8  
    - **Actor:** Codex  
    - **Tags:** Test  
    - **Description:**  
      - Ensure CI fails when `pytest --cov-fail-under=50` is violated.

---

## Phase 9: Enhancements & Data Governance

33. **Integrate DVC or Git LFS for data versioning**  
    - **Phase:** 9  
    - **Actor:** Codex  
    - **Tags:** Data, Enhancement  
    - **Description:**  
      - Configure DVC or Git LFS to track `data/*.csv` and store metadata in repo.

34. **Add Prometheus metrics stub**  
    - **Phase:** 9  
    - **Actor:** Codex  
    - **Tags:** Enhancement, DevOps  
    - **Description:**  
      - In `src/logging/telemetry.py`, add Prometheus client counters for API calls, trades executed, and backtest runs.

35. **Create sample data-fetch script**  
    - **Phase:** 9  
    - **Actor:** Codex  
    - **Tags:** Data, Enhancement  
    - **Description:**  
      - Add `data/scripts/fetch_all.py` that reads `config.yml`, loops through `assets` list, calls `fetcher.get_ohlcv`, and stores all CSVs.
