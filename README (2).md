# Quarterly Edge

A single-page stock research tool for options traders. No build step, no backend, no server — one `index.html` file that runs entirely in the browser and works on a phone.

Four tabs:

| Tab | What it does |
|---|---|
| **Seasonality** | Median return for every calendar month and every quarter across the full price history, plus hit rate, best/worst year, and a year × month heatmap. |
| **F & Z Scores** | Piotroski F-Score (0–9) and Altman Z-Score, with every component shown so you can see what drove the number. |
| **Contract Value** | Black-Scholes value and Greeks for a call or put, plus a profit/loss figure against **your own** price target and date. |
| **News & Analysts** | Analyst consensus and target, the last eight earnings surprises, and recent headlines with the vendor's sentiment score. |

---

## 1. Put it on GitHub Pages

```bash
# in a new, empty folder
git init
# copy index.html and README.md into this folder
git add .
git commit -m "Quarterly Edge"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/YOUR-REPO.git
git push -u origin main
```

Then in the repo on github.com: **Settings → Pages → Source: Deploy from a branch → Branch: `main` / `/ (root)` → Save.**

A minute or two later the site is live at:

```
https://YOUR-USERNAME.github.io/YOUR-REPO/
```

To use your own domain instead, add a file named `CNAME` containing just your domain name, and point a CNAME DNS record at `YOUR-USERNAME.github.io`.

**On your phone:** open that URL in Safari or Chrome and use *Add to Home Screen*. It then opens like an app. Your API key and cached data are stored per browser, so the phone needs the key entered once too.

To update the site later, edit `index.html`, then `git commit` and `git push`. Pages redeploys on its own.

---

## 2. Get a free API key

Price history and fundamentals come from [Alpha Vantage](https://www.alphavantage.co/support/#api-key). The free key takes about thirty seconds to claim — enter an email, copy the key.

In the app, open **Alpha Vantage API key** in the header, paste the key, press **Save**. The key is written to your browser's `localStorage` and nothing else; it is never sent anywhere except to Alpha Vantage itself. There is no server in this project that could receive it.

Because the key lives in the browser, **do not hard-code your key into `index.html` and push it to a public repo** — anyone could read it from the page source.

### Why Alpha Vantage and not Yahoo

Yahoo Finance (what your notebook uses via `yfinance`) blocks browser requests with CORS. From a static page there is no way around it short of running your own proxy. Alpha Vantage allows direct browser calls, and one key covers prices, filings, earnings, news, and analyst ratings.

---

## 3. The 25-requests-a-day budget

The free tier allows **25 requests per day** and 5 per minute. The app is built around that limit:

- Each section loads **only when you press its button**, so you never spend requests on a tab you didn't open.
- Every response is cached in the browser with its own expiry: price history 24h, quote 2h, filings 7 days, earnings 24h, news 3h. Re-loading a ticker you looked at this morning costs nothing.
- The header shows `n/25 calls today`. It turns amber at 15 and red at 22.
- **Clear cache** forces fresh data on the next load. It keeps your key and the counter.

What a full look at one new ticker costs:

| Action | Requests |
|---|---|
| **Load** (seasonality + current price) | 2 |
| **Load scores** (balance sheet, income, cash flow, overview) | 3–4 |
| **Load context** (overview, earnings, news) | 1–3 |
| Contract Value tab | 0 — pure math |

So roughly three complete tickers a day, or many more if you stay on names you've already looked at. Alpha Vantage sells a higher tier if you outgrow it; nothing in the code needs to change.

Type `DEMO` and press Load to browse the whole interface on synthetic data without spending a request.

---

## 4. What the numbers mean

**Seasonality.** Monthly returns come from `TIME_SERIES_MONTHLY_ADJUSTED`, which is split- and dividend-adjusted. For each calendar month the app reports the median across all years, because one 2008 or one 2020 can drag a mean far enough to invert the ranking. Quarterly returns compound three monthly returns; a quarter with fewer than three months of data is dropped rather than averaged in alongside complete ones.

Twenty years of history is twenty observations per month. That is a small sample and a strong-looking month can be the work of two or three years. The hit rate column and the year grid are there so you can see whether a median is broad or lumpy.

**Piotroski F-Score.** Nine accounting tests, one point each, comparing the two most recent annual filings: four on profitability, three on leverage and liquidity, two on operating efficiency. Piotroski treated 8–9 as strong and 0–1 as weak. It measures whether the books got healthier, not whether the stock is cheap. A test whose inputs are missing from the filing shows `–` and is excluded from the denominator, which the card states.

**Altman Z-Score.** Distress risk over roughly two years. The classic manufacturer form is `1.2·X1 + 1.4·X2 + 3.3·X3 + 0.6·X4 + 1.0·X5`, safe above 2.99 and distressed below 1.81. For technology, services, healthcare and communications the app shows the Z″ variant (`6.56·X1 + 3.26·X2 + 6.72·X3 + 1.05·X4b`, thresholds 1.1 and 2.6), which drops asset turnover — that term punishes asset-light businesses. Both figures are always shown so you can see the other one. For banks and insurers the app flags the score as not applicable: Altman excluded financials from the model, because their balance sheets make these ratios mean something different.

**Contract value.** Black-Scholes for a European option with a continuous dividend yield, at the implied volatility you type in. Delta, gamma, vega and rho are per contract of 100 shares; theta is per calendar day. Real contracts on single stocks are American, trade inside a bid-ask spread, and reprice when implied volatility moves, so read the model value as a reference rather than a quote.

**News and analysts.** Sentiment scores are Alpha Vantage's own model reading headlines and summaries. They describe tone. They are not a forecast, and they do not become one by being averaged.

---

## 5. What this tool does not do

It does not predict where a share price will go, and nothing in it turns news, earnings, or analyst opinion into a projected option value. That calculation does not work — a sentiment average has no reliable relationship to a future return, and dressing one up as a price target would make the output look more trustworthy than it is.

What it does instead: shows you those signals as data, on their own terms, and lets you enter your own price target and date. The Black-Scholes math then tells you what the contract would be worth **if** that target is hit on that date, and what the profit or loss would be on the premium you paid. The judgement stays yours; the arithmetic is the tool's job.

---

## Notes

- **Not investment advice.** Research tool. Verify any figure against the filing before acting on it.
- Free-tier Alpha Vantage data is delayed end-of-day, not real time.
- No dependencies, no bundler, no framework. One HTML file, hand-rolled inline SVG charts, roughly 40 KB.
- Works offline for anything already cached, including the Contract Value tab, which needs no network at all.
