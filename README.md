# LSD Zone Indicator

**Version: 4.7.1**

A TradingView Pine Script (v6) overlay indicator implementing the **LSD** methodology —
**L**iquidity + **S**upply/**D**emand — for forex intraday trading.

## What it does

The indicator marks supply and demand zones from an origin ("marking") candle plus a
directional impulse, then walks each zone through a state machine before it is allowed to
produce an entry:

| State | Meaning |
| ----- | ------- |
| 0 | Hunting liquidity |
| 1 | Waiting for break of structure (BoS) |
| 2 | Armed |
| 3 | Tapped / waiting for entry trigger |

Only an armed-and-tapped zone can fire. Three entry models are supported:

- **CLOSE** — directional close back out of the zone
- **BOC** — break of the prior candle
- **FLIP** — higher-timeframe candle open (true flip detection is pending — see TASKS.md)

Zones that are tapped before arming, that close inside themselves, or that trigger outside
the configured session are retired and greyed out rather than left live.

## Features

- Supply/demand zone detection with ATR-based size filter, overlap dedupe, and a
  cooldown so a consumed zone can't immediately re-mark at the same price
- FVG detection, tagged `(FVG)` in the zone label
- Pullback-run liquidity engine with on-chart liquidity levels and BoS markers
- Session logic (Asia / London / NY / overlap) on a **UTC** basis, with an "RD" tradable
  window (default 08:00–15:00 UTC) and a session dashboard
- Trade registration with 1R–4R target tracking, plus a trade-stats table broken down by
  entry model and session
- Live break-of-candle scanner and a liquidity debug panel
- Zone death log — why each zone died and which state it was in
- Unified alerts as Discord-compatible JSON embeds on entry signals

## Installation

1. Open TradingView → **Pine Editor**.
2. Paste the contents of [`lsd-zone-indicator.pine`](lsd-zone-indicator.pine).
3. **Save**, then **Add to chart**.

There is no build step, package manager or test runner — Pine Script is compiled by
TradingView itself. Validate changes by pasting into the Pine Editor and confirming there
are no compiler errors or warnings.

## Repository contents

- `lsd-zone-indicator.pine` — the indicator
- `pine-viewer.html` — standalone, dependency-free `.pine` viewer with syntax highlighting;
  open directly in a browser
- `CHANGELOG.md` — version history
- `TASKS.md` — upcoming tasks and known defects
- `CLAUDE.md` — working notes and conventions for this repository

Reference screenshots (`rdforex/`) and the source strategy transcripts (`transcripts/`) are
kept local and intentionally excluded from this repository — see `.gitignore`.

## Versioning

Semantic-ish versioning, tagged `vMAJOR.MINOR.PATCH`. The version stated at the top of this
README always matches the most recent git tag.

Work happens on branches named `{type}/{short-description}` (e.g. `fix/zone-redraw-after-entry`)
and lands on `main` via merge request.
