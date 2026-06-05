# Plant-State Dashboard — Build Spec & Generation Prompt

> Reusable spec for generating **Plant-State Intelligence** dashboards (the "calm dashboard") from the
> *From Telemetry to Intelligence* white paper. Paste the **PROMPT** block into a capable code model, fill
> the `[FILL]` slots, and it will produce a single self-contained HTML dashboard that obeys the philosophy
> **and** dodges the build traps learned the hard way (Chart.js ↔ reactivity recursion, token leakage, etc.).

---

## 0. What this produces

A single self-contained `.html` file that:
- Leads with **inferred plant state**, not raw sensors.
- Is **calm by default** — speaks only when something needs the grower.
- Surfaces **advisories** (prescriptive: next step + confidence + evidence) and a **watchlist** (leading indicators).
- Demotes raw numbers/graphs to a drill-down "basement".
- Runs live (Home Assistant REST) **or** on seed data, and feels alive (polling + smooth updates).
- Is responsive (mobile bottom-nav → desktop sidebar) and accessible (WCAG AA, reduced-motion, optional read-aloud).

---

## 1. Philosophy (the load-bearing rules — do not violate)

1. **Infer the plant, instrument the room.** Headline unit = plant state, never a sensor channel.
2. **Normal is a trajectory, not a line.** Judge every value against an expected envelope for *cultivar × stage × point-in-photoperiod*. Bands, not fixed thresholds.
3. **No signal stands alone.** Conclusions fuse environment + substrate + irrigation + equipment (+ vision).
4. **Lead, don't lag.** Optimize for the earliest reliable precursor.
5. **Prescribe with a number.** An advisory without a named action + setpoint is just a prettier alarm.
6. **Show your work.** Every advisory carries an evidence chain + a confidence level.
7. **Silence is the product.** A calm/empty screen means "nothing needs you." Never fill space for its own sake.
8. **Automation is opt-in and gated.** Advise first. Never actuate irreversible/expensive things without a human.

**The six inversions** (gauge cluster → calm dashboard): instrumentation→inference · thresholds→trajectories · one-signal→fusion · lagging→leading · alerting→prescription · always-on→exception-based.

---

## 2. Mandatory information architecture

Four surface zones, in priority order. On a healthy day, zones 2–4 are empty/quiet.

| Zone | Purpose | Rule |
|---|---|---|
| ① **Headline** | One line of plant truth + status color | Plain language, not a number. 90% of what a busy grower needs. |
| ② **Watchlist** | Drifting, not yet wrong (precursors) | One sentence each. Exists to make ③ rare. |
| ③ **Advisories** | Act-now, prescriptive | The ONLY zone that may interrupt. Each = title + detail + **next step** + **confidence** + **evidence** (collapsible) + acknowledge/investigate. |
| ④ **Basement** | Raw readings + full graphs | Collapsed by default. Demoted, never deleted. |

Plus: **plant-state tiles/gauges** (inferred signals — see §4), a **zone/room grid**, and a **trends chart**.

Navigation: desktop collapsible left sidebar + sticky top header; mobile fixed bottom nav (4–5 items) + bottom-sheet modals. Command palette (⌘K / `/`). Optional AI co-pilot panel that answers from live context.

---

## 3. Visual design system

Pick ONE tier per build.

**Tier A — Premium operational (default).**
- Deep near-black bg (`#0a0c10`), subtle glassmorphism (`backdrop-blur`, `bg-white/[0.04]`, `border-white/8`), soft shadows, `rounded-2xl/3xl`.
- Palette: **emerald-400/500** healthy, **amber-400** warning, **red-500** critical, slate grays for text. Subtle green undertone (cannabis, not gimmicky).
- Status is **triple-coded**: icon + word + color (colorblind-safe), never color alone.
- Typography: Inter-like system sans, `tabular-nums`, 8px spacing grid.
- Micro-interactions: hover-lift on `@media(hover:hover)` only; honor `prefers-reduced-motion`.
- Charts: Chart.js v4, dark-themed, smooth; sparklines on cards.

