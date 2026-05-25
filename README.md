
# PineScriptV6-trend-following-strategy-example
PineScriptV6 trend following strategy example that uses multiple trend following  indicators to get a sentiment of trend and momentum. This is example code to learn from not live trading!

<img width="1494" height="777" alt="Capture23123" src="https://github.com/user-attachments/assets/bbe871b9-f865-4c64-a6e2-9ef0f917f8c8" />

# Sentiment Multiple Indicators — TradingView Pine Script Strategy

A TradingView Pine Script v6 strategy that combines multiple technical indicators into a single bullish, bearish, and neutral sentiment score.

The script is named:

```pine
strategy("sentiment multiple indicators")
```

It is designed to measure market sentiment by counting how many enabled indicator modules are bullish, bearish, or neutral. It then smooths those sentiment counts and uses the bullish score to open or close a long TradingView strategy position.

The script also emits a small JSON webhook alert showing whether the long signal is active.

---

## What This Code Does

This strategy:

- Runs as a TradingView `strategy()` script.
- Calculates a multi-indicator sentiment score.
- Tracks three sentiment buckets:
  - Positive / bullish sentiment
  - Negative / bearish sentiment
  - Neutral / consolidation sentiment
- Smooths the sentiment values with EMA logic.
- Opens a long trade when positive sentiment is strong enough.
- Closes the long trade when the bullish condition fails.
- Sends a JSON alert with a `long_active` state.
- Includes commented experimental sections for short trading, OI/DCA logic, and grid logic.

The active production-style part of the script is mainly a **long-only sentiment strategy**.

---

## High-Level Strategy Flow

```text
Price data
   |
   v
Multiple indicator modules calculate signals
   |
   v
Each module votes:
   bullish, bearish, or neutral
   |
   v
Votes are added into sentiment counters:
   slope_sentiment_positive
   slope_sentiment_negative
   slope_sentiment_neutral
   |
   v
Sentiment counters are smoothed
   |
   v
Long trade is opened or closed
   |
   v
Webhook alert sends long_active state
```

---

## Main Sentiment Counters

The script builds three main counters on every bar:

```pine
slope_sentiment_positive
slope_sentiment_negative
slope_sentiment_neutral
```

Each active indicator module adds one vote to one of these counters.

For example:

```text
Bullish signal  -> positive counter +1
Bearish signal  -> negative counter +1
No clear signal -> neutral counter +1
```

This creates a voting system where the final strategy decision is based on the combined result of many indicators instead of one signal.

---

## Indicator Modules Used

The script contains many indicator toggles, such as:

```pine
indicator_1
indicator_2
indicator_3
...
indicator_27
```

Many later indicators are commented out, but the active sections include the following major groups.

---

## 1. Ichimoku Sentiment

The first active module uses Ichimoku-style logic.

It calculates:

- Tenkan-sen
- Kijun-sen
- Senkou Span A
- Senkou Span B
- Kumo/cloud direction
- Chikou-style momentum confirmation

The Ichimoku module classifies trend using conditions such as:

```text
Bullish:
- Tenkan is above Kijun
- Momentum is positive
- Price is above the cloud
- Senkou cloud is green

Bearish:
- Tenkan is below Kijun
- Momentum is negative
- Price is below the cloud
- Senkou cloud is red
```

The script then applies slope analysis to Ichimoku components and adds the result to the sentiment vote.

---

## 2. Supertrend Sentiment

The code includes three Supertrend modules with different factors:

```pine
factor_S1 = 3.0
factor_S2 = 6
factor_S3 = 12
```

Each Supertrend module checks whether its direction has flipped bullish or bearish.

The modules vote:

```text
Supertrend bullish -> positive sentiment
Supertrend bearish -> negative sentiment
No clear state     -> neutral sentiment
```

Using multiple Supertrend factors gives the strategy a mix of faster and slower trend responses.

---

## 3. EMA Slope Sentiment

The script calculates slope values from EMA changes across several lookback groups.

Inputs include:

```pine
main_ema_len1 = 10
main_ema_len2 = 20
main_ema_len3 = 30
main_ema_len4 = 50
main_ema_len5 = 100
main_ema_len6 = 200
```

The script derives longer comparison windows by multiplying those lengths by 10.

For each EMA slope group, it compares the current normalized slope against historical average positive and negative slope values.

The result is classified as:

```text
Slope above positive average -> bullish
Slope below negative average -> bearish
Otherwise                    -> neutral
```

This lets the script decide whether current slope strength is meaningful compared with the symbol's own historical behavior.

---

## 4. HMA Slope Sentiment

The script also uses Hull Moving Average slope logic.

The HMA modules calculate a normalized slope and compare it against historical up-slope and down-slope averages.

This is used to detect faster trend changes than the slower EMA slope modules.

The active HMA groups add additional bullish, bearish, or neutral votes.

---

## 5. VWMA Slope Sentiment

