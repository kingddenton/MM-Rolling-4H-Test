# 3M–5M RSI Monitor — Stable Working Version

**Project:** MM-Rolling-4H-Test  
**Version date:** 21 November 2026  
**Primary file:** `3m-5m-rsi-monitor.html`  
**Stable backup:** `backups/3m-5m-rsi-monitor-stable-working-version-2026-11-21.html`  
**Purpose:** Binance USDⓈ-M USDT perpetual-futures market scanner. It identifies and displays potential setups; it does not place trades.

## 1. Current milestone

This version is the stable working checkpoint following two important fixes:

1. **Candle-cache recovery.** Empty or partial WebSocket candle caches are replenished with Binance REST kline history. The scanner requires at least 40 closes (or RSI period + 1, if greater) before calculating MACD/RSI. Insufficient history is reported explicitly instead of being misclassified as a bearish/bullish signal.
2. **Qualification separated from timeframe price-change filtering.** A setup is first evaluated for signal qualification and initial filters. The selected Buy/Sell timeframe price-change range is then applied as a post-qualification visibility filter. A qualified setup is retained while waiting for the price filter; it is not disqualified merely because current candle movement is too small.

The user regards timeframe price changes below **0.2%** as largely noise. Treat 0.2% as the current operating threshold when configured, not as a hard-coded qualification requirement.

## 2. Market universe and scan process

- Uses Binance USDⓈ-M USDT perpetual contracts (USDT quote and margin assets; trading status required).
- Universe is assembled from exchange metadata and the 24-hour ticker data, with the configured minimum 24-hour quote-volume and ranking/universe controls.
- Scanner evaluates each selected symbol across the timeframe jobs required by its enabled monitor components.
- Buy and Sell setups have independent timeframe selectors. The current UI offers 1m, 3m and 5m; each selector is independently controlled.
- Concurrency is configurable and capped at 20 workers in the code. The performance screenshot for this checkpoint showed 20 peak workers.

## 3. Potential Buy and Potential Sell

The Buy and Sell setup panels have separate controls and independent timeframe selection.

### Buy side
- Detects the latest bullish MACD crossover on the selected Buy timeframe.
- Uses MACD (12, 26, 9) and RSI-based qualification.
- Optional positive 24-hour change filter.
- Optional EMA alignment check: EMA 7 > EMA 25 > EMA 99 on the selected Buy timeframe.
- Optional minimum/maximum Buy-timeframe price-change range, applied after qualification.

### Sell side
- Detects the latest bearish MACD crossover on the selected Sell timeframe.
- Uses MACD (12, 26, 9) and RSI-based qualification.
- Optional negative 24-hour change filter.
- Optional EMA alignment check: EMA 7 < EMA 25 < EMA 99 on the selected Sell timeframe.
- Optional minimum/maximum Sell-timeframe price-change range, applied after qualification.

### Qualification and retention rules
- A valid directional MACD crossover and the configured RSI/candle-window conditions are required for initial setup qualification.
- Enabled initial filters (such as 24-hour direction, EMA alignment and any enabled OI filters in this particular monitor) are checked at qualification time.
- Once qualified, the setup is retained in its side's list. Later changes in RSI or EMA alignment do not by themselves revoke an already-retained setup.
- Retained setups are removed when the opposite MACD crossover replaces the qualifying crossover, the selected timeframe changes, the symbol is no longer present in the current result set, or the applicable candle-age window is exceeded.
- Buy and Sell candle windows are independently configurable. The screenshot supplied on 21 November showed a 25-candle Buy window and a 15-candle Sell window; these are observed settings, not assumed universal defaults.
- Price-change thresholds do **not** determine initial qualification or retention. They determine whether the qualified/retained setup is currently displayed as having passed the post-qualification price filter.
- Price change is measured using the latest completed candle on the selected setup timeframe, not the currently forming candle. It is a single-candle open-to-close percentage, not cumulative movement since crossover.

## 4. Diagnostics and performance panel

The lightweight diagnostics report scan performance and signal-decision reasons without changing signal decisions.

### Performance counters
- Scan duration and universe-loading time.
- Selected/checked timeframe jobs, started, finished, failed and not-started jobs.
- Coin count and average time per coin.
- REST request count, candle-cache hits/misses and cache-hit rate.
- Peak active workers and recent scan-cycle history.

### Signal rejection diagnostics
- Lists each scanned symbol separately for BUY and SELL.
- Shows timeframe, latest MACD crossover direction, candle age, RSI and decision/rejection reason.
- Includes candle-fetch failures and insufficient candle-history reasons.
- Summary counts are per side: for a universe of 164 coins, 164 BUY checks plus 164 SELL checks produce 328 side checks. They are not 328 unique coins.

### Observed validation snapshot (user screenshot, 21 Nov 2026)
- 164 coins; 984/984 timeframe jobs started and finished.
- 0 failed jobs; 0 not-started jobs.
- Last scan: 4.4 seconds; average per coin: 27 ms.
- 20 peak workers.
- 983 cache hits and 1 miss (approximately 99.9% cache hits).
- BUY diagnostic reasons summed to 164; SELL diagnostic reasons also summed to 164.
- This is a single observed scan snapshot, not a guaranteed performance benchmark.

## 5. Other visible monitor components

The page also contains:
- RSI filter matches, including 1M, 3M, 5M, 15M, 1H and 4H RSI columns.
- A negative-MACD/RSI alert component.
- A 24-hour high rejection / short-watch component, described in the UI as price within 1% of the rolling 24-hour high followed by a completed bearish 1-minute candle.
- Optional OI filter controls are present in this particular RSI Monitor file. Do not confuse this page with the separate MM Live Scanner version from which OI components were removed.

## 6. Data and candle handling

- Uses Binance public Futures market-data endpoints; no API key is required for these public market-data requests.
- Candle history is cached and updated through WebSocket/REST handling.
- Empty or partial cache (<40 candles) triggers a full REST history request (up to 100 klines).
- Signal calculations require adequate closed-candle history; the currently forming candle is excluded from completed-candle signal/price-change decisions.
- If candle history cannot be retrieved, the symbol's error is surfaced in diagnostics rather than silently treated as a valid signal.

## 7. Operating guidance

- Preserve this checkpoint while observing live market behaviour; avoid adding features or changing qualification logic without a demonstrated issue.
- Treat 0.2% as the current noise-control price-change threshold when entered in the post-qualification filter.
- Check that scan jobs reconcile with the selected coin/timeframe workload and that BUY and SELL diagnostic reason totals each reconcile to the number of scanned symbols.
- A high cache-hit rate and zero failed/not-started jobs are useful health indicators, but do not alone prove every signal is financially correct.
- The scanner is an analysis/alert tool only. It does not execute orders.

## 8. Restore point and file identity

The stable backup was created in the repository and fetched again to verify that its contents exactly matched the main scanner file at the time of backup.

- Backup path: `backups/3m-5m-rsi-monitor-stable-working-version-2026-11-21.html`
- Main file at checkpoint: `3m-5m-rsi-monitor.html`
- Main-file blob SHA at checkpoint: `5619932b0b5be4bfc3ea97ac1ba21c6e1d0fe761`
- Backup creation commit: `9bad563fc34ea757cfec2e901322b6635999b7cb`

**Change control:** This brief documents the stable checkpoint. If the scanner logic changes later, update this brief and create a new dated backup rather than silently rewriting the description of this version.
