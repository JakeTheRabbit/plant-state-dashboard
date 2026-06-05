# From Telemetry to Intelligence
### A Plant-State Dashboard Philosophy for the Modern Grow Room

**A Legacy Ag white paper · v1.0 · 2026-06-04**

---

## Abstract

The instrumented grow room has won. We can measure air temperature, relative humidity, VPD, CO₂, PPFD, substrate water content, pore-water EC, root-zone temperature, pH, runoff, dosing volumes, and power draw — second by second, channel by channel. And yet the dashboards we built to display all of this have quietly become the bottleneck. They are gauge clusters: walls of live numbers and time-series graphs that show *what is happening* but never *what it means, what is about to happen, or what to do about it*. The human is forced to be the integrator — synthesizing fifteen graphs into a judgment, in real time, with expertise, often while tired.

This paper argues for a different design center. The next dashboard should not show sensors; it should reason about the plant. It should fuse many inputs over time, judge each reading against what is *normal for this cultivar on this day of this stage*, surface the leading indicators of trouble days before they become visible on the plant, and prescribe the next step — with its evidence and its confidence attached. Most of the time it should be quiet. We call this philosophy **Plant-State Intelligence**, and the surface it presents the **calm dashboard**.

---

## 1. The problem with the sensor dashboard

The conventional grow-room dashboard is a *telemetry dump*. Its implicit theory of operation is: "expose every measurement, and a skilled grower will know what to do." That theory fails in seven predictable ways.

1. **It shows the environment, but the grower cares about the plant.** Every sensor measures the plant's *surroundings* — air, root zone, light. None measures vigor, stress, or transpiration directly. There is an inference gap between "VPD is 1.6 kPa" and "the plant is about to close its stomata," and today the dashboard makes the human cross that gap, unaided, every time they look.

2. **It is reactive, not anticipatory.** A graph only looks bad *after* the plant is already stressed. By the time a line crosses a threshold, the damage — salt accumulation, a stalled dryback, a missed irrigation — has been accruing for hours or days.

3. **It has no memory and no context.** A graph shows *now* against a fixed time window. It does not know that 1.4 kPa is fine in late flower but punishing in early veg, or that this EC climb is abnormal *for day 23*, or that this dryback rate is slower than the same cultivar ran last cycle. Judgment requires a baseline; the dashboard supplies none.

4. **Threshold alarms cry wolf.** Static high/low limits fire on transient blips — a door opened, a lights-on spike — and growers learn to ignore them. Alarm fatigue means the one alert that mattered is buried in noise.

5. **It thinks one sensor at a time.** Real plant health is multivariate and temporal. "Leaf temp up" alone is noise; "leaf temp up *and* transpiration proxy flat *and* substrate dryback unusually deep" is a diagnosis. Single-channel widgets can't express the cross-signal patterns where the truth lives.

6. **It imposes cognitive load as a feature.** More graphs are sold as more visibility. In practice they are more for the human to scan, correlate, and hold in working memory. The dashboard competes with the plant for the grower's attention — and the plant should win.

