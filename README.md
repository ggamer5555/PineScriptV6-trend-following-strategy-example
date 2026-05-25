
# PineScriptV6-trend-following-strategy-example
PineScriptV6 trend following strategy example that uses multiple trend following  indicators to get a sentiment of trend and momentum. This is example code to learn from, not live trading! can be used on any timeframe but designed for 1h, 2h, 4h!

<img width="1494" height="777" alt="Capture23123" src="https://github.com/user-attachments/assets/bbe871b9-f865-4c64-a6e2-9ef0f917f8c8" />

# Sentiment 2 TradingView Strategy

A TradingView Pine Script strategy that builds a custom market sentiment score from several technical indicators, then uses that score to generate long and optional short trade signals.

The script combines EMA slope, HMA slope, high/low midpoint positioning, Supertrend distance, and Parabolic SAR distance into one normalized sentiment value. It can then trade using threshold rules, custom PSAR rules, Supertrend rules, or sentiment-vote rules.

> **Disclaimer:** This project is for educational and research purposes only. It is not financial advice. Always backtest, forward-test, and manage risk before using any trading strategy with real capital.

## What the Code Does

The strategy calculates a sentiment score called `total_percentage`, which usually ranges between `0` and `1`.

- Values closer to `1` indicate stronger bullish conditions.
- Values closer to `0` indicate stronger bearish conditions.
- Values around `0.5` are treated as more neutral.

The strategy then uses this sentiment score to decide when to enter or exit trades.

By default, the script is configured for:

- Initial capital: `1000`
- Position sizing: `99%` of equity
- Pyramiding: disabled
- Long trades: enabled
- Short trades: disabled
- Trading mode: `2`

## Core Components

### 1. EMA Slope Sentiment

The script calculates four Exponential Moving Averages using different lengths:

- Slow EMA
- Medium EMA
- Fast EMA
- Ultra-fast EMA

For each EMA, it calculates the slope, normalizes that slope using standard deviation, then converts it into a sentiment percentage.

### 2. HMA Slope Sentiment

The script repeats a similar process using Hull Moving Averages. HMA is generally more responsive than EMA, so this adds a faster trend-following component.

### 3. High/Low Midpoint Sentiment

The strategy calculates midpoint levels between recent highs and lows over several lookback periods. It then compares the close price against smoothed midpoint values to determine bullish or bearish pressure.

### 4. Supertrend Sentiment

The script calculates several Supertrend lines with different ATR factors. It measures how far price is from each Supertrend line and converts those readings into sentiment values.

### 5. SAR Sentiment

The strategy calculates multiple Parabolic SAR readings using different start, increment, and maximum values. Price distance from SAR is normalized and added to the overall sentiment calculation.

### 6. Combined Sentiment Score

Each enabled indicator group contributes to the final `total_percentage` score.

The script also tracks:

- `sentiment_p`: bullish readings
- `sentiment_n`: bearish readings
- `sentiment_neu`: neutral readings

These are smoothed with a 10-period SMA and plotted for visual analysis.

## Plotted Outputs

The script plots several custom outputs in a separate pane:

- Custom PSAR based on the sentiment score
- Custom ATR based on changes in the sentiment score
- Custom Supertrend based on the sentiment score
- Smoothed bullish, bearish, and neutral sentiment counts
- Horizontal threshold lines for long and short entries

## Trading Modes

The `trading_mode` input controls how entries and exits are generated.

| Mode | Logic |
|---:|---|
| `1` | Uses custom PSAR direction against `total_percentage`. |
| `2` | Enters long when sentiment is above the long threshold and short when sentiment is below the short threshold. |
| `3` | Requires both PSAR confirmation and sentiment threshold confirmation. |
| `4` | Uses PSAR plus entry thresholds, with separate exit thresholds. |
| `5` | Trades based on the custom sentiment Supertrend direction. |
| `6` | Uses sentiment Supertrend plus entry and exit thresholds. |
| `7` | Trades when bullish or bearish sentiment is stronger than both opposing and neutral sentiment. |
| `8` | Similar to mode 7, but requires bullish or bearish sentiment to exceed full neutral sentiment. |
| `9` | Trades based only on whether bullish sentiment is greater than bearish sentiment, or vice versa. |
| `10` | Trades based on the custom PSAR value crossing fixed levels around `0.5`. |

## Main Inputs

### Trading Settings

| Input | Description |
|---|---|
| `trading_enabled` | Turns automated strategy entries and exits on or off. |
| `longs_enabled` | Allows long trades. |
| `shorts_enabled` | Allows short trades. |
| `long_thresh_entry` | Sentiment level required for long entries. |
| `short_thresh_entry` | Sentiment level required for short entries. |
| `long_thresh_exit` | Sentiment level used to exit long trades in some modes. |
| `short_thresh_exit` | Sentiment level used to exit short trades in some modes. |
| `trading_mode` | Selects which entry and exit logic to use. |

### EMA and HMA Settings

| Input | Description |
|---|---|
| `ema_enabled` | Enables or disables EMA-based sentiment. |
| `hma_enabled` | Enables or disables HMA-based sentiment. |
| `average_periods` | Slow moving average length. |
| `average_periods1` | Medium moving average length. |
| `average_periods2` | Fast moving average length. |
| `average_periods3` | Ultra-fast moving average length. |
| `array_size_ema` | Number of historical normalized slopes used in rolling arrays. |

### High/Low Settings

| Input | Description |
|---|---|
| `hl_enabled` | Enables or disables high/low midpoint sentiment. |
| `hl_sma_length` | Smoothing length for high/low midpoint calculations. |
| `hl_length1` to `hl_length4` | Lookback periods used to calculate high/low midpoints. |

### Supertrend Settings

| Input | Description |
|---|---|
| `super_enabled` | Enables or disables Supertrend-based sentiment. |
| `atrPeriod_S1` to `atrPeriod_S4` | ATR lengths for the Supertrend calculations. |
| `factor_S1` to `factor_S4` | Supertrend multiplier factors. |

### SAR Settings

| Input | Description |
|---|---|
| `sar_enabled` | Enables or disables SAR-based sentiment. |
| `start_SAR_1` to `start_SAR_4` | SAR starting acceleration values. |
| `increment_SAR_1` to `increment_SAR_4` | SAR acceleration increments. |
| `maximum_SAR_1` to `maximum_SAR_4` | SAR maximum acceleration values. |

## How to Use

1. Open TradingView.
2. Open the Pine Editor.
3. Paste the strategy code into the editor.
4. Save the script.
5. Add it to a chart.
6. Open the strategy settings and adjust the inputs for your market and timeframe.

## Suggested Backtesting Workflow

1. Test one market and one timeframe at a time.
2. Compare all trading modes.
3. Test long-only, short-only, and long/short configurations separately.
4. Adjust thresholds slowly instead of over-optimizing.
5. Include realistic fees, slippage, and position sizing.
6. Validate results across out-of-sample periods.

## Important Notes

- The strategy uses normalized indicator readings, so results may vary significantly across assets and timeframes.
- Some calculations rely on long lookback periods, including standard deviation windows and rolling arrays.
- If too few bars are available, early results may be less reliable.
- The default configuration has shorts disabled.
- The script plots in a separate pane because `overlay=false`.

## Risk Warning

Algorithmic trading strategies can lose money. Backtested performance does not guarantee future results. Use proper risk management, avoid overfitting, and test thoroughly before trading live.
