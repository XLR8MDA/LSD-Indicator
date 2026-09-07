# Changelog

All notable changes to the LSD Zone Indicator are documented here.

## [4.7.1] — 2026-09-05

### Removed
- `alertFormat` input — declared but never read; both alert emitters always emitted
  Discord embed JSON regardless of its value
- `fvgWindow` input — declared but never read; the FVG scan has always used
  `impulseWindow`'s window, not this one
- `alertOnInvalid` / `alertOnPreArmTap` inputs and their `invalidMsg()` / `preArmMsg()`
  builders — only entry-signal alerts (`alertOnEntry`) remain configurable
- Gold `fvgBorder` zone border — FVG is now indicated by a `(FVG)` tag in the zone label
  text instead, restored from variant A's `zoneLabelText(isDemand, state, isFVG)`

### Added
- `zoneCooldownBars` input (default 30, 0 disables): after a zone is consumed — entry
  taken, entry skipped, closed through, pre-arm tap, or aged out — a new zone can't be
  marked at the same price for this many bars.

### Changed
- Trade-stats month filter: replaced the `monthOffset` integer input with `statsMonthStr`
  / `statsYearStr` "Auto" dropdowns (variant A's original design). "Auto" resolves against
  `timenow` at runtime, working around Pine Script's `const`-only `input.*` defaults.

### Fixed
- `demandFVG` / `supplyFVG` were pushed on zone creation but never read back; they now
  drive the `(FVG)` zone-label tag at every state transition, not just at creation.
- Restored the wicked-through exclusion from variant A: an entry whose stop sits beyond the
  whole zone (the wick blew straight through rather than cleanly tapping and rejecting it)
  is shown on chart tagged "(wicked through — not counted)" but no longer registered into
  the trade-stats table.
- A consumed zone (entry taken, entry skipped, closed through, pre-arm tap, or aged out)
  could be immediately re-marked at the same price, because the 70% overlap dedupe in zone
  creation only compared against zones still flagged valid. The dying zone's top/bottom
  is now retained for `zoneCooldownBars` bars and checked by the same dedupe.

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
