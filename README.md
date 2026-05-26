# Synthetic VIX Alert — MQL4 Script

A MetaTrader 4 script that computes a **synthetic implied volatility index** by calculating the annualized historical volatility from squared log returns over a rolling lookback window, deriving a rolling average of that volatility via a sequential `CalculateSyntheticVIX()` call chain, and firing spike or dip alerts when the current synthetic VIX reading exceeds `SpikeThreshold × vixAverage` or falls below `DipThreshold × vixAverage` — providing a broker-agnostic, formula-derived volatility index for any instrument where VIX data is unavailable.

---

## Overview

The Chicago Board Options Exchange Volatility Index (VIX) is derived from S&P 500 options pricing and is unavailable for most individual forex and CFD instruments. This script constructs a synthetic equivalent using the classical annualized historical volatility formula: for each bar in the lookback period, the natural log return `ln(closeToday / closeYesterday)` is computed and squared, the squared returns are averaged over the period, the result is square-rooted to produce standard deviation, then annualized by multiplying by `√252` (the conventional trading-day annualization factor) and scaled to percentage form. The resulting `syntheticVIX` is a percentage that approximates the implied volatility of the instrument. The script then computes a rolling average of these values across the same lookback window by calling `CalculateSyntheticVIX()` with progressively smaller sub-windows, and fires alerts when the current reading deviates from that average by the configured multiplier thresholds — giving traders a normalized, relative volatility signal rather than an absolute one.

> **Note on file naming:** This file is distributed as `Neural_Network_Prediction_001.mq4` but implements a Synthetic VIX alert. The README documents the actual implemented logic.

---

## Features

- **Annualized historical volatility computation** — iterates `i = 1` to `period`, computing `logReturn = MathLog(closeToday / closeYesterday)` for each bar; accumulates `logReturn²` into `sumLogReturns`; computes `syntheticVIX = MathSqrt(sumLogReturns / period) × MathSqrt(252) × 100`
- **Division-by-zero and non-positive price guard** — `closeToday > 0 && closeYesterday > 0` condition wraps each log return computation, skipping bars with zero or negative prices
- **Rolling VIX average** — `CalculateVIXAverage()` calls `CalculateSyntheticVIX(symbol, timeframe, i)` for `i = 1` to `period`, accumulating sub-window VIX values into `sumVIX / period` — produces a smoothed baseline for relative spike/dip comparison
- **Relative threshold detection** — `vixCurrent >= vixAverage × SpikeThreshold` → **Volatility Spike Detected**; `vixCurrent <= vixAverage × DipThreshold` → **Volatility Dip Detected** — both thresholds are multipliers of the rolling average, not absolute values
- **Alert message includes current and average VIX** — `AlertVIX()` formats both values for immediate relative context
- **Three notification channels:** sound alert, email, and mobile push
- **Lightweight loop** — polls once per minute (`Sleep(60000)`)

---

## How It Works

1. Every minute, `CalculateSyntheticVIX()` computes annualized volatility over `LookbackPeriod` bars
2. `CalculateVIXAverage()` computes rolling average of VIX sub-windows
3. Two relative threshold comparisons evaluated:
   - `vixCurrent >= vixAverage × SpikeThreshold` → **Volatility Spike Detected**
   - `vixCurrent <= vixAverage × DipThreshold` → **Volatility Dip Detected**
4. `AlertVIX()` dispatches via all enabled channels with both values

---

## Input Parameters

| Parameter         | Type            | Default     | Description                                                              |
|-------------------|-----------------|-------------|--------------------------------------------------------------------------|
| `TradeSymbol`     | string          | `EURUSD`    | Symbol for analysis                                                      |
| `Timeframe`       | ENUM_TIMEFRAMES | `PERIOD_H1` | Timeframe for volatility calculation                                     |
| `LookbackPeriod`  | int             | `14`        | Lookback period for log return accumulation and rolling average          |
| `SpikeThreshold`  | double          | `2.0`       | Multiplier of rolling average above which a volatility spike is triggered|
| `DipThreshold`    | double          | `0.5`       | Multiplier of rolling average below which a volatility dip is triggered  |
| `EnableAlerts`    | bool            | `true`      | Fire an on-screen/sound alert                                            |
| `EnableEmail`     | bool            | `false`     | Send an email notification                                               |
| `EnablePush`      | bool            | `false`     | Send a mobile push notification                                          |

---

## Alert Message Format

```
Volatility Spike Detected detected on EURUSD (Timeframe: PERIOD_H1)
Current VIX: 18.42, Average VIX: 8.95
```

---

## Installation

1. Copy `Neural_Network_Prediction_001.mq4` to `MQL4/Scripts/` in your MT4 data folder
2. Compile in MetaEditor (F7)
3. Drag onto any chart from Navigator → Scripts
4. Configure inputs and click **OK**

---

## Requirements

- MetaTrader 4 (`#property strict` compatible build)
- MQL4 compiler (MetaEditor)

---

## License

MIT License

Copyright (c) 2026

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
