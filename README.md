# Ledger — Micro-Cap Quant Screener

A single-file, static web app that screens US micro-cap stocks ($50M–$300M by default) using a blended value + momentum score. Runs entirely in the browser — no backend, no build step.

## What it does

- You maintain a **watchlist** of tickers (edit the textarea in the app — starts with a small preset list).
- Pulls live company profile data from [Financial Modeling Prep](https://financialmodelingprep.com)'s **free API tier** (250 requests/day, no card required) — one request per ticker.
- Filters by market cap, beta, average volume, and sector.
- Computes a **Quant Score** per stock: 50% "cheapness" (how close price sits to its own 52-week low) + 50% momentum (today's % price change).
- Sortable results table, client-side only.
- Your API key is stored in your browser's `localStorage` — it never leaves your machine except in direct calls to FMP.

### Why a watchlist instead of a full-market scan?

FMP's automated **Stock Screener** endpoint (the one that scans the entire market by market cap/sector/etc in one call) requires their **Starter plan ($22/mo)** — it's not included in the free tier. The **Company Profile** endpoint, which returns price, market cap, sector, beta, and 52-week range for any individual US ticker, *is* free. So this app fetches profiles for a list of tickers you provide, then filters and scores them client-side. With 250 free requests/day, you can screen up to 250 tickers per run.

If you want a true full-market scan without maintaining a manual list, upgrade to FMP's Starter plan and swap the app back to calling `/stable/company-screener`.

## Run it locally

Just open `index.html` in a browser. No install needed.

## Deploy to GitHub Pages (free hosting)

1. Create a new GitHub repo (e.g. `microcap-screener`).
2. Upload `index.html` (and this README) to the repo — either via the GitHub web UI ("Add file → Upload files") or:
   ```bash
   git init
   git add index.html README.md
   git commit -m "Initial screener"
   git branch -M main
   git remote add origin https://github.com/YOUR_USERNAME/microcap-screener.git
   git push -u origin main
   ```
3. In the repo, go to **Settings → Pages**.
4. Under "Build and deployment", set **Source** to `Deploy from a branch`, branch `main`, folder `/ (root)`. Save.
5. GitHub will give you a live URL within a minute or two, typically:
   `https://YOUR_USERNAME.github.io/microcap-screener/`

That's it — it's live, free, and shareable.

## Get your free API key

Sign up at [financialmodelingprep.com](https://site.financialmodelingprep.com/pricing-plans) — the free tier gives 250 requests/day, US equities, end-of-day data. No credit card required. Paste the key into the app's "Data Connection" panel and click "Save Key."

## Backtesting

Below the screener, there's a second panel: a **momentum backtest** that runs against the same watchlist.

**How it works:**
- At each rebalance date (monthly or quarterly, your choice), it ranks every ticker in your watchlist by trailing price return over a lookback window (default 90 days).
- It buys the top N ranked tickers, equal-weighted, holding them until the next rebalance — then repeats.
- It compares this against a **buy-and-hold benchmark**: an equal-weight basket of your entire watchlist, bought once at the start and held for the whole period, no rebalancing.
- Reports Total Return, CAGR, Max Drawdown, Annualized Volatility, Sharpe Ratio (assuming 0% risk-free rate), and total trading costs paid, for both, plus an equity curve chart.

**Transaction costs and slippage (now modeled):**
Every trade — every buy and every sell, at every rebalance — now has a cost applied. This has two parts:
- A **fixed cost** in basis points (configurable in the UI, default 50bps = 0.5% per trade) representing spread/commission.
- A **liquidity-impact estimate**: the app calculates each stock's trailing 20-day average dollar volume from the price data it already has, and scales up the cost when a trade is large relative to that — approximating that trying to trade a large amount in an illiquid micro-cap moves the price against you. This is a simple, transparent heuristic (`estimateSlippageBps()` in the source), not a precise market-impact model — treat it as directionally useful, not exact.

**Multi-period validation (now included):**
The "Validation Periods" setting splits your chosen date range into 2-4 independent, non-overlapping chunks and re-runs the same strategy in each one separately, using data already fetched (no extra API calls). This tells you whether the strategy's edge holds up across different market conditions, or whether the headline result is being carried by one lucky stretch. The results table shows how many of the sub-periods the strategy actually beat the benchmark in.

**Data cost:** one API call per ticker (full historical price series for the date range), pulled from FMP's free `/stable/historical-price-eod/full` endpoint. Multi-period validation reuses this same data, so it doesn't cost extra calls.

**What this backtest still does *not* do** — the one limitation left that this app genuinely cannot fix on free/cheap data:
- **No survivorship bias correction.** Your watchlist is whatever tickers you type in today — it doesn't account for stocks that existed in the past but delisted, went bankrupt, or got acquired before the backtest window. This tends to make results look better than reality. Fixing this properly requires point-in-time historical constituent data (who provides this: Norgate Data, Sharadar via Nasdaq Data Link) which is a paid, different kind of data product — Norgate in particular is a local Windows database, not a web API, so using it would mean rearchitecting this app with a real backend rather than swapping an API key. For a personal learning tool, this is a known, documented gap rather than something worth the added complexity right now.

If you want a more rigorous backtest beyond what's here, the natural next step is expanding to a much larger, ideally survivorship-bias-free universe with a proper backend — a meaningfully bigger project than this one.


Open `index.html` and find the `computeScores()` function. The current model is intentionally simple and built entirely from free-tier data:

- **Cheapness score**: where the current price sits within its own 52-week range (closer to the low → higher score). This is a range-position proxy, not a true value metric.
- **Momentum score**: today's % price change, squashed into a 0–1 range.
- **Quant Score**: `50% cheapness + 50% momentum`.

FMP's free plan does not include P/E, P/B, ROE, or other fundamental ratios — those live behind the Starter plan's Ratios/Key Metrics endpoints. If you upgrade, you can pull `/stable/ratios?symbol=X` per ticker and fold real value metrics into `computeScores()`.

## Disclaimer

Not investment advice. Micro-cap stocks carry high volatility, wide bid-ask spreads, and liquidity risk. Do your own diligence before trading on anything this tool surfaces.
