# Upcoming Tasks

Backlog for the LSD Zone Indicator. The base indicator is
[`lsd-zone-indicator.pine`](lsd-zone-indicator.pine) at tag `v4.7.0` — treat it as the
reference build and land each item below as its own branch and release.

Line numbers refer to the v4.7.0 base. "Variant A" is the alternative build preserved in
commit `8beae64`, which carries several features this base does not.

---

## Defects

### 1. Liquidity debug panel silently switches zones
**Priority: low** · documented in the tooltip at line 52

The panel always reports whichever zone is newest with `state >= 1`, not a zone you pinned, so
it changes subject without warning. Add a way to pin the panel to one zone.

---

## Features to port from variant A (`8beae64`)

### 2. True HTF candle-flip detection
**Priority: high**

The base fires the FLIP model on `isHTFOpen` — any new 30m/1h candle opening. That is not a
flip. Variant A implements the real definition: the HTF candle opens, trades through its own
open, and closes back on the other side of it, tracked across the whole HTF candle via
`h30Open` / `h60Open` state. A also carries `h30Low` / `h30High` so it can require that the
flipping candle's wick actually reached the zone.

Port A's SECTION 5 and the `demandFlipSig` / `supplyFlipSig` / `demandFlipLow` /
`supplyFlipHigh` wiring into the state machines.

### 3. Entry log panel
**Priority: medium**

A's SECTION 19 plus `logEntry()` and its eight backing arrays: one row per entry signal that
fired — model, entry, SL, originating zone top, and whether it was taken, blocked by the
session filter, or excluded as wicked-through. Useful alongside the existing zone death log
for working out why a setup did or did not trade.

---

## Resolved

### Zone re-marked at the same price immediately after an entry — fixed
Fixed in `chore/cleanup-alerts-fvg-monthly-stats`. When a zone was consumed — entry taken,
entry skipped, closed through, pre-arm tap, or aged out — `entryDemand()` / `entrySupply()`
and the state machines deleted the box and flagged it invalid, but the 70% overlap dedupe in
zone creation only compared against zones still flagged valid, so the just-consumed zone
dropped out of the comparison. The entry impulse candle itself could then immediately
re-qualify as a new marking candle at the same price, and a fresh zone would keep extending
right where the old one had stopped.

Added a `zoneCooldownBars` input (default 30, 0 disables) plus `retireDemandZone()` /
`retireSupplyZone()`, called at all five demand and five supply invalidation points, which
record the dying zone's top/bottom/bar into a short `deadDemandTop/Bot/Bar` /
`deadSupplyTop/Bot/Bar` array (capped at 40, same pattern as the zone death log).
`zoneOverlapsCooldown()` — shared by both sides — extends the existing overlap check to also
reject a new zone that overlaps ≥70% of its range with one of these still-cooling entries.
Suppression expires after `zoneCooldownBars` bars, chosen over an ATR-distance rule for
simplicity; revisit if a price-distance-based cooldown proves better in testing.

### `alertFormat` input did nothing — removed
Fixed in `chore/cleanup-alerts-fvg-monthly-stats`. The input offered "Plain Text" /
"Discord JSON" / "Generic JSON" but was never read — both alert emitters hardcode the Discord
embed shape. Removed rather than wired in, since nothing consumes the other two formats.

### `fvgWindow` input did nothing — removed
Fixed in `chore/cleanup-alerts-fvg-monthly-stats`. The FVG scan in `findDemand()` /
`findSupply()` has always used `impulseWindow`'s `winStart`, not `fvgWindow`. Removed the
input and its `grpFVG` group.

### FVG detection result was stored but never used — now read
Fixed in `chore/cleanup-alerts-fvg-monthly-stats`. `demandFVG` / `supplyFVG` are now read via
`zoneLabelText(isDemand, state, isFVG)`, restoring the `(FVG)` label tag on zone text at every
state transition. The gold `fvgBorder` zone border was dropped in favour of the label tag —
zones render with their normal demand/supply border colour regardless of FVG.

### Alert-on-invalid / alert-on-pre-arm-tap toggles — removed
Not originally tracked here. `alertOnInvalid`, `alertOnPreArmTap` and their `invalidMsg()` /
`preArmMsg()` builders were removed by request — only `alertOnEntry` remains configurable.

### Trade-stats month filter — restored Auto Month/Year dropdowns
Not originally tracked here. `monthOffset` (a single int walking back from the current month)
was replaced with the `statsMonthStr` / `statsYearStr` "Auto" dropdown pair used in variant A
— Pine Script requires `input.*` defaults to be `const`, so "Auto" is the sentinel resolved
against `timenow` at runtime. Add next year to the `statsYearStr` options list each January.

### Wicked-through entries counted in stats — excluded again
Not originally tracked here; found while restoring variant A behaviour. This base's
`entryDemand()` / `entrySupply()` had silently dropped variant A's wicked-through check —
every triggered entry was registered into the stats table via `registerTrade()`, even one
where the stop sits on the far side of the whole zone (the wick blew straight through
instead of cleanly tapping and rejecting it). Restored: such entries are still shown on
chart with SL/TP, but tagged "(wicked through — not counted)" and excluded from
`registerTrade()`, matching variant A.

**Heads up:** this is the second silent behavioural gap found between the two variants
(after the FVG-array one) — worth a closer side-by-side diff against variant A if more
missing behaviour turns up.

---

## Conventions

- One concern per branch, named `{type}/{short-description}`.
- Validate in the TradingView Pine Editor — zero compiler errors *and* zero warnings.
- Bump `README.md`, `CHANGELOG.md` and the git tag together; the README version must always
  match the newest tag.
