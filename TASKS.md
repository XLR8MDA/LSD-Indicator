# Upcoming Tasks

Backlog for the LSD Zone Indicator. The base indicator is
[`lsd-zone-indicator.pine`](lsd-zone-indicator.pine) at tag `v4.7.0` — treat it as the
reference build and land each item below as its own branch and release.

Line numbers refer to the v4.7.0 base. "Variant A" is the alternative build preserved in
commit `8beae64`, which carries several features this base does not.

---

## Defects

### 1. Zone re-marked at the same price immediately after an entry
**Priority: high** · confirmed in code

When an entry fires, `entryDemand()` / `entrySupply()` delete the live zone box and flag the
zone `demandInvalid` / `supplyInvalid = true`. One or two bars later the entry impulse candle
itself qualifies as a new marking candle, so a fresh zone is drawn at effectively the same
price and keeps extending — the zone appears to "stop, then restart".

The 70% overlap dedupe is supposed to suppress this, but it only compares against zones still
flagged valid:

```pine
if not array.get(supplyInvalid, ck)          // line 1131 (demand: line 1072)
    eTop = box.get_top(array.get(supplyBoxes, ck))
```

The just-consumed zone is excluded from the comparison, so `isOverlap_s` stays false.

Removing the guard is not a fix — the box behind an invalidated zone has been `box.delete()`d,
so `box.get_top()` on it is unsafe. The fix needs the consumed zone's top/bottom retained
separately (e.g. a short-lived "recently consumed levels" array with an expiry) that the
overlap check also scans.

**Open decision:** should suppression expire after a fixed number of bars, or persist until
price has moved a set distance (× ATR) away from the zone?

### 2. Liquidity debug panel silently switches zones
**Priority: low** · documented in the tooltip at line 52

The panel always reports whichever zone is newest with `state >= 1`, not a zone you pinned, so
it changes subject without warning. Add a way to pin the panel to one zone.

---

## Features to port from variant A (`8beae64`)

### 3. True HTF candle-flip detection
**Priority: high**

The base fires the FLIP model on `isHTFOpen` — any new 30m/1h candle opening. That is not a
flip. Variant A implements the real definition: the HTF candle opens, trades through its own
open, and closes back on the other side of it, tracked across the whole HTF candle via
`h30Open` / `h60Open` state. A also carries `h30Low` / `h30High` so it can require that the
flipping candle's wick actually reached the zone.

Port A's SECTION 5 and the `demandFlipSig` / `supplyFlipSig` / `demandFlipLow` /
`supplyFlipHigh` wiring into the state machines.

### 4. Entry log panel
**Priority: medium**

A's SECTION 19 plus `logEntry()` and its eight backing arrays: one row per entry signal that
fired — model, entry, SL, originating zone top, and whether it was taken, blocked by the
session filter, or excluded as wicked-through. Useful alongside the existing zone death log
for working out why a setup did or did not trade.

---

## Resolved

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

---

## Conventions

- One concern per branch, named `{type}/{short-description}`.
- Validate in the TradingView Pine Editor — zero compiler errors *and* zero warnings.
- Bump `README.md`, `CHANGELOG.md` and the git tag together; the README version must always
  match the newest tag.
