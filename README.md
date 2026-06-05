# 🌿 Plant-State Dashboards — From Telemetry to Intelligence

**[Read the field guide → jaketherabbit.github.io/plant-state-dashboard](https://jaketherabbit.github.io/plant-state-dashboard/)**

The grow-room dashboard, reimagined. Stop showing sensors — reason about the plant. Lead with what it
*means*, prescribe the next step, and stay quiet until something needs you.

Part of the grow-room field-guide series.

---

## What's inside

| File | What it is |
|---|---|
| **[`index.html`](https://jaketherabbit.github.io/plant-state-dashboard/)** | The white paper as a site — philosophy, architecture, worked scenarios, screenshots, and the build prompt. |
| **[`demo.html`](https://jaketherabbit.github.io/plant-state-dashboard/demo.html)** | A **live, interactive** multi-room facility dashboard (seed data, runs in your browser). Click rooms, open the AI co-pilot, press `⌘K`. |
| `whitepaper.md` | The white paper in Markdown. |
| `plant-state-dashboard-spec.md` | The full build spec + a copy-paste generation prompt for producing your own dashboards. |
| `assets/` | Reference screenshots (facility desktop, single-room live, mobile). |

## The idea in one line

> The old dashboard was 100% raw graphs. The calm dashboard leads with a one-line **plant truth**, a
> **watchlist** of leading indicators, and prescriptive **advisories** (next step + confidence + evidence) —
> and demotes the raw sensors to a drill-down.

Six inversions: instrumentation → **inference** · thresholds → **trajectories** · one-signal → **fusion** ·
lagging → **leading** · alerting → **prescription** · always-on → **exception-based**.

## Build your own

`plant-state-dashboard-spec.md` (and §13 of the site) carry a copy-paste prompt. It encodes the philosophy,
a premium Tailwind + Alpine + Chart.js design system, a rules-engine contract, and the non-obvious build
traps — the worst being a charting library sharing a reactive proxy with the UI framework, which recurses
into a stack overflow. Fill the `[FILL]` slots and run it.

## 🔒 Security

The interactive `demo.html` runs on **seed data only — no credentials, no API calls**. The live
Home-Assistant-wired builds shown in screenshots keep their bearer token out of any published file
(localStorage token preferred; embedded token, if used, is never committed). Nothing in this repo contains
a token.

## Series

- 🌿 **Plant-State Dashboards** *(this one)*
- 🎛️ [Cannabis Grow-Room Levers](https://jaketherabbit.github.io/cannabis-grow-room-levers/)
- 🫧 [Hash Rosin Pressing Levers](https://jaketherabbit.github.io/hash-rosin-pressing-levers/)
- 📘 [Grow-Room Training](https://jaketherabbit.github.io/grow-room-training/)

---

© 2026 Legacy Ag · JakeTheRabbit · v1.0
