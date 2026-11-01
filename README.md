# PegGuard AI — The Stablecoin Observatory

> **Trust, measured in basis points.**  
> A stablecoin monitoring dashboard that tracks market data, measures deviation from the US dollar peg, and highlights price-based risk signals.

PegGuard AI is a research-oriented dashboard for monitoring selected dollar-pegged crypto assets. It currently tracks **Tether (USDT), USD Coin (USDC), and Dai (DAI)** using market data from CoinGecko, then applies a deterministic, rule-based risk engine to each asset.

## Overview

Stablecoins are designed to maintain a stable value, commonly around **$1.00 USD**. PegGuard focuses on the small price movements that can indicate deviation from that target.

The dashboard presents:

- Current asset prices and market metrics.
- Price deviation from the $1.00 target, expressed in basis points.
- A price-based risk score and risk level.
- Signals explaining which price conditions triggered a warning.
- A market overview that refreshes automatically every 30 seconds.
- A manual refresh control and API connection status.

## Monitored assets

| Asset | Symbol | Market data ID |
|---|---|---|
| Tether | USDT | `tether` |
| USD Coin | USDC | `usd-coin` |
| Dai | DAI | `dai` |

## How the risk engine works

The current backend uses a transparent heuristic model (`heuristic-v1`). It does **not** use an ML model or an LLM to generate the numeric score.

### 1. Peg deviation

The absolute distance from the $1.00 target is calculated in basis points:

```text
deviation_bps = abs(price_usd - 1.00) × 10,000
peg_risk = min(deviation_bps / 50, 1) × 100
```

### 2. 24-hour movement

When 24-hour percentage-change data is available:

```text
movement_risk = min(abs(change_24h_pct), 1) × 100
risk_score = round(0.8 × peg_risk + 0.2 × movement_risk)
```

If 24-hour change data is unavailable, the score is based on peg risk alone. If price data is unavailable, the score and risk level are returned as unknown.

### 3. Risk levels

| Score | Risk level |
|---:|---|
| 0–34 | Low |
| 35–69 | Moderate |
| 70–100 | High |
| Unavailable | Unknown |

The API also returns explanatory signals, such as a detected price deviation or a large 24-hour price movement.

**Important:** This is an experimental, price-based heuristic—not a complete assessment of stablecoin safety, reserves, liquidity, issuer risk, or redemption risk. It is not financial advice.

## Tech stack

### Frontend
- Next.js 16
- React 19
- TypeScript
- Tailwind CSS 4
- Recharts
- Lucide React

### Backend
- Python
- FastAPI
- HTTPX
- CoinGecko API

## Project structure

```text
PegGuard-AI/
├── backend/
│   └── app/
│       ├── __init__.py
│       ├── config.py
│       ├── main.py
│       └── services/
│           ├── __init__.py
│           ├── market_data.py
│           └── risk_engine.py
└── frontend/
    ├── public/
    ├── src/
    │   └── app/
    │       ├── favicon.ico
    │       ├── globals.css
    │       ├── layout.tsx
    │       └── page.tsx
    ├── package.json
    └── next.config.ts
```

## Getting started

You will need:

- Python 3.10 or later
- Node.js 20.9 or later
- npm
- Internet access for CoinGecko market data

### 1. Clone the repository

```bash
git clone https://github.com/<your-username>/<your-repository>.git
cd <your-repository>
```

### 2. Start the backend

Open a terminal in the `backend` directory:

```bash
cd backend
```

Create and activate a virtual environment.

**Windows (PowerShell):**

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

**macOS / Linux:**

```bash
python3 -m venv .venv
source .venv/bin/activate
```

Install the backend dependencies:

```bash
pip install fastapi "uvicorn[standard]" httpx
```

Optionally, set a CoinGecko Demo API key if your API access requires one.

**Windows (PowerShell):**

```powershell
$env:COINGECKO_API_KEY="your_api_key"
```

**macOS / Linux:**

```bash
export COINGECKO_API_KEY="your_api_key"
```

Run the API from inside the `backend` directory:

```bash
uvicorn app.main:app --reload --host 127.0.0.1 --port 8000
```

The backend will be available at:

- API: `http://127.0.0.1:8000`
- Health check: `http://127.0.0.1:8000/health`
- Interactive API docs: `http://127.0.0.1:8000/docs`

### 3. Start the frontend

Open a second terminal in the `frontend` directory:

```bash
cd frontend
npm install
```

Create a `.env.local` file in `frontend/` if you want to configure a different backend URL:

```env
NEXT_PUBLIC_API_URL=http://127.0.0.1:8000
```

Start the development server:

```bash
npm run dev
```

Open **http://localhost:3000** in your browser.

The frontend expects the backend to be running on port `8000`.

## API reference

### `GET /health`

Returns the backend service health status.

Example response:

```json
{
  "status": "healthy",
  "service": "PegGuard AI"
}
```

### `GET /api/v1/stablecoins`

Fetches the latest available market data for the monitored stablecoins and adds a risk assessment to each asset.

The response includes:

- `observed_at` — timestamp of the API observation.
- `count` — number of returned assets.
- `assets` — market metrics and risk assessment for each asset.
- `model_version` — currently `heuristic-v1`.

## Configuration

| Variable | Purpose | Default |
|---|---|---|
| `COINGECKO_API_KEY` | Optional CoinGecko Demo API key | Empty |
| `COINGECKO_BASE_URL` | CoinGecko API base URL | `https://api.coingecko.com/api/v3` |
| `NEXT_PUBLIC_API_URL` | Backend URL used by the frontend | `http://127.0.0.1:8000` |

## Troubleshooting

**The dashboard says the market feed is unavailable**
- Confirm that the FastAPI backend is running on port `8000`.
- Open `http://127.0.0.1:8000/health` to check the backend.
- Check your internet connection and CoinGecko API access.
- If required, configure a valid CoinGecko API key.

**The frontend cannot connect to the backend**
- Confirm `NEXT_PUBLIC_API_URL` points to the backend.
- Restart the Next.js development server after changing `.env.local`.
- The backend CORS configuration currently allows `http://localhost:3000` and `http://127.0.0.1:3000`.

**CoinGecko returns an error**
- The public API may enforce rate limits or require an API key.
- Try again later or configure `COINGECKO_API_KEY`.

## Development notes

- The frontend requests fresh dashboard data on initial load and every 30 seconds.
- The Refresh button triggers a manual data request.
- The chart shows the current price deviation snapshot; it is not historical price tracking.
- The current implementation monitors three configured assets. It does not currently provide on-chain reserve verification, proof-of-reserves analysis, historical alerting, or automated trading.

## Security and repository hygiene

Do not commit API keys, `.env` files, virtual environments, dependency folders, or generated build output. Keep secrets in environment variables and use a `.gitignore`.

Suggested `.gitignore` entries:

```gitignore
# Python
__pycache__/
*.py[cod]
.venv/
venv/

# Environment variables
.env
.env.*
!.env.example

# Node / Next.js
node_modules/
.next/
out/

# Logs and OS files
*.log
.DS_Store
Thumbs.db
```

## Disclaimer

PegGuard AI is an educational and research project. Its risk scores are based on a limited set of market-price inputs and a heuristic formula. They should not be interpreted as guarantees of solvency, peg stability, or investment safety. Always verify information independently before making financial decisions.

## License

No license has been specified yet. Add a `LICENSE` file before presenting this project as open source.
