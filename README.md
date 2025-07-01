# Crypto Trend-Following System

A Python-based framework to implement and validate the methods from  
**“Catching Crypto Trends: A Tactical Approach for Bitcoin and Altcoins”**  
(Zarattini et al., 2025) using daily OHLCV data.

---

## 🚀 Features

- **Claims Extraction**  
  Bulleted summary of every key claim, methodology step, and performance metric from the paper, with exact page/section citations.
- **Data Ingestion**  
  – Fetch daily OHLCV from CoinGecko, CryptoCompare, yfinance  
  – Cache raw CSVs in `data/` with built-in validation  
  – Retry logic with Tenacity (exponential backoff)  
- **Backtest Engine**  
  – Ensemble Donchian-channel breakout strategy (5–360 day lookbacks)  
  – Volatility-targeted sizing (25% annual target, 2× max leverage)  
  – Rotational portfolio rebalancing with 20% drift threshold  
  – Monte Carlo ±10% performance bands  
- **Project Scaffolding & Governance**  
  – `README.md`, exhaustive `TODO.md`, and `CODEX.md` design docs  
  – `.github/` CI workflows (tests, lint, coverage ≥ 50%, Docker build toggle)  
  – `CONTRIBUTING.md`, `CODE_OF_CONDUCT.md`, issue & PR templates  
- **Logging & Telemetry**  
  – Structured JSON logs & Prometheus metrics  
- **Config Management**  
  – YAML/JSON config templates + `.env` support  
- **Docker**  
  – Multi-stage Dockerfile for local and CI use

---

## 📂 Repository Layout

```text
/
├── README.md
├── TODO.md
├── CODEX.md
├── CONTRIBUTING.md
├── CODE_OF_CONDUCT.md
├── requirements.txt
├── .gitignore
│
├── src/                     # All source code
│   ├── data/                # Data ingestion package
│   │   ├── fetcher.py
│   │   ├── coingecko_client.py
│   │   ├── cryptocompare_client.py
│   │   ├── yfinance_client.py
│   │   └── local_csv.py
│   │
│   ├── backtest/            # Backtesting engine
│   │   ├── strategy.py
│   │   ├── portfolio.py
│   │   ├── engine.py
│   │   ├── analysis.py
│   │   └── indicators.py
│   │
│   ├── logging/             # Telemetry & logging setup
│   │   ├── logging_conf.yml
│   │   └── telemetry.py
│   │
│   └── config/              # Configuration templates
│       ├── config.sample.yml
│       ├── config.yml       # (git-ignored)
│       └── .env             # (git-ignored)
│
├── data/                    # Cached flat-file time series & scripts
│   ├── BTC-COINGECKO-OHLCV-2015-01-01-2025-03-19.csv
│   ├── ETH-YFINANCE-OHLCV-2015-01-01-2025-03-19.csv
│   └── scripts/
│       └── validate_data.py # one-off data validation & formatting
│
└── .github/
    └── workflows/
        ├── ci.yml
        └── docker.yml
    └── ISSUE_TEMPLATE/
        ├── bug_report.md
        └── feature_request.md
    └── PULL_REQUEST_TEMPLATE.md
```

---

## 🔧 Prerequisites

- Python 3.10 – 3.12  
- Docker (for local containerized runs)  
- [pipenv](https://pipenv.pypa.io/) or `venv` + `pip`  

---

## ⚙️ Installation

1. **Clone the repo**  
   ```bash
   git clone https://github.com/your-org/crypto-trend-system.git
   cd crypto-trend-system
   ```

2. **Create & activate virtual environment**  
   ```bash
   pipenv install --dev
   pipenv shell
   ```

3. **Install dependencies**  
   ```bash
   pip install -r requirements.txt
   ```

4. **Copy & configure**  
   ```bash
   cp src/config/config.sample.yml src/config/config.yml
   cp src/config/.env.example src/config/.env
   # Edit src/config/config.yml & .env with your settings and API keys
   ```

---

## 🗄️ Data Folder & File Naming

- **Flat CSVs only**—no code.  
- Naming convention:  

  ```text
  {TICKER}-{DATASOURCE}-OHLCV-{STARTDATE}-{ENDDATE}.csv
  ```

  e.g.  

  ```text
  BTC-COINGECKO-OHLCV-2015-01-01-2025-03-19.csv
  ```

- **Header format** (all files):  

  ```csv
  Timestamp,Open,High,Low,Close,Volume,NotValid
  ```

  - `NotValid`: initially `False`; set to `True` by `validate_data.py` if retrieval issues are detected.

- **Validation script**:  
  - `data/scripts/validate_data.py`—checks each CSV for missing rows, outliers, or API errors and updates `NotValid` flags.

---

## 🚀 Quickstart

### Fetch & Cache Data (one-off)

```bash
python src/data/scripts/fetch_all.py
# populates data/ with flat CSVs named per convention
```

### Validate Data

```bash
python src/data/scripts/validate_data.py
# marks rows/files as NotValid if issues found
```

### Run a Backtest

```bash
python src/backtest/engine.py \
  --config src/config/config.yml \
  --start 2015-01-01 \
  --end   2025-03-19
```

Outputs a performance report to console and saves results to `results/`.

---

## 🐳 Docker

Build and run locally:

```bash
docker build -t crypto-trend-system .
docker run --rm -v "$(pwd)/data:/app/data" crypto-trend-system \
  python src/backtest/engine.py --config src/config/config.yml
```

---

## 🤝 Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for guidelines on issues, PRs, and code style.

---

## 📄 License

This project is licensed under MIT License — see [LICENSE](LICENSE) for details.