7. **It stops at symptoms.** Even a perfect reading of a symptom (tip burn appearing) is a lagging confession of a problem that began upstream (pore-water EC creeping past the plant's tolerance four days earlier). The dashboard names the symptom and leaves the diagnosis, the cause, and the fix to the human.

The summary indictment: **we built instruments and called them intelligence.** A cockpit full of gauges is not a co-pilot.

---

## 2. The shift: six inversions

Plant-State Intelligence is defined by inverting six assumptions baked into the sensor dashboard.

| | From (gauge cluster) | To (calm dashboard) |
|---|---|---|
| **1. Object** | Instrumentation — show the environment | Inference — estimate the plant's state |
| **2. Reference** | Fixed thresholds | Learned trajectories / per-stage baselines |
| **3. Breadth** | One signal per widget | Multi-input sensor fusion |
| **4. Timing** | Lagging indicators (symptoms) | Leading indicators (precursors) |
| **5. Output** | Alerting — "a number moved" | Prescription — "do this, by when, or expect that" |
| **6. Posture** | Always-on wall of graphs | Exception-based; quiet by default |

None of these throws away the raw data. The sensors and graphs still exist — they move to the basement, available on drill-down for the expert and the post-mortem. What changes is what occupies the grower's primary attention: not the measurements, but the *meaning*.

---

## 3. Design principles

- **Infer the plant, instrument the room.** The headline unit of the dashboard is plant state, not a sensor channel.
- **Normal is a trajectory, not a line.** Every value is judged against the expected envelope for this cultivar × stage × point-in-photoperiod, learned from your own historical runs.
- **No signal stands alone.** Conclusions come from fused evidence across environment, substrate, irrigation, vision, and equipment.
- **Lead, don't lag.** Optimize for the earliest reliable precursor, not the cleanest confirmation.
- **Prescribe, with a number.** An advisory that doesn't name the action and the setpoint is just a prettier alarm.
- **Show your work.** Every advisory carries its evidence chain and a confidence level. Trust is earned by explainability.
- **Silence is the product.** A calm screen means "nothing needs you." The dashboard speaks only when it has something worth saying.
- **Automation is opt-in and gated.** The system advises first. It acts only on what the grower has explicitly licensed it to act on — and never on the irreversible or the expensive without a human.

---

## 4. The plant-state model — what we actually infer

The pipeline estimates four interacting states. The first two are measured directly; the third is the point of the whole exercise; the fourth keeps the system honest.

**A. Environmental state.** Air temp, RH, VPD, CO₂, PPFD/DLI. Already well covered — but reframed as *integrals and rates*, not instants: VPD-hours accumulated today, DLI to date, rate of RH recovery at lights-off.

**B. Substrate state.** Water content, pore-water EC, root-zone temp, and the *derived* crop-steering metrics that matter more than any raw value: dryback depth, dryback rate, field-capacity recovery, shot-to-shot WC response, the generative/vegetative balance the irrigation is actually producing.

**C. Plant physiological state — the inference layer.** Not measured, *estimated* by fusing A and B (and E and F below):
- **Transpiration proxy** — modeled from VPD, light, and substrate WC response; the closest thing to "is the plant breathing."
- **Stress index** — a composite flag for stomatal-closure conditions (VPD too high, dryback too deep, root zone too hot).
- **Vigor / stacking trajectory** — is the plant on the growth curve this cultivar held last cycle at this stage?
- **Steering response** — did the plant respond to the generative/vegetative push the way the irrigation strategy intended?

**D. Operational & equipment state.** Dosing events, valve/emitter behavior, dehumidifier and HVAC current draw and recovery slopes, and — critically — **sensor health itself**. A flatlined or drifting sensor is a first-class signal: the system must know when it is blind rather than infer confidently from bad data.

**E. Vision state (where cameras exist).** Canopy imagery — already on-site for security — becomes a horticultural input: canopy color and uniformity, wilt/droop at lights-on, height/stacking over days, early discoloration. Slow visual drift the human eye misses across daily walk-throughs is exactly what time-lapse differencing catches.

The output of this layer is not fifteen numbers. It is a short list of **named conditions**, each with a confidence and an evidence chain.

---

## 5. Architecture

A six-layer pipeline. It maps cleanly onto a Home Assistant–centered stack; most operations already have layers 0–1 and don't know it.

```
0  INGEST        All raw streams → one timebase.
                 Climate, substrate (TEROS/WC-EC-temp), irrigation/dosing
                 events, power, cameras, HVAC/dehu telemetry.

1  DERIVE        Raw → meaningful. VPD, dryback % and rate, DLI, VPD-hours,
                 shot response, recovery slopes, integrals. (Template/
                 derived sensors.)

2  BASELINE      Per-cultivar × stage × photoperiod-phase expected
                 envelopes. Seeded from horticultural priors (e.g. Athena
                 targets); refined from your own historical runs.

3  INFER         Fuse layers 1–2 into plant-state estimates and named
                 conditions, each with confidence + evidence. Rules +
                 anomaly detection + (optionally) an LLM reasoning pass
                 over the fused feature vector.

4  PRESCRIBE     Map condition → concrete action: the setpoint change,
                 the irrigation tweak, the service task — with a deadline
                 and an expected outcome if ignored.

5  PRESENT       The calm dashboard. Status, watchlist, advisories,
                 with evidence drill-down and raw graphs demoted beneath.

       (5b OPTIONAL, GATED) CLOSED LOOP — auto-apply only the low-risk,
        explicitly-licensed actions; human-approve everything else.
```

A note on layer 5b, and a deliberate one. The advisory-first, human-in-the-loop posture is not a limitation to be engineered away — it is the design. An operation can run permanently at "advise only" and capture most of the value. Autonomous control is earned, channel by channel, after the advisories have proven themselves correct over many cycles. *The intelligence moves to the dashboard long before the actuation does.*

---

## 6. The new dashboard surface

What the grower opens to. Four zones, in priority order. On a good day, three of them are empty.

**① Headline — one line of plant truth.**
> *"Flower Day 24 · Room 3 · On-track. Steering generative as intended. No action needed."*

Not a number. A judgment, in plain language, with a status color. This is 90% of what a busy grower needs 90% of the time.

**② Watchlist — drifting, not yet wrong.**
The early-warning shelf. Things trending out of envelope but not yet at action threshold — the precursors. Each is a sentence, not a graph:
> *"Pore-water EC up 0.7 mS/cm over 3 days against flat feed. Watching — salt accumulation pattern."*

This zone is the whole point. It exists to make the *next* zone rare.

**③ Advisories — act now, here's how.**
The only zone that should ever interrupt. Each advisory is prescriptive and time-bound:
> ⚠️ **Reduce dryback target 3% in Room 3 (Day 24).**
> Pore-water EC has climbed 0.9 mS/cm over 4 days while feed EC held flat — dryback is too aggressive and salts are concentrating in the root zone. **Tip burn likely within ~48h** if unaddressed. *Confidence: high. [Show evidence]*

**④ Evidence & raw — show your work / the basement.**
Every advisory expands to its evidence chain: the fused signals, the baseline it violated, the historical precedent. Beneath that, the full raw graph wall still lives — for the expert who wants it and the post-mortem that needs it. Demoted, not deleted.

The inversion in one sentence: **the old dashboard was 100% zone ④; the new one leads with ①–③ and keeps ④ in the basement.**

---

## 7. Worked scenarios

Four real failure patterns, shown as the old dashboard handles them vs. the new one.

**Scenario A — The silent salt creep.**
*Old:* An EC graph slowly rises. It's still "in the green band." Nobody notices until tip burn appears on day 26; now you're reacting to damage.
*New:* Layer 3 fuses *rising pore-water EC* + *flat feed EC* + *dryback deeper than this cultivar's baseline* into one condition on day 22. Advisory: reduce dryback 3% or add a maintenance shot; tip burn projected ~48h out. Caught four days upstream of the symptom.

**Scenario B — The irrigation that didn't land.**
*Old:* Dosing controller reports "irrigation ran ✓." The dashboard shows a green checkmark. But an emitter clogged and one row got nothing — invisible until that row wilts.
*New:* Layer 3 cross-checks *dosing event fired* against *substrate WC response*. WC didn't rise in zone 4. Advisory: "Irrigation fired but zone 4 substrate didn't respond — suspect clogged emitter / channeling, check row 4 line." A single-sensor dashboard literally cannot see this; it requires fusing the command with its effect.

**Scenario C — Stealth morning stress.**
*Old:* Every instantaneous value looks fine all day. But RH drops too fast at lights-on, VPD spikes for 40 minutes each morning, the plant won't wake up cleanly, and stacking quietly slows. No single reading ever looks alarming.
*New:* Camera shows lights-on droop; transpiration proxy stalls in the first hour; VPD-hours in the morning window run above baseline three days running. Advisory: ramp RH down more slowly / delay first irrigation; cumulative cost is lost stacking. A pattern only visible across *time* and *fused* signals.

**Scenario D — The dehumidifier dying slowly.**
*Old:* Night RH control is fine — until one night mid-flower the dehu fails outright, RH spikes, and you're fighting mold at the worst possible time.
*New:* Layer 0 watches equipment telemetry. The dehu's current draw is creeping up for the same RH delta, and its nightly RH-recovery slope is degrading cycle over cycle. Advisory days ahead: "Dehumidifier recovery degrading — likely compressor wear, service before it fails under load." The failure is predicted, not survived.

The common thread: every catch comes from **fusing signals over time against a baseline** — precisely what a wall of independent live graphs cannot do, and a human staring at them cannot reliably do either.

---

## 8. Adoption path — crawl, walk, run

This is not a boil-the-ocean rebuild. Each stage ships value and earns the next.

- **Stage 0 — Telemetry (today).** Raw graphs. The starting point.
- **Stage 1 — Derive.** Compute the metrics that actually matter — dryback %/rate, DLI, VPD-hours, shot response, recovery slopes. Still graphs, but *meaningful* ones. Low effort, immediate clarity.
- **Stage 2 — Baseline & go quiet.** Establish per-stage/cultivar envelopes (start from known-good targets). Switch from threshold alarms to exception-based attention. The dashboard gets calm. This single step kills alarm fatigue.
- **Stage 3 — Fuse & diagnose.** Combine signals into named conditions with confidence and evidence. The watchlist and advisory zones come alive.
- **Stage 4 — Prescribe.** Attach the specific action to each condition. The dashboard becomes an advisor.
- **Stage 5 — Close the loop (opt-in, gated).** License the system to auto-apply only the low-risk, well-proven actions; everything irreversible or expensive stays human-approved.

Most of the payoff lands by Stage 3. Stage 5 is optional and should remain so for as long as the operator wants — advisory-first is a stable, valuable end state, not a stepping stone you're obligated to leave.

---

## 9. Trust, confidence, and failure modes

An advisory system that is wrong and confident is worse than no system. Five guardrails are non-negotiable.

- **Cold-start honestly.** With thin history, seed baselines from horticultural priors, widen the confidence bands, and label outputs "still learning." Never project false certainty from one cycle of data.
- **Make false advisories cheap to correct.** Every advisory is dismissable and markable as a false positive; those marks tune the baselines. Track advisory precision as a first-class metric (see §10). Trust is a balance you spend with every wrong call and earn with every right one.
- **Keep the human in the loop where it counts.** Irreversible or costly actions are always human-approved. The system's job is to make the decision obvious, not to make it alone.
- **Treat the system's own blindness as a signal.** A drifted, noisy, or flatlined sensor is itself an advisory ("EC probe in Room 2 reads implausibly flat — suspect failure, EC-derived advisories paused"). The system must know when it cannot see and say so, rather than infer confidently from garbage.
- **Never be a black box.** Every conclusion expands to its evidence. A grower who can't see *why* will, correctly, stop trusting the *what*.

---

## 10. Measuring success

The dashboard itself becomes measurable. Targets, run over run:

- **Lead time** — hours/days between an advisory and when the problem would have become visible on the plant or in a raw graph. *Higher is better; this is the core KPI.*
- **Surprises** — issues that reached visible plant damage with *no* prior advisory. *Drive to zero.*
- **Advisory precision** — acted-upon advisories ÷ total; and the false-positive rate. *Higher precision = more trust = more action.*
- **Dashboard dwell time** — minutes/day the grower spends staring at the dashboard. *Lower is better.* The calm dashboard should *return* attention to the plants.
- **Decisions surfaced** — count of real, actionable decisions raised per week. The dashboard's output is decisions, not pageviews.
- **Outcome variance** — yield and quality consistency cycle over cycle. The end goal of catching drift early is fewer bad surprises at harvest.

If the new dashboard is working, the grower looks at it *less*, is surprised *less*, and harvests *more consistently*.

---

## 11. Closing

The sensor dashboard answered "can we measure it?" — and the answer is now an emphatic yes. That question is solved. The question that actually governs yield and quality is the one the gauge cluster never even asked: **"what does all of this mean, and what should I do next?"**

Plant-State Intelligence moves the dashboard from instrument to advisor. It infers the plant instead of displaying the room, judges against learned trajectories instead of fixed lines, fuses many signals over time instead of one at a glance, leads with precursors instead of lagging on symptoms, prescribes a next step instead of sounding an alarm, and — most radically — stays quiet until it has something worth saying.

The graphs don't go away. They go to the basement, where the expert and the post-mortem can still find them. What rises to the top is the thing that was always the point: keeping the crop on its trajectory, and catching the drift before it becomes damage.

---

*Working name for the system: **Plant-State Intelligence (PSI)**; the surface, the **calm dashboard**. Both are placeholders — substance over branding.*

*Next step if you want to build it: pick one room and one failure pattern from §7, implement Stages 1–3 for just that pattern in Home Assistant, and run it shadow-mode for a cycle against the existing dashboard. Prove the lead time on one advisory before scaling.*
