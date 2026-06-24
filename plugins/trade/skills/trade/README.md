# Trade

Multi-leg options trading assistant — concrete strikes, IV-aware structures, probability-weighted scenarios. Single skill with four subcommands, modeled on the [`pbakaus/impeccable`](https://github.com/pbakaus/impeccable) pattern.

## Commands

```
/trade setup                           # scaffold a personal knowledge directory
/trade import <file_path>              # parse one PDF / screenshot / text artifact into YAML
/trade report [tickers]               # daily capital-flow / 资金流向 read
/trade analysis [ticker | situation]   # default — trade analysis flow
/trade <natural language>              # any unrecognized first word routes to analysis
```

Each subcommand has its own reference file under `references/commands/`. The main `SKILL.md` carries always-on context (Hard Rule, Response Rules, Core Principles, Structure-to-Regime matrix) plus the routing logic.

## Triggers

- Trade analysis requests, options strategy recommendations, post-mortems
- Mentions of multi-leg structures: Jade Lizard, bull put / bear call spread, iron condor, diagonal, calendar
- Earnings positioning, IV / IV crush, channel checks, AH price action
- Any single-stock options play in a US-equity context
- Personal-knowledge management: "save this substack post", "parse this tweet screenshot", "set up my trade knowledge"

Full trigger list in the `description` field of `SKILL.md`.

## Platform

**CLI only** — primary market data via TradingView desktop reader (`finance-data-providers:tradingview-reader`); fallback to Futu OpenD API (`futuapi` skill) for options chains/Greeks, historical K-lines, capital flow/distribution, daily short selling, valuation percentiles, analyst ratings, and community sentiment (via `futu-comment-sentiment`).

## Setup

1. Make sure FutuOpenD client/gateway is running locally on `127.0.0.1:11111` (see `futuapi` or run `/install-futu-opend` to install/setup).
2. Install the `finance-data-providers:tradingview-reader` skill for TradingView quotes and basic options data.
3. (Optional) Run `/trade setup` once to scaffold a personal knowledge directory for substack posts, X / twitter threads, and writedowns.

## Reference Files

### Always-relevant frameworks

| File | Description |
|---|---|
| `references/strategies.md` | Structure-to-regime matching, LEAPS stock replacement, setup checklist, position management |
| `references/gamma-framework.md` | Dealer GEX + options chain + IV term + flow → multi-factor probability map |
| `references/price-action-framework.md` | Orderbook microstructure mental model — buy/sell imbalance, vacuum zones, consensus shifts |

### Subcommand references (lazy-loaded by the router)

| File | Subcommand |
|---|---|
| `references/commands/setup.md` | `/trade setup` workflow |
| `references/commands/import.md` | `/trade import` workflow (raw artifact → YAML) |
| `references/commands/report.md` | `/trade report` workflow (capital-flow / 资金流向 read) |
| `references/commands/analysis.md` | Default analysis preflight + situation → reference map |

### Lazy-loaded library

| File | Description |
|---|---|
| `references/pitfalls/index.md` | Index of 27 trading pitfalls (severity-tagged, lookup by trade type) |
| `references/pitfalls/NN-*.md` | One file per pitfall — loaded only when relevant |
| `references/ticker/index.md` | Index of closed trade case studies |
| `references/ticker/<name>.md` | One file per case study (INTC, Mag-7, APP, NOK, TSEM, CBRS, SNOW, MDB, VIX, SATS, 6981) |

### Templates (used by `/trade setup`)

| File | Copied to |
|---|---|
| `references/commands/templates/knowledge-index.md` | `<knowledge>/index.md` |
| `references/commands/templates/knowledge-README.md` | `<knowledge>/README.md` |
| `references/commands/templates/substack-template.yaml` | `<knowledge>/substack/_template.yaml` |
| `references/commands/templates/twitter-template.yaml` | `<knowledge>/twitter/_template.yaml` |
| `references/commands/templates/writedown-template.md` | `<knowledge>/writedowns/_template.md` |

## Coverage

- 27 analytical pitfalls covering consensus anchoring, flow misreading, IV crush traps, T+1 reverse drift, LEAPS vega tax, manipulator-tape recognition, channel-check sample bias, AH order-book fades, demand-IV vs event-IV, vega-axis sanity checks, hazard-rate discounting, capped-upside vs bull conviction, VIX mechanics, stock-consideration anchoring, retest-entry confirmation, and more.
- 11 detailed case studies (INTC, Mag-7, APP, NOK, TSEM, CBRS, SNOW, MDB, VIX, SATS, 6981) showing thesis evolution, structure selection, and post-mortem lessons.
- Structure-to-regime quick reference covering high/low IV regimes paired with directional / neutral / manipulator-tape views.
- Personal-knowledge layer for the user's own substack / X / writedown collection, auto-loaded on every analysis.
