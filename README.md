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
- Reports Total Return, CAGR, Max Drawdown, Annualized Volatility, and Sharpe Ratio (assuming 0% risk-free rate) for both, plus an equity curve chart.

**Data cost:** one API call per ticker (full historical price series for the date range), pulled from FMP's free `/stable/historical-price-eod/full` endpoint.

**What this backtest does *not* do** — be aware of these before trusting the numbers:
- **No transaction costs or slippage.** Real trading has spreads and commissions, especially on illiquid micro-caps; this backtest ignores them entirely.
- **No survivorship bias correction.** Your watchlist is whatever tickers you type in today — it doesn't account for stocks that existed in the past but delisted, went bankrupt, or got acquired before the backtest window. This tends to make results look better than reality.
- **No look-ahead bias auditing.** The momentum ranking only uses data available as of each rebalance date, which is correct, but if you built your watchlist *today* based on knowledge of which stocks did well, that's a form of look-ahead bias baked into the ticker selection itself.
- **Small sample sizes.** A 20-ticker watchlist over 2 years is not statistically robust — treat results as illustrative, not a basis for real capital allocation.

If you want a more rigorous backtest, the natural next steps are: walk-forward validation (test on multiple non-overlapping periods), a much larger and survivorship-bias-free universe, and realistic transaction cost modeling.


Open `index.html` and find the `computeScores()` function. The current model is intentionally simple and built entirely from free-tier data:

- **Cheapness score**: where the current price sits within its own 52-week range (closer to the low → higher score). This is a range-position proxy, not a true value metric.
- **Momentum score**: today's % price change, squashed into a 0–1 range.
- **Quant Score**: `50% cheapness + 50% momentum`.

FMP's free plan does not include P/E, P/B, ROE, or other fundamental ratios — those live behind the Starter plan's Ratios/Key Metrics endpoints. If you upgrade, you can pull `/stable/ratios?symbol=X` per ticker and fold real value metrics into `computeScores()`.

## Disclaimer

Not investment advice. Micro-cap stocks carry high volatility, wide bid-ask spreads, and liquidity risk. Do your own diligence before trading on anything this tool surfaces.