The strategy includes Volume Weighted Moving Average slope modules.

VWMA slope is useful because it weights price movement by volume.

The logic is similar to the EMA and HMA modules:

```text
Current VWMA slope > historical bullish average -> positive vote
Current VWMA slope < historical bearish average -> negative vote
Otherwise                                      -> neutral vote
```

This helps the strategy account for volume-weighted trend direction.

---

## 6. Parabolic SAR Sentiment

Two Parabolic SAR configurations are active.

They use different settings:

```pine
SAR 1:
start = 0.02
increment = 0.02
maximum = 0.2

SAR 2:
start = 0.01
increment = 0.01
maximum = 0.1
```

Each SAR output is converted into a normalized slope and compared with historical slope averages.

The result becomes another sentiment vote.

---

## Source Price

The script currently uses the midpoint of the candle high and low as its main source price:

```pine
source_price = (high + low) * 0.5
```

There are commented options for using `close` or an alternate price calculation, but the active code uses high/low midpoint.

---

## Sentiment Smoothing

The raw sentiment counters are plotted first:

```pine
plot(slope_sentiment_neutral)
plot(slope_sentiment_positive)
plot(slope_sentiment_negative)
```

Then the script calculates smoothed versions:

```pine
slope_sentiment_neutral_sma
slope_sentiment_positive_sma
slope_sentiment_negative_sma
```

Despite the variable name `sma`, the active smoothing uses `ta.ema()`.

The update logic is time-aware:

```text
On minute charts:
- Updates at 00:00, 04:00, 08:00, 12:00, 16:00, and 20:00

On non-minute / higher timeframes:
- Updates directly using EMA
```

The smoothed values are also stored in arrays to build longer-term averages:

```pine
slope_sentiment_neutral_avg
slope_sentiment_positive_avg
slope_sentiment_negative_avg
```

These averages are used as dynamic thresholds.

---

# Trading Logic

## Active Trading Direction

The active strategy is **long-only**.

This input controls whether long trading is allowed:

```pine
long_trading_enabled = input.bool(true, 'enable long trading')
```

Short trading code exists in comments, but it is not active in the current script.

---

## Long Entry Logic

The script has two active long-entry modes controlled by:

```pine
close_trades
```

Options:

```text
avg line
avg line and consolidation
avg line and consolidation*0.5
```

The active default is:

```text
avg line and consolidation*0.5
```

### Default Long Entry Condition

A long trade is opened when:

```text
1. Positive smoothed sentiment is above its own long-term average.
2. Positive smoothed sentiment is greater than half of neutral sentiment.
3. Positive smoothed sentiment is greater than negative smoothed sentiment.
4. No existing Lsentiment trade is open.
5. Long trading is enabled.
```

In code form, the key condition is:

```pine
slope_sentiment_positive_sma > slope_sentiment_positive_avg
slope_sentiment_positive_sma > (slope_sentiment_neutral_sma * 0.5)
slope_sentiment_positive_sma > slope_sentiment_negative_sma
```

When true, the script enters:

```pine
strategy.entry('Lsentiment', strategy.long)
```

---

## Alternative Long Entry Mode

If `close_trades` is set to:

```text
avg line and consolidation
```

Then the condition becomes stricter.

The positive smoothed sentiment must be greater than the full neutral smoothed sentiment, not just half:

```pine
slope_sentiment_positive_sma > slope_sentiment_neutral_sma
```

This mode filters out more consolidation and should produce fewer trades.

---

## Long Exit Logic

The strategy closes `Lsentiment` when the selected long condition is no longer true.

The active close command is:

```pine
strategy.close('Lsentiment', comment = 'L closed')
```

In plain English:

```text
Stay long while bullish sentiment is strong.
Close the long when bullish sentiment weakens, loses to neutral sentiment, or loses to bearish sentiment.
```

---

## Position ID

The active long position uses this TradingView entry ID:

```text
Lsentiment
```

This makes it easy to filter and track the trade inside TradingView’s strategy tester.

---

# Webhook Alert

At the end of the script, the strategy builds a JSON alert payload.

The active payload contains:

```json
{
  "passphrase": "...",
  "long_active": "1"
}
```

or:

```json
{
  "passphrase": "...",
  "long_active": "0"
}
```

The alert is sent using:

```pine
alert(alertstring, alert.freq_all)
```

---

## Webhook Field Meaning

| Field | Meaning |
|---|---|
| `passphrase` | A shared secret or message string sent with each alert. |
| `long_active` | `1` when the script's active long condition is true, otherwise `0`. |

---

## Important Webhook Note

The current passphrase value in the script is not a clean API secret. It appears to be a note string:

```text
NOTE: i need to mine kaspa with virsion 2.04 not 2.05 cuz bugs
```

For a production webhook, replace this with a proper secret, for example:

```pine
_passphrase = 'CHANGE_ME_TO_A_RANDOM_SECRET'
```

