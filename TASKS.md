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

### 2. `alertFormat` input does nothing
**Priority: medium** · confirmed in code

`alertFormat` (line 107) offers "Plain Text" / "Discord JSON" / "Generic JSON", but the
identifier appears exactly once in the file — it is never read. Both emitters hardcode the
Discord embed shape:

```pine
alert('{"embeds":[' + array.join(bocQueue, ",") + ']}', ...)   // line 1848
alert('{"embeds":[' + array.join(msgQueue, ",") + ']}', ...)   // line 1932
```

Either wire the input up to all three payload shapes, or remove it. Any change must keep the
Discord embed shape byte-identical for existing webhook consumers.

### 3. `fvgWindow` input does nothing
**Priority: medium** · confirmed in code

`fvgWindow` (line 35, "FVG Search Window") is never read. The FVG scan in `findDemand()` /
`findSupply()` iterates `for j = winStart to m - 1` (line 352), where `winStart` derives from
`impulseWindow`, not `fvgWindow`. Either wire it in or remove the input.

### 4. FVG detection result is stored but never used
**Priority: low** · confirmed in code

`demandFVG` / `supplyFVG` (lines 704, 719) are pushed on zone creation (lines 1094, 1153) and
shifted on eviction, but never read back — there is no `array.get(demandFVG, ...)` anywhere.
FVG currently only affects the zone border colour at creation time (`fvgBorder`, lines 1087,
1146). Either consume the arrays or drop them.

### 5. Liquidity debug panel silently switches zones
**Priority: low** · documented in the tooltip at line 52

The panel always reports whichever zone is newest with `state >= 1`, not a zone you pinned, so
it changes subject without warning. Add a way to pin the panel to one zone.

---

## Features to port from variant A (`8beae64`)

### 6. True HTF candle-flip detection
**Priority: high**

The base fires the FLIP model on `isHTFOpen` — any new 30m/1h candle opening. That is not a
flip. Variant A implements the real definition: the HTF candle opens, trades through its own
open, and closes back on the other side of it, tracked across the whole HTF candle via
`h30Open` / `h60Open` state. A also carries `h30Low` / `h30High` so it can require that the
flipping candle's wick actually reached the zone.

Port A's SECTION 5 and the `demandFlipSig` / `supplyFlipSig` / `demandFlipLow` /
`supplyFlipHigh` wiring into the state machines.

### 7. Entry log panel
**Priority: medium**

A's SECTION 19 plus `logEntry()` and its eight backing arrays: one row per entry signal that
fired — model, entry, SL, originating zone top, and whether it was taken, blocked by the
session filter, or excluded as wicked-through. Useful alongside the existing zone death log
for working out why a setup did or did not trade.

### 8. FVG tag in the zone label
**Priority: low**

A's `zoneLabelText(isDemand, state, isFVG)` renders `DEMAND (FVG)`. The base signature drops
the `isFVG` parameter and shows FVG only as a border colour. Pairs naturally with item 4.

---

## Conventions

- One concern per branch, named `{type}/{short-description}`.
- Validate in the TradingView Pine Editor — zero compiler errors *and* zero warnings.
- Bump `README.md`, `CHANGELOG.md` and the git tag together; the README version must always
  match the newest tag.