**Tier B — Accessibility-first (dyslexia / floor staff).**
- High-legibility font (Verdana/Tahoma default + optional OpenDyslexic toggle), warm off-white on softened dark (never pure #fff on #000), big letter/word/line spacing, sentence case (no ALL-CAPS — destroys word shape), short ≤62ch lines, left-aligned.
- Gauges with the OK-band drawn in; zone water-tanks (fill = wetness); a schematic room-map that lights up where a hotspot is.
- Read-aloud (Web Speech API) on headline + advisories. Theme options (dark / cream / high-contrast) + text-size control, persisted to localStorage.

Both tiers: large touch targets (≥44px), visible focus states, real empty states.

---

## 4. Plant-state signals to surface (cannabis crop-steering)

Inferred/derived, not raw. Each with a status (ok/watch/warn/crit) judged vs band:
- **Plant stress** (CWSI, 0–1; <0.3 good)
- **Transpiration** (leaf-air Δ / transpiration cooling; "is it drinking")
- **Leaf VPD** vs band (lights-on only)
- **Dryback** now vs target % + rate
- **Pore EC (fused)** vs generative target + trend
- **Stress flags** (substrate / light-load binaries)
- Per-zone: VWC vs P3 floor, pore EC, phase (P0–P3), last-shot response

Use correct units: VPD kPa, PPFD µmol/m²/s, EC mS/cm, DLI mol/m²/day, dryback %. Bands default to Athena-style targets; make them editable config.

---

## 5. Rules engine contract

Advisories/watchlist come from a **pure rules array**, each rule a function returning `null` or:

```js
{
  sev: "crit" | "warn" | "watch",   // crit/warn -> Advisories; watch -> Watchlist
  icon: "fire",                      // maps to an inline-SVG icon
  confidence: "high" | "med" | "low",
  title: "Back-left corner is running hot",      // plain language
  detail: "Back-left is 27.5C vs 24.8C room — 2.7C hotter. Those plants stress first.",
  action: "Check AC throw / add a fan to the back-left corner. Re-check in 30 min.",
  evidence: "sensor.x=27.5  sensor.y=24.8  delta=2.7"   // shown on drill-down
}
```

Rules read derived state and FUSE signals (e.g. `pump_power > 100W AND all valves closed → dead-head`). Encode real failure modes, not single-threshold trips. A `computeHeadline(adv, watch)` collapses them into the one-line plant truth. Dedup + allow acknowledge.

---

## 6. ⚠ NON-NEGOTIABLE BUILD RULES (learned from real breakage)

These are not style preferences — skipping them produces broken dashboards.

1. **Keep Chart.js instances AND their data buffers OUT of the reactive system.**
   If a Chart object (or an array you hand to Chart) lives in Alpine/Vue reactive state, the two proxy systems
   wrap each other's proxies forever → `RangeError: Maximum call stack size exceeded`. Store charts and rolling
   history in **module-scope** variables (`let CHARTS={}; let HIST={...}`). Never pass a reactive array to
   `new Chart()` — pass a copy (`[...arr]` / `arr.map(...)`).

2. **Use double-quoted JS strings for any text containing apostrophes.** `"don't"`, not `'don\'t'`.
   Hand-escaping `\'` is the #1 source of silent syntax errors that blank the whole page.

3. **Balance every `new Chart(...)` call.** Nested chart option objects are brace-heavy; one missing `}` blanks
   the page. Count braces / run `node --check` on the extracted script before shipping.

4. **Token security (live-data builds).** The HA REST API needs a bearer token. Prefer the short-lived token
   from `localStorage.hassTokens` (present when embedded in HA, logged in — no secret in the file). Only fall
   back to an embedded long-lived token for standalone/kiosk use, and put a loud SECURITY comment at the top:
   the file is served with no auth, so anyone on the LAN who can reach the URL gets full HA control. Blank the
   token if it will only ever be viewed logged-in. Never expose a token-bearing file to the public internet.

5. **The system must know when it is blind.** On fetch failure, show a "can't reach HA — showing last readings,
   treat as blind" banner. A flatlined/implausible sensor is itself an advisory; don't infer confidently from garbage.

6. **Self-contained.** Only CDNs allowed (premium tier): Tailwind v4 browser, Alpine 3, Chart.js 4. Icons are
   inline SVG (Lucide-style). No build step, no other deps. (Accessibility tier: zero CDNs, fully offline.)

7. **Verify in a real browser before claiming done.** Syntax-check the script, load the page, confirm no console
   errors (especially stack-overflow), confirm charts render and live data flows. A 200 on the file is not proof.

---

## 7. Output & acceptance criteria

- Output **only** the complete HTML file, starting `<!DOCTYPE html>`.
- Realistic data object at top (live entity map for HA, or seed for demo). Accurate cannabis terminology/units.
- Everything live: numbers/charts update on an interval with smooth, reduced-motion-respecting animation.
- Cards/rows/zones clickable → rich modals/bottom-sheets (history, notes saved to localStorage, actions).
- An operator understands whole-room/whole-facility health in **under 5 seconds** on first glance.
- Calm, trustworthy, premium — never cluttered or toy-like. Mobile genuinely usable one-handed on the floor.

---

## 8. The PROMPT (copy, fill `[FILL]`, run)

```
You are a world-class UI/UX designer and senior front-end engineer who ships award-winning real-time
operational dashboards for GMP cannabis cultivation. Build the single best browser dashboard for:

  SCOPE:        [FILL: e.g. "one room, F2, 3 crop-steering zones" OR "whole facility, 6 rooms"]
  DATA SOURCE:  [FILL: "LIVE Home Assistant REST at http://HOST:8123/api/states, token via localStorage
                 fallback to embedded long-lived token" OR "seed data, simulated live updates"]
  DESIGN TIER:  [FILL: "A Premium operational" OR "B Accessibility-first"]
  ENTITIES:     [FILL: entity map / metric list with units, bands, P3 floors, EC target, etc.]

Follow the Plant-State Intelligence philosophy:
- Infer the plant, not the room. Calm by default — quiet unless something needs the grower.
- Information architecture, in priority order: ① Headline (one line of plant truth + status color)
  ② Watchlist (drifting, not yet wrong) ③ Advisories (prescriptive: next step + confidence + evidence,
  the only zone that may interrupt) ④ Basement (raw readings + graphs, collapsed). Plus plant-state gauges,
  a zone grid, a trends chart, and an AI co-pilot panel that answers from live context.
- Advisories come from a pure rules array that FUSES signals (encode real failure modes, e.g. pump dead-head =
  pump drawing power with all valves closed; corner hotspot; shallow dryback; CO2 high while dark; engine
  unhealthy; sensor health degraded). Each advisory: {sev, icon, confidence, title, detail, action, evidence}.

Tech (premium tier): Tailwind v4 browser CDN, Alpine 3, Chart.js 4, inline Lucide-style SVG icons. Self-contained.
Responsive: mobile bottom-nav + bottom-sheets → desktop sidebar + header. Command palette (Cmd-K). WCAG AA,
visible focus, prefers-reduced-motion, ≥44px targets.

NON-NEGOTIABLE (these break the page if ignored):
- Keep Chart.js instances AND their data arrays in MODULE scope, never in Alpine/reactive state, and never pass
  a reactive array into new Chart() — copy it. (Reactive proxy + Chart proxy = infinite recursion / stack overflow.)
- Use double-quoted JS strings for any text with apostrophes (no hand-escaped \').
- Balance every new Chart() option object.
- Token: prefer localStorage.hassTokens access_token; embedded long-lived token only as fallback, with a loud
  SECURITY comment (served no-auth on LAN = full HA control). Never expose publicly with a token set.
- On fetch failure, show a "can't reach HA — treat as blind" banner.

Make it feel like $200k–500k enterprise software while staying a lightweight single HTML file. An operator must
grasp whole-room health in under 5 seconds. Output ONLY the complete HTML file, starting with <!DOCTYPE html>.
```

---

*Companion to: `whitepaper.html` (From Telemetry to Intelligence). Reference builds: `f2.html` (live, single-room),
`facility.html` (premium, multi-room demo). Legacy Ag · 2026-06.*
