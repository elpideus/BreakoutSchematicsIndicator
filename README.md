# Breakout Schematics

**Pine Script v6 · Version 0.3.0 · In Development**

A TradingView indicator that automatically detects and visualizes high-probability
breakout schematics following a user-defined consolidation session. Built around a
clean state-machine architecture designed for accuracy, readability, and easy expansion.

> **Note:** This project is under active development and not yet ready for production use.

---

## Overview

Breakout Schematics works by tracking price behavior relative to a reference session
(default: Asian Session, 00:00–08:00 UTC). Once the session closes, the indicator
monitors for structured breakout patterns and plots signals directly on the chart.

The current release implements two fully working schematics:

- **SSA** — a three-phase breakout-continuation-confirmation pattern (B-C and C variants)
- **S1** — a two-phase breakout-and-reversal pattern

---

## How It Works

### Session Engine

At the start of each session, the indicator begins tracking the rolling high and low.
When the session closes, it draws a shaded box over the session range and locks the
boundaries as reference levels for schematic detection.

- Configurable session time and timezone (full IANA support)
- Real-time high/low box rendered only during the active session
- State resets cleanly on every new session open

---

### SSA Schematic — State Machine

After session close, the indicator runs two parallel state machines (one long, one short),
each progressing through the following phases:

**Phase 1 — Breakout (Candle A)**
Price wicks beyond the session boundary by at least 1 pip. The body must not close
through the boundary — a full close invalidates the setup and resets state.

**Phase 2 — Continuation (Candle B)**
A second spike extends the wick extreme by at least 0.6 pip, confirming directional
pressure. The extreme is updated if price pushes further.

**Phase 3 — Confirmation (Candle C)**
Price pulls back from the extreme by at least 0.6 pip, then makes a decisive move in
the breakout direction — body close clearing the prior extreme by 0.6 pip. This triggers
the signal.

Two signal variants are produced depending on the path through the state machine:

| Signal              | Description                                     |
| ------------------- | ----------------------------------------------- |
| `SSA B-C Long/Short` | Phases 1 → 2 → 3 completed sequentially         |
| `SSA C Long/Short`   | Phase 3 fires directly after a Phase 2 pause    |

Any candle body closing back through the session boundary immediately invalidates the
active setup.

---

### S1 Schematic — State Machine

A two-phase pattern that captures breakout-and-reversal structures relative to the
session boundary.

**Phase 1 — Breakout (Reference Candle)**
A candle's body closes beyond the session boundary by at least 1 pip. The candle's wick
extreme and body edge are locked as tracking references.

**Phase 2 — Reversal (Signal Candle)**
A subsequent candle reverses and closes through the reference body edge by at least
0.6 pip. This triggers the signal.

If price instead continues in the breakout direction (new wick extreme), the reference
candle is updated to the new one and Phase 2 restarts.

| Signal          | Description                                          |
| --------------- | ---------------------------------------------------- |
| `S1 Long/Short` | Body breaks out, then reverses through reference body |

---

## Inputs

### Session

| Input            | Default     | Description                                                      |
| ---------------- | ----------- | ---------------------------------------------------------------- |
| Session Time     | `0000-0800` | Time range for the reference consolidation session               |
| Session Timezone | `UTC`       | IANA timezone (e.g. `Europe/Rome`, `America/New_York`)           |

### Session Style

| Input            | Default                    | Description                              |
| ---------------- | -------------------------- | ---------------------------------------- |
| Border Color     | Blue (40% transparency)    | Color of the session box border          |
| Background Color | Blue (85% transparency)    | Fill color of the session box            |
| Border Width     | `1`                        | Thickness of the session box border (1–4)|

### Pip Settings

| Input           | Default  | Description                                                         |
| --------------- | -------- | ------------------------------------------------------------------- |
| Auto Pip Size   | `true`   | Derives pip from `syminfo.mintick × 10` — recommended               |
| Manual Pip Size | `0.0001` | Used only when Auto is off; use `0.01` for JPY, `1.0` for indices  |

### SSA Schematic

| Input               | Default | Description                                  |
| ------------------- | ------- | -------------------------------------------- |
| Show Long Signals   | `true`  | Plots green labels for SSA long setups       |
| Show Short Signals  | `true`  | Plots red labels for SSA short setups        |

### S1 Schematic

| Input               | Default | Description                                  |
| ------------------- | ------- | -------------------------------------------- |
| Show Long Signals   | `true`  | Plots green labels for S1 long setups        |
| Show Short Signals  | `true`  | Plots red labels for S1 short setups         |

### Display

| Input               | Default | Description                                                         |
| ------------------- | ------- | ------------------------------------------------------------------- |
| Show Signal Labels  | `true`  | Shows full text label; when off, renders triangle marker only       |
| Label Distance      | `0.25`  | Vertical offset from candle as an ATR(14) multiplier                |
| Triangle Size       | `Auto`  | Options: Auto · Tiny · Small · Normal · Large · Huge                |

---

## Installation

1. Open TradingView and navigate to **Pine Editor**
2. Paste the full contents of `BreakoutSchematics.pine`
3. Click **Save** and name it (e.g. `Breakout Schematics`)
4. Click **Add to chart**

Compatible with all timeframes and most instruments. Pip auto-detection handles
Forex, indices, and commodities without manual adjustment.

---

## Roadmap

### Phase 1 — Core Foundation

- [x] Session engine with timezone support
- [x] SSA schematic (full state machine, both directions)
- [x] Label and triangle signal rendering
- [x] User input system and clean code structure

### Phase 2 — Expanded Schematics

- [x] S1 schematic detection and labeling
- [ ] MUT schematic detection and labeling
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

Development proceeds iteratively with live-chart testing. No fixed delivery dates.
See [CHANGELOG.md](CHANGELOG.md) for the full version history.

---

_Pine Script Developer — Stefan Narcis Cucoranu · March 2026_\
_Commissioner — Giovanni Roma_
