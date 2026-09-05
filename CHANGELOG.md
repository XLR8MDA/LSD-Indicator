# Changelog

All notable changes to the LSD Zone Indicator are documented here.

## [4.7.0] — 2026-09-05

Initial versioned baseline, corresponding to the `indicator("LSD Zone Detector V4.7")`
script previously maintained as `sample.pine` with ad-hoc versioned filenames.

### Included at baseline

- Supply/demand zone detection: marking candle + impulse window, ATR-based maximum zone
  size, 70% overlap dedupe
- FVG detection, shown as a gold zone border
- Zone state machine (0 hunting liquidity → 1 waiting BoS → 2 armed → 3 tapped)
- Pullback-run liquidity engine with liquidity levels and BoS markers
- Entry models: directional CLOSE, break of candle (BOC), HTF candle open (FLIP)
- Session logic on UTC (Asia / London / NY / overlap) and the RD tradable window
- Trade registration with 1R–4R tracking; stats table by entry model and session,
  filtered by `monthOffset`
- Session trade monitor, live break-of-candle scanner, liquidity debug panel
- Zone death log with per-zone death reason and state
- Discord JSON alerts on entry, on zone invalidation, and on pre-arm taps

### Known issues

Tracked in [TASKS.md](TASKS.md). Most significant:

- A consumed zone can be immediately re-marked at the same price, because the overlap
  dedupe skips zones flagged invalid (TASKS.md item 1)
- The FLIP model triggers on any HTF candle *open*, not on a true flip (item 6)
- `alertFormat` and `fvgWindow` inputs are declared but never read (items 2, 3)

### Note on provenance

Two divergent builds both self-identified as "V4.7". This baseline is the build found in
the working tree. The alternative — which carries a full HTF flip state machine and an
entry log — is preserved in commit `8beae64`; the features worth keeping from it are
itemised in TASKS.md.
