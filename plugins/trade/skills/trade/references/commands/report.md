---
type: Command Reference
title: "/trade report [tickers | basket]"
description: Today's capital-flow / 资金流向 read for one or more names — retail / 大单 / institutional from Futu OpenD actual capital distribution, short selling, and derivatives anomalies, mapped to a comparison table + cross-section synthesis. Read-only, not investment advice.
tags: [command, report, capital-flow, money-flow, options-flow, funds-flow, short-selling]
timestamp: 2026-06-23T20:00:00Z
---

# /trade report <tickers | basket>

A daily **capital-flow / 资金流向** read across one or more names: who is buying vs selling today, split as a **散户 / 大单 / 机构** representation, plus the price/volume and short selling context — rendered as a comparison table with a cross-section synthesis.

Runs whenever the user invokes `/trade report ...`, or asks for 资金流向 / 流入流出 / 净流入·净流出 / 散户·大单·机构 / capital flow / money flow / "who's buying" across a name or a basket.

> **Read the 口径 (data-source reality) FIRST — and state it in every reply.** This command leverages a running **FutuOpenD gateway** (default `127.0.0.1:11111`) and Futu OpenAPI python scripts (`futuapi` and `futu-*` skills) to fetch actual stock-side and derivatives capital flow data:
>
> - **大单 / 机构 (Smart Money)** ← Stock-side large and extra-large orders (`super` + `big` from `get_capital_distribution.py`), plus unusual option trades and sweep alerts from `handle_derivatives_anomaly.py`.
> - **散户 (Retail)** ← Stock-side small orders (`small` from `get_capital_distribution.py`), plus community feed sentiment from the Futu discussion API.
> - **做空情况 (Short Selling)** ← Daily short volume and short volume ratio from `get_daily_short_volume.py`.
>
> **Prerequisite:** FutuOpenD must be running locally. If it fails, report that OpenD is not running, and suggest the user start the OpenD client or run `/install-futu-opend`.

## Arguments

- **Explicit tickers** (space- or comma-separated): `report COHR LITE MU` → run those.
- **A sector / theme word** (e.g. "光" / optical, "存储" / memory, "光模块+存储"): **confirm the ticker universe first** (propose constituents + a market, ask via `AskUserQuestion`) — don't silently guess a basket. Once confirmed, group the output by basket.
- **Optional date**: default **today**. The endpoints return the latest session; if the user names a date, pass it through where the endpoint supports `date=`.
- Mixed baskets → render one table per basket so the cross-section reads cleanly.

## Workflow

### 1. Resolve the data path & verify OpenD status

- The Futu OpenD API is executed via Python scripts located in the `futuapi` skill folder.
- Always check that the FutuOpenD gateway is active at `127.0.0.1:11111` first. If it cannot connect, guide the user to launch OpenD or run `/install-futu-opend`.
- Run commands using the shell (`run_command`). Normalize tickers to Futu format (e.g., `US.AAPL` or `HK.00700`).

### 2. Pull, per ticker

