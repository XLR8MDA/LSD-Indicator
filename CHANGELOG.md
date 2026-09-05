# Changelog

All notable changes to the LSD Zone Indicator are documented here.

## [4.7.0] — 2026-09-05

Initial import into version control. Baseline corresponds to the existing
`indicator("LSD Zone Detector V4.7")` script, previously maintained as
`sample.pine` with ad-hoc versioned filenames.

### Included at baseline

- Supply/demand zone detection: marking candle + impulse window, FVG tagging,
  ATR-based maximum zone size, 70% overlap dedupe
- Zone state machine (0 hunting liquidity → 1 waiting BoS → 2 armed → 3 tapped)
- Pullback-run liquidity engine with liquidity levels and BoS markers
- Entry models: directional CLOSE, break of candle (BOC), HTF candle FLIP
- Session logic on UTC (Asia / London / NY / overlap) and the RD tradable window
- Trade registration with 1R–4R tracking; stats table by entry model and session
- Session trade monitor, live break-of-candle scanner, liquidity debug panel
- Zone death log with per-zone death reason and state
- Discord JSON alert payloads

### Known issues

- After a zone is consumed by an entry, the entry impulse candle can be re-marked
  as a new zone at effectively the same price. The 70% overlap dedupe in the zone
  creation section skips zones flagged invalid, so it does not suppress this.