A webhook receiver should reject alerts when the passphrase does not match.

---

## Example Alert Payload

When the long condition is active:

```json
{
  "passphrase": "CHANGE_ME_TO_A_RANDOM_SECRET",
  "long_active": "1"
}
```

When the long condition is inactive:

```json
{
  "passphrase": "CHANGE_ME_TO_A_RANDOM_SECRET",
  "long_active": "0"
}
```

---

# How a Webhook Receiver Could Use This

A Python, Flask, FastAPI, Node.js, or MT5 bridge can read the alert and decide whether to open or close a broker-side trade.

Example logic:

```python
payload = request.json

if payload["passphrase"] != EXPECTED_SECRET:
    reject_alert()

long_active = int(payload["long_active"])

if long_active == 1:
    open_or_hold_long_position()
else:
    close_or_hold_flat()
```

Important: the receiver should not blindly open a new trade on every alert. It should check whether a long position already exists.

---

# Strategy Settings

## Core Inputs

| Input | Purpose |
|---|---|
| `long_trading_enabled` | Enables or disables active long entries. |
| `close_trades` | Selects the long entry/exit condition style. |
| `neutral_input` | Multiplier used to adjust neutral/bullish/bearish slope thresholds. |
| `window_orig` | Base slope window. |
| `array_size_slope` | Number of historical slope values kept in arrays. |
| `ema_cal` | Intended source selector, although the active code currently uses high/low midpoint. |
| `array_size_sma` | EMA length used to smooth sentiment counters. |
| `dca_sentiment_thresh` | Threshold used by inactive/commented market-maker/grid style logic. |

---

## Indicator Toggle Inputs

The script includes toggles for many indicator modules:

```pine
indicator_1
indicator_2
indicator_3
...
indicator_27
```

Active sections use several of these toggles to include or exclude indicator votes.

Commented-out indicators include parts of ADX, OBV, additional SAR, and other experimental modules.

---

# Visual Output

The script is set with:

```pine
overlay=false
```

So it displays in its own panel below the chart.

It plots:

- Neutral sentiment
- Positive sentiment
- Negative sentiment
- Smoothed neutral sentiment
- Smoothed positive sentiment
- Smoothed negative sentiment
- Long-term average sentiment lines

Typical color meaning:

```text
Green  -> bullish / positive sentiment
Red    -> bearish / negative sentiment
Blue   -> neutral sentiment
Faded lines -> long-term averages
```

---

# Commented / Experimental Sections

The file contains large commented sections that are not active in the current strategy.

These include:

- Short trading logic
- OI-based DCA logic
- Grid trading logic
- Multiple grid entries from `grid_1` to `grid_50`
- Additional OBV / ADX / SAR modules
- More complex TP / SL alert fields

Because they are commented, they do not affect the current strategy unless re-enabled.

---

# Installation

1. Open TradingView.
2. Open the Pine Editor.
3. Paste the full Pine Script code.
4. Save the script.
5. Add it to a chart.
6. Review the strategy tester results.
7. Configure inputs in the TradingView settings panel.

---

# Alert Setup

To use the webhook output:

1. Add the strategy to the chart.
2. Click **Create Alert**.
3. Select this strategy as the alert condition.
4. Use an alert condition that listens for `alert()` function calls.
5. Enable **Webhook URL**.
6. Paste your webhook server URL.
7. Replace the passphrase in the code with a real secret.
8. Make sure the receiving server parses the JSON payload.

---

# Recommended Repository Structure

```text
sentiment-multiple-indicators/
│
├── README.md
├── pine/
│   └── sentiment_multiple_indicators.pine
│
├── webhook/
│   └── receiver_example.py
│
└── docs/
    └── strategy-notes.md
```

---

# Important Notes

- The active trading logic is long-only.
- Short logic appears to be planned or previously tested, but it is commented out.
- The strategy uses `strategy.percent_of_equity` sizing with a default quantity value of 100.
- `pyramiding = 999` is enabled, although the active `Lsentiment` logic prevents duplicate `Lsentiment` entries by checking open trade IDs.
- The script uses a very large `calc_bars_count` and large arrays, which may be heavy on lower timeframes or long histories.
- The webhook sends a state signal, not a complete trade order with TP/SL.
- A live execution bridge should include duplicate-trade protection and risk controls.

---

# Risk Warning

This code is for research and educational use. TradingView strategy results are not guaranteed to match live broker execution.

Before using this with real money:

- Backtest across multiple symbols and timeframes.
- Forward-test with paper trading.
- Confirm that alerts fire correctly.
- Use a secure webhook passphrase.
- Add broker-side risk limits.
- Avoid blindly opening duplicate trades from repeated alerts.
- Start with very small position sizes.

---

# License

The source code header states that the Pine Script is subject to the Mozilla Public License 2.0.

---

# Author

Created by JoeT_AlphaCapital.

