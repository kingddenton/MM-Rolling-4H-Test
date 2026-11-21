# Project Brief — 3M–5M RSI Monitor (Stable Working Version)

**Snapshot date:** 2026-11-21  
**Repository:** `kingddenton/MM-Rolling-4H-Test`  
**Live file:** `3m-5m-rsi-monitor.html`  
**Stable backup:** `backups/3m-5m-rsi-monitor-stable-working-version-2026-11-21.html`  
**Main source blob at snapshot:** `5619932b0b5be4bfc3ea97ac1ba21c6e1d0fe761`

## 1. Purpose and scope

A browser-based Binance USDⓈ-M USDT-margined market scanner for identifying potential bullish and bearish setups. It is a scanner only: it does not place trades and does not require a Binance API key. The page describes its market universe as Crypto + TradFi, while scanning Binance USDⓈ-M instruments.

This brief documents the source snapshot above. It is not a claim that the live page has been browser-tested in this session.

## 2. Core design

- The scanner first selects a market universe using the configured universe mode (24H-change threshold, Top X / Bottom X, or 15M Strong EMA).
- Universe membership is separate from signal qualification. Being selected does not itself make a coin a Potential Buy or Potential Sell.
- Buy and Sell use independent timeframe selectors. Each selector offers 1m, 3m and 5m; the source currently defaults both to 3m.
- Signal calculations use completed candles. The currently forming candle is excluded from the Buy/Sell timeframe price-change calculation and the indicator close series.
- The scanner calculates MACD (12, 26, 9), Wilder RSI, and EMA 7/25/99 for the selected setup timeframes. It also computes additional multi-timeframe indicator values used by the wider monitor.

## 3. Candle-history recovery — key reliability correction

MACD/RSI signal evaluation requires at least 40 completed closes (or the RSI period plus one, if greater).

The current `getCachedKlines(symbol, tf)` recovery path:
- Uses a fresh WebSocket cache only when it contains at least 40 candles and is sufficiently recent.
- If the cache is missing or has fewer than 40 candles, fetches a full Binance REST kline history (up to 100 candles).
- If a stale-cache merge still leaves fewer than 40 candles, fetches a full history again.
- Raises an explicit error if Binance returns no candle history.

The signal state reports `insufficient-data`, loaded candle count and required candle count when history is inadequate. Diagnostics can therefore distinguish missing candle history from a genuine absence of a MACD crossover.

## 4. Potential Buy / Potential Sell qualification

The scanner evaluates the latest MACD crossover on each side:
- Potential Buy requires the latest crossover to be bullish (DIF crossing above DEA), with a qualifying RSI + MACD candle in the configured post-crossover window.
- Potential Sell uses the opposite bearish crossover (DIF crossing below DEA) and its configured RSI + MACD conditions.
- Buy and Sell have separate RSI ranges and candle-window inputs.
- The HTML source defaults are Buy RSI 60–72, Sell RSI 33–42, and 5 qualification candles for each side. These are editable UI defaults; saved or manually selected browser settings may differ.

A coin can be present in the selected universe but fail to qualify because its latest crossover is the wrong direction, it has no crossover, it is outside the candle window, its RSI/MACD conditions are not met, or its candle history is insufficient.

## 5. Retention after qualification

Once a setup passes qualification and the enabled initial filters, it is retained in the corresponding Potential Buy or Potential Sell set.

A retained setup is removed when:
- The opposite MACD crossover replaces the qualifying direction.
- The original crossover identity changes.
- Its age exceeds the configured qualification window (or is no longer valid).

Later changes in RSI or EMA alignment do not by themselves revoke an already retained setup. The display is limited to the newest 10 Buy entries and newest 10 Sell entries, ordered newest first.

Retention is in browser memory; a page refresh clears the in-memory retained setup lists.

## 6. Price-change filter — qualification is not price filtering

**This is a defining behavior of this stable version.**

The Buy TF and Sell TF price-change controls are optional post-qualification display filters. They do not determine whether the MACD/RSI setup qualifies, and they are not part of the initial retention gate.

- The scanner first qualifies and retains a setup.
- It then applies the relevant Buy or Sell timeframe price-change minimum/maximum to decide whether the retained setup is currently displayed.
- A qualified setup that does not meet the price-change filter remains retained and can appear later when its price change meets the configured range.
- Diagnostics label these separately, including “Qualified — waiting for Buy TF price-change filter”, “Qualified — waiting for Sell TF price-change filter”, and “Qualified / retained — price filter passed”.

The price change is calculated from the latest completed candle on the selected side's timeframe:
`(completed candle close − completed candle open) / completed candle open × 100`.

The currently forming candle is excluded. This is a single-candle open-to-close move, not cumulative movement since the crossover.

The user's current working baseline is **0.2% minimum** to screen out small, noisy timeframe moves. The source leaves the min/max inputs blank by default, so 0.2% is a user-configured operating setting, not a hard-coded threshold.

## 7. Other optional initial filters

- Buy: optional positive 24H change direction filter.
- Sell: optional negative 24H change direction filter.
- Buy: optional EMA alignment filter (7 > 25 > 99) at qualification time.
- Sell: optional EMA alignment filter (7 < 25 < 99) at qualification time.
- The current source also contains an optional 1M Open Interest filter hook and controls. Do not describe this particular file as having all OI components removed.

These enabled initial filters can prevent a new setup from entering retention. They do not continuously remove a setup that has already been retained.

## 8. Diagnostics and observability

The signal diagnostics are intended to show the selected universe's Buy and Sell evaluation, including coins with no crossover, and to distinguish:
- Candle-fetch errors.
- Insufficient candle history (loaded / required count).
- No MACD crossover.
- Latest crossover in the wrong direction.
- Crossover outside the configured window.
- No qualifying RSI + MACD candle.
- Enabled initial-filter rejection.
- Qualified setup waiting for its post-qualification price-change filter.
- Qualified/retained setup passing that filter.

Diagnostics are calculated from the scan results; they are not intended to trigger a separate diagnostic API scan.

## 9. Operating principles for this snapshot

1. Preserve the current working logic while observing live market behavior.
2. Do not move timeframe price-change conditions back into the qualification or retention gate.
3. Keep the 0.2% minimum as the current noise-screening baseline unless observation justifies a deliberate change.
4. If a coin is absent, inspect universe selection, candle-history status, MACD crossover, RSI/MACD qualification, initial filters, retention, and finally the post-qualification price filter—in that order.
5. Before future code changes, create a dated backup and verify the resulting source.
6. Distinguish source inspection from live browser/runtime testing; do not claim runtime validation unless it has actually been performed.

## 10. Snapshot and restore reference

The exact source snapshot is also stored at:

`backups/3m-5m-rsi-monitor-stable-working-version-2026-11-21.html`

That backup was created from the main file and verified byte-for-byte at the time of creation. The main scanner file was not changed when the backup was created.

**Current status:** Stable working snapshot — observe before adding or changing features.
