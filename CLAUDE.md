# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository purpose

This is a Pine Script (TradingView v6) forex trading indicator project, not a general software codebase. There is no build system, package manager, linter, or automated test suite — Pine Script indicators are validated by pasting them into the TradingView Pine Editor and checking for compiler errors/warnings there, or against the TradingView Pine Script v6 reference (https://www.tradingview.com/pine-script-reference/v6/).

## Structure

- `lsd-zone-indicator.pine` — the main indicator, "LSD Zone Detector" (Liquidity + Supply/Demand zone detector, overlay indicator, `@version=6`). Organized into numbered `SECTION` blocks (grep for `SECTION` to get a table of contents): inputs, forex/accuracy-zone detection, session logic, session dashboard, HTF candle-open detection, zone detection engine, liquidity engine, draw helpers, storage, trade registration, open-trade tracker, zone creation, demand/supply state machines, trade stats table, session trade monitor, live break-of-candle scanner, liquidity debug panel, unified alerts, and zone death log.
- `pine-viewer.html` — a standalone, dependency-free HTML tool (drag-and-drop or file-open) for viewing `.pine` files with basic syntax highlighting (comments, strings, hex colors, numbers, keywords, namespaces like `ta.`/`math.`/`strategy.`, and series variables like `close`/`bar_index`). Open directly in a browser; no build step.
- `rdforex/` — reference screenshots (chart setups, zone examples) used while iterating on the indicator's logic.
- `transcripts/` — source material the trading strategy is derived from: course/video transcripts (supply & demand strategy, "A+ Trading Checklist"), a trade log (`rd_trades.txt`), and `liquidity-sd-checklist.html` (a standalone rendered checklist).

## Working with lsd-zone-indicator.pine

- Versioning: past iterations have used suffixed filenames (`sample_script_v4.10.2.pine`, `v4.11`, `v4.12.1`, etc. — visible in `.claude/settings.local.json` permission history) before consolidating to `lsd-zone-indicator.pine`. When making a significant change, prefer editing in place and confirm with the user before introducing a new versioned filename.
- Pine Script v6 scoping: variables must be declared with `=` before first use in a block; do not use `:=` for a first-time declaration inside a nested block (`if`/`for`/`while`) — this causes the `Undeclared identifier` compiler error. See project memory `pine-v6-ce10272-undeclared-identifier` for the specific pattern to avoid.
- The indicator relies on TradingView's box/line/label draw-object counts (`max_boxes_count`, `max_lines_count`, `max_labels_count` — currently 500 each in the `indicator()` call); if adding new persistent drawings, check these limits aren't silently exceeded.
- Session times (`rdSession`, London/NY/overlap logic) are UTC-based; keep new session-dependent logic consistent with the existing UTC convention rather than local time.
- The script emits alerts in "Discord JSON" format (`alertFormat` input) — if extending alert payloads, keep the existing JSON shape consumers may already parse.

## Terminology (from transcripts)

- "RD" = the course's session window (default 08:00–15:00 UTC, London into NY).
- "LSD" = Liquidity + Supply/Demand (the zone-detection methodology the indicator implements).
- Zone lifecycle states referenced in code/comments: 0=hunting liquidity, 1=waiting BoS (break of structure), 2=armed, 3=tapped/waiting entry.