| # | Command / Script | Gives | Use for |
|---|---|---|---|
| 1 | `python ${FUTUAPI_DIR}/scripts/quote/get_capital_distribution.py <T> --json` | `capital_in_super`/`out_super`, `capital_in_big`/`out_big`, `capital_in_mid`/`out_mid`, `capital_in_small`/`out_small` | **核心** — Stock-side capital distribution by order size (today's session EOD/realtime snapshot) |
| 2 | `python ${FUTUAPI_DIR}/scripts/quote/get_daily_short_volume.py <T> --json` | `short_sell_volume`, `total_volume`, `short_sell_ratio` | Short selling activity and pressure |
| 3 | `python ${FUTUAPI_DIR}/scripts/quote/get_snapshot.py <T> --json` | `last_price`, `prev_close_price`, `open_price`, `high_price`, `low_price`, `volume`, `turnover`, `pe_ratio`, `pe_ttm_ratio` | Price change, volume, basic valuation metrics |
| 4 | `python ${DERIVATIVES_ANOMALY_DIR}/scripts/handle_derivatives_anomaly.py <T> --time-range 2 --json` | Options sweep/block orders, `option_unusual`, `option_volatility`, `option_sentiment` (Put/Call Ratio) | Option big-ticket flow, IV rank, sentiment skew |
| 5 | `curl -sG 'https://ai-news-search.futunn.com/stock_feed' --data-urlencode 'keyword=<ticker>' --data-urlencode 'size=30'` | Feed comments, timestamps, author, title, content | 散户 comment sentiment aggregation (via `futu-comment-sentiment`) |

*Note: The executing Agent must dynamically locate the absolute path of `futuapi` (for `${FUTUAPI_DIR}`) and `futu-derivatives-anomaly` (for `${DERIVATIVES_ANOMALY_DIR}`) under the global agents config folder (typically `~/.gemini/config/skills/` on Unix-like systems, or `C:\Users\<Username>\.gemini\config\skills\` on Windows) or workspace `.agents/skills` folder before running.*

### 3. Derive the per-ticker metrics

- **涨跌%** — calculated from `(last_price - prev_close_price) / prev_close_price * 100` (from #3).
- **主力净流入 (Smart Money Flow)** = `(capital_in_super + capital_in_big) - (capital_out_super + capital_out_big)` (from #1). Positive = main-force net buying.
- **散户净流入 (Retail Flow)** = `capital_in_small - capital_out_small` (from #1).
- **做空占比 (Short Ratio)** = `short_sell_ratio` (%) (from #2). High ratio (e.g. >25%) indicates significant short selling pressure.
- **期权大单异动 (Unusual Option Trade)** = sweep/block total premium, dominant strike/expiry, and call/put skew (from #4).
- **社区情绪 (Community Sentiment)** = Bullish vs Bearish percentage ratio, total posts count (from #5).

### 4. Classify each name (资金动向与聪明钱判定)

| Label | Trigger |
|---|---|
| 🟢 **多头确认** | price up **and** 主力净流入 (super+big) > 0 **and** call option sweeps/unusual premium dominates **and** short ratio is moderate/low. Strong, confirmed institutional buying. |
| 🔴 **空头压制 / 派发** | price down (or slightly up) **and** 主力净流入 (super+big) < 0 **and** put option sweeps dominate and/or short ratio is high (e.g. >25%). Clear distribution. |
| 🟡 **游资/散户主导** | price up **but** 主力净流入 ~0 or negative **and** 散户净流入 (small) strongly positive **and** community sentiment is hyper-bullish. Retail FOMO / momentum-driven, unconfirmed by institutions. |
| ⚖️ **双押 / 财报事件** | Option sweeps are high on both calls/puts **and** community sentiment is split/neutral **and** earnings date is close. Indicates directional hedging or straddle positioning. |

### 5. Output

- **One table per basket**, columns: `股票代码 | 股价 / 涨跌% | 主力净流入 (超大+大单) | 散户净流入 (小单) | 卖空占比 | 期权异动 (大单/sweep) | 综合判定`. 流入单位为对应市场货币（如 USD 或 HKD），百万（M），保留一位小数。
- Then a **cross-section synthesis**: who's the clean long, who's distributing, who's retail-driven/unconfirmed, who's event-hedging; and the **basket vs basket** comparison if more than one.
- A **社区/舆情 (Futu Community Sentiment)** line: Bull/Bear ratio, top viewpoint summary.
- Respond in the user's language (**Chinese** by default — see User Profile).

## Constraints

- **Read-only.** This is data presentation, never a trade recommendation, price target, or buy/sell call. Close with a one-line **非投资建议** note.
- **State the 口径 every time**: Futu OpenD actual stock-side capital distribution (super+big as main force, small as retail) + derivatives anomaly sweeps for smart money; daily short ratio; feed comments for community sentiment.
- **Don't fabricate** numbers or a retail/institutional split the feed doesn't provide. If OpenD is disconnected or a script errors, report it clearly and guide the user on how to run OpenD.
- This is a **read**, not the full structure flow — if the user then wants to *act* (size, pick a structure, model P/L), route to [`analysis.md`](analysis.md) and run the three-axes / bull-conviction checks there.

## Related

- [`../pitfalls/02-single-flow-not-smart-money.md`](../pitfalls/02-single-flow-not-smart-money.md) — one institutional order isn't edge.
- [`../pitfalls/17-dealer-flow-not-retail.md`](../pitfalls/17-dealer-flow-not-retail.md) — options flow is dealer hedging, not retail direction.
- [`../pitfalls/20-post-earnings-momentum-vs-fade.md`](../pitfalls/20-post-earnings-momentum-vs-fade.md) · [`../pitfalls/21-event-iv-vs-demand-iv.md`](../pitfalls/21-event-iv-vs-demand-iv.md) — pull flow + check the catalyst clock before any "fade / IV crush" call.
- [`../gamma-framework.md`](../gamma-framework.md) — add GEX (`type=greek-exposure`) for dealer-positioning context when asked.
- [`analysis.md`](analysis.md) — when the read turns into an actual trade decision.
