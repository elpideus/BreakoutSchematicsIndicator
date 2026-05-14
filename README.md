# Breakout Schematics

**Pine Script v6 · Version 5.0.0 · In Development**

A TradingView indicator that automatically detects and visualizes high-probability breakout schematics following a user-defined consolidation session. Built on a clean state-machine architecture for accuracy, readability, and easy expansion.

> **Note:** This project is under active development and not yet ready for production use.

---

## Table of Contents

- [How It Works](#how-it-works)
- [The Reference Session](#the-reference-session)
- [Key Thresholds](#key-thresholds)
- [Schematics](#schematics)
  - [SSA Schematic](#ssa-schematic)
  - [S1 Schematic](#s1-schematic)
  - [MUT Schematic](#mut-schematic)
- [Schematic Interactions](#schematic-interactions)
- [Inputs Reference](#inputs-reference)
- [Installation](#installation)
- [Roadmap](#roadmap)

---

## How It Works

The indicator operates in two phases each day:

1. **During the session** — it tracks the highest high and lowest low of a configurable reference session (default: the Asian session, 00:00–08:00 UTC) and draws a shaded box around it in real time.
2. **After the session closes** — the session high and low are locked as reference levels. Three independent state machines (SSA, S1, MUT) then monitor every subsequent candle for specific structured patterns and fire signals when those patterns complete.

Every schematic operates on the same underlying idea: price approaches or briefly exceeds the session boundary, then shows evidence of a directional commitment. The schematics differ in *how* they define that evidence.

---

## The Reference Session

The session is fully configurable. By default it is the Asian session (00:00–08:00 UTC), which is a widely-used low-volatility consolidation range before the London and New York sessions bring directional moves.

When the session closes:
- The **session high** becomes the upper reference level — the level a short setup must interact with.
- The **session low** becomes the lower reference level — the level a long setup must interact with.

The session is drawn as a shaded box on the chart. Each new session resets all state machines, so patterns from a previous day do not carry over.

---

## Key Thresholds

All pattern detection relies on two pip-based thresholds. These ensure signals are not triggered by noise (sub-pip fluctuations) and that meaningful price movement is required at each step.

| Threshold | Value | Used For |
|-----------|-------|----------|
| **Breakout threshold** | 1.0 pip | Whether a candle has genuinely broken the session boundary |
| **Continuation threshold** | 0.6 pip | Whether price has meaningfully extended or reversed during pattern phases |

Pip size is derived automatically from `syminfo.mintick × 10`, which is correct for most instruments including standard FX pairs, JPY pairs, indices, and commodities. You can override it manually if needed.

---

## Schematics

### SSA Schematic

**Session Spike Analysis** — detects a wick-based fake-out beyond the session boundary followed by a return into the range.

The core idea is that price spikes briefly through the session high (or low) without the candle *body* closing through it — a classic liquidity grab or stop hunt. The SSA schematic waits for that spike, watches for any extension, and then fires a signal when price reverses back below (or above) the spike extreme.

#### How It Triggers (Short Example)

Imagine the session high is at **1.0800**. After the session closes:

**Step 1 — Breakout candle (A):** A candle's wick pushes above 1.0800 + 1 pip = **1.0801**. Critically, the candle *body* does not close above 1.0800 + 1 pip — the close stays below. The wick high is recorded (say **1.0810**) and the candle low is recorded (say **1.0790**). This is candle A.

```
      |       ← wick above 1.0801 (breakout)
   ┌──┴──┐
   │     │   ← body stays below 1.0801 (no body close above)
   └──┬──┘
      |
──────────── session high at 1.0800
```

**Step 2 — Continuation (B):** A subsequent candle's wick reaches *higher* than candle A's high by at least 0.6 pip (so above 1.0810.6). This becomes the new candle A — its high and low replace the previous ones. This step can repeat multiple times if price keeps extending higher. Each extension updates the reference to the most recent extreme.

```
         |   ← new candle extends 0.6+ pip above prior high → new candle A
      |
   ┌──┴──┐
   │     │
   └──┬──┘
      |
──────────── session high at 1.0800
```

**Step 3 — Confirmation (C):** After the last extension, price must stop extending and then the candle body must close *below* candle A's low by at least 0.6 pip. This confirms the spike was a fake-out and the market is heading back down.

```
      |
   ┌──┴──┐
   │     │
   └──┬──┘
      |
   ┌──┴──┐
   │     │   ← body closes 0.6+ pip below candle A's low → SSA Short signal fires ▼
   └──┬──┘
      |
──────────── session high at 1.0800
```

The signal label **SSA Short** (red, pointing down) is placed above the confirmation candle.

#### Invalidation

If any candle's *body* closes more than 1 pip above the session high (for the short setup), the entire setup is cancelled immediately. This means the market has genuinely broken out in the short direction rather than faking — there is no reversal to trade.

#### Long Variant

The long version is the exact mirror image. Price must wick below the session low by 1+ pip without the body closing through, optionally extend lower on subsequent candles, and then a candle body must close 0.6+ pip above candle A's high. Signal: **SSA Long** (green, pointing up).

---

### S1 Schematic

**Session 1** — detects a *body* breakout beyond the session boundary that is immediately followed by a reversal back through the breakout candle's body edge.

Where SSA requires the body to stay inside the session range during the breakout candle, S1 requires the opposite: the body must *close through* the boundary. The signal fires when price reverses back through that breakout candle's body edge — a failed breakout confirmed by the close.

#### How It Triggers (Short Example)

Imagine the session high is **1.0800**. After the session closes:

**Step 1 — Breakout candle (A):** A candle's *body* (the open-to-close range, ignoring wicks) closes more than 1 pip above the session high — the body high exceeds **1.0801**. The wick high and the body low of this candle are both recorded. For example, wick high = **1.0820**, body low = **1.0803**.

```
      |       ← wick high: 1.0820
   ┌──┴──┐
   │     │   ← body closes above 1.0801 (body high = ~1.0815, body low = 1.0803)
   └──┬──┘
      |
──────────── session high at 1.0800
```

**Step 2 — Extension update:** If the very next candle's wick reaches *higher* than 1.0820, the reference updates — the new candle becomes the new candle A (new wick high and body low recorded). This can happen on any subsequent candle before the reversal fires.

**Step 3 — Reversal:** A subsequent candle's *body* closes below the reference candle A's body low by at least 0.6 pip (below **1.0803 − 0.6pip = 1.0797**). This confirms the breakout failed and price is heading down.

```
      |
   ┌──┴──┐
   │     │   ← reference candle A: body low 1.0803
   └──┬──┘
      |
   ┌──┴──┐
   │     │   ← body closes below 1.0803 − 0.6pip → S1 Short signal fires ▼
   └──┬──┘
      |
```

Signal: **S1 Short** (red, pointing down) placed above the reversal candle.

#### Key Difference vs SSA

| | SSA | S1 |
|---|---|---|
| Breakout candle body | Stays inside range | Closes outside range |
| Pattern type | Wick spike / fake | Body breakout failure |
| Reversal measured from | Wick extreme | Body edge of breakout candle |

#### Long Variant

Price must *body-close* more than 1 pip below the session low. Then a subsequent candle's body must close 0.6+ pip above the reference candle A's body high. Signal: **S1 Long** (green, pointing up).

---

### MUT Schematic

**Move-Under-Test** — a continuation/re-entry pattern that triggers *after* an SSA or S1 signal has already fired, using the extreme from that prior signal as a new reference level.

The concept: after an SSA or S1 signal prints, the market has shown it wants to move in that direction. But price often pulls back (or extends further) before continuing. MUT catches the moment price makes one more push beyond the prior signal's extreme and then reverses back — confirming the original directional bias.

#### How It Triggers (Short Example)

Suppose an **SSA Short** or **S1 Short** signal just fired. The spike high from that pattern (say **1.0820**) is saved as the *watch level*.

**Step 1 — Candle A:** A candle's *wick* breaks above the watch level by at least 0.6 pip (above **1.0820.6**). This candle becomes MUT candle A. Its high and low are recorded.

```
         |   ← wick breaks 1.0820 + 0.6pip → MUT candle A
      ┌──┴──┐
      │     │
      └──┬──┘
         |
─────────────── watch level: 1.0820 (prior SSA/S1 spike high)
```

**Step 2 — Extension update:** If the very next candle's wick goes *higher* than candle A's high by 0.6+ pip, candle A updates to the new candle. This can happen on the same candle that triggers a reversal — in that case, the reversal check uses the *previous* candle A's low, not the updated one.

**Step 3 — Reversal:** While in the "have candle A" state, the wick drops below candle A's low by at least 0.6 pip. The signal fires immediately.

```
         |
      ┌──┴──┐
      │     │   ← candle A: low = 1.0805
      └──┬──┘
         |
      ┌──┴──┐
      │     │
      └──┬──┘
         |       ← wick breaks 0.6+ pip below 1.0805 → MUT Short signal fires ▼
```

Signal: **MUT Short** (teal, pointing down) placed above the trigger candle.

#### Why MUT Exists

SSA and S1 signals are first-touch entries. MUT provides a second entry opportunity for traders who missed the original signal, or a confirmation signal for those who want to see one more failed push before committing. Because it requires the prior signal to exist, MUT never fires independently — it is always preceded by an SSA or S1 print in the same direction.

#### Long Variant

After an **SSA Long** or **S1 Long** signal, the watch level is the spike *low* from that pattern. MUT Long fires when a candle wicks below the watch level by 0.6+ pip (candle A), then the wick breaks back above candle A's high by 0.6+ pip. Signal: **MUT Long** (teal, pointing up).

---

## Schematic Interactions

The three schematics are not fully independent — they are designed to interact in specific ways:

**SSA disarms S1 (same direction):** When a candle qualifies as an SSA breakout candle (A), it cancels the corresponding S1 state machine. This is because the candle's body stayed inside the range (SSA condition), which means it did *not* meet the S1 body-breakout condition. Keeping both active would produce redundant signals.

**SSA and S1 both seed MUT:** When either an SSA or S1 signal fires (in either direction), the spike extreme from that signal is handed to the MUT state machine as its watch level. MUT then independently monitors for its two-step pattern using that level.

**Session reset:** At the start of every new session, all state machines (SSA long, SSA short, S1 long, S1 short, MUT long, MUT short) are fully reset. No state carries over between days.

---

## Inputs Reference

### Session

| Input | Default | Description |
|-------|---------|-------------|
| Session Time | `0000-0800` | The consolidation window to track. Format: `HHMM-HHMM`. |
| Session Timezone | `UTC` | Timezone the session hours are expressed in. Supports 60+ IANA zones. |
| Auto-extend Session End | `true` | Adds one bar period to the session end so the last candle is fully included. On a 15m chart with session 00:00–08:00, the effective end becomes 08:15. Disable to use the exact end time. |

### Pip Settings

| Input | Default | Description |
|-------|---------|-------------|
| Auto Pip Size | `true` | Derives 1 pip as `syminfo.mintick × 10`. Correct for most instruments. |
| Manual Pip Size | `0.0001` | Only active when Auto is off. Use `0.01` for JPY pairs, `1.0` for indices. |

### Schematics

Each schematic can be shown for both directions, one direction only, or turned off entirely.

| Input | Default | Options |
|-------|---------|---------|
| SSA Schematic | `Both` | Long · Short · Both · Off |
| S1 Schematic | `Both` | Long · Short · Both · Off |
| MUT Schematic | `Both` | Long · Short · Both · Off |

### Session Style

| Input | Default | Description |
|-------|---------|-------------|
| Border Color | Blue (40% opacity) | Color of the session box outline. |
| Background Color | Blue (85% opacity) | Fill color of the session box. |
| Border Width | `1` | Thickness of the border (1–4). |

### Display

| Input | Default | Description |
|-------|---------|-------------|
| Show Signal Labels | `true` | Renders the full signal name (e.g. `SSA Short`). When off, only a triangle marker is shown. |
| Label Distance | `0.25` | Vertical gap between the signal candle and the label, expressed as a multiple of ATR(14). Increase to avoid overlap on dense charts. |
| Triangle Size | `Auto` | Size of the triangle marker when labels are hidden. Options: Auto · Tiny · Small · Normal · Large · Huge. |

---

## Installation

1. Open TradingView and navigate to the **Pine Editor** (bottom panel).
2. Paste the full contents of `BreakoutSchematics.pine`.
3. Click **Save** and give it a name (e.g. `Breakout Schematics`).
4. Click **Add to chart**.

Compatible with all timeframes and most instruments. Pip auto-detection handles Forex, indices, and commodities without manual configuration.

---

## Roadmap

### Phase 1 — Core Foundation
- [x] Session engine with timezone support
- [x] SSA schematic (full state machine, both directions)
- [x] Label and triangle signal rendering
- [x] User input system and clean code structure

### Phase 2 — Expanded Schematics
- [x] S1 schematic detection and labeling
- [x] MUT schematic detection and labeling
- [ ] S5 schematic detection and labeling
- [ ] Shared state management for overlapping patterns

### Phase 3 — Alert System
- [ ] Native TradingView alerts for all schematics
- [ ] Optional webhook support
- [ ] Customizable alert messages (schematic name, direction, price level)

### Phase 4 — Visual Enhancements
- [ ] Order block and mitigation block drawing
- [ ] Fair value gap highlighting
- [ ] Extended session high/low lines
- [ ] Color themes and styling options

### Phase 5 — Strategy & Backtesting
- [ ] Full strategy version with entry and exit logic
- [ ] Risk management parameters (R:R ratio, stop-loss rules)
- [ ] Performance dashboard (win rate, profit factor, expectancy)

---

Development proceeds iteratively with live-chart testing. No fixed delivery dates.
See [CHANGELOG.md](CHANGELOG.md) for the full version history.

---

_Pine Script Developer — Stefan Narcis Cucoranu · March 2026_\
_Commissioner — Giovanni Roma_
