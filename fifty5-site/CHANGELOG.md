# Changelog

All notable changes to the Fifty5 website (published at `general.fifty5trades.here.now`) are documented here.

## 2026-09-06

### Added
- Initial site build: the TradingAsHobby course (18 chapters) as a single-page app with a
  sidebar, published to here.now under the `fifty5trades` workspace.
- Home, Community & Brokers, and Indicator pages linking out to the Discord, YouTube
  (@Fifty5Speaks), Exness/Elefin broker referral links, and the LSD Zone Detector V2.5
  TradingView script.
- Real brand logos: text wordmark ("Fifty5 Trades") in the header/sidebar in place of an
  image logo, plus brand-colored icons/buttons — YouTube (red), Discord (blurple), Exness
  (gold), Elefin (blue) — sourced via Simple Icons and DuckDuckGo's favicon service.
- Redesigned visual identity from an imported Claude Design mockup: navy/white two-tone
  layout, Archivo (italic display) + JetBrains Mono typefaces, hard-edged TradingView-style
  UI instead of the original soft/rounded template.
- Rotating trading-quote panel in the Home hero (Livermore, Seykota, Buffett, Elder, Soros).
- Sticky risk-disclosure banner ("educational purposes only") pinned to the top of Home.
- Community and Brokers merged into a single page, with each item (YouTube, Discord,
  Exness, Elefin) shown in its own white rounded card.
- Collapsible sidebar course groups (expand/collapse per part, only the active part open
  by default) with a working chapter search filter.
- Course restructured into three parts: **Trading Psychology** (chapter 1 alone),
  **Trading Basics** (candles, price action, structure, risk — chapters 2–6), and
  **The LSD Model** (chapters 7–18, unchanged).
- Interactive probability/risk toolset embedded in the Trading Psychology chapter: a
  breakeven win-rate calculator with a live SVG chart, a maximum-consecutive-losses
  calculator with a win-rate table, a projected equity curve, and a win-rate × reward:risk
  profitability matrix — all driven by shared inputs.
- Responsive breakpoints for mobile/tablet (header, hero, broker rows, sidebar height).

### Changed
- Rewrote all course copy in plain, beginner-friendly language (explaining terms instead of
  just naming them) for viewers with zero prior trading knowledge.
- Hero headline/tagline simplified from "TRADE. TEST. PROVE." to "LEARN. TRADE. GROW." with
  a friendlier subhead.
- Home hero's stat strip (Chapters/Price/Method/Starts at) replaced with the rotating
  quote panel.

### Removed
- A separate full-screen "welcome" quotes splash screen — consolidated into the Home hero
  instead, so there is a single entry point (Home) rather than two.
- A short-lived experiment with automatic/permanent dark mode and a light-dark toggle —
  reverted back to the fixed navy/white two-tone design from the original redesign.

### Fixed
- Several headings that inherited dark text color while sitting on navy backgrounds
  (invisible/near-invisible text) on the Home, Community & Brokers, and Indicator pages.
- A white gap showing below short navy-only pages (Community, Indicator) — the page
  background now defaults to navy with a full-viewport minimum height per panel.
