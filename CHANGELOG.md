# 📜 Changelog

## [0.3.0] — 2026-04-01

### New Features

- ✅ S1 schematic fully implemented — two parallel state machines (long and short) detecting breakout-and-reversal patterns relative to the session boundary
- 📋 Settings overview table added to the top-right of the chart, displaying active session, timezone, pip size, SSA direction, and S1 direction at a glance
- 🎨 Session box style inputs added (border color, background color, border width) under a dedicated "Session Style" group

### Input System Overhaul

- 🗂️ All inputs reorganized into labeled groups: ⏱ Session, 📏 Pip Settings, 📈 Schematics, 🎨 Session Style, 🎨 Display
- 🌍 Session timezone input replaced free-text entry with a curated dropdown of ~80 IANA timezones with UTC offset annotations
- ➕ "Auto-extend Session End" toggle added — automatically extends the session end by one timeframe period so the last candle is fully included
- 🔀 SSA and S1 schematic controls replaced simple show/hide booleans with "Long / Short / Both / Off" direction dropdowns
- 🏷️ "Show Signal Labels" replaces the old "Show Candle Letter" toggle with clearer naming and updated tooltip
- 📐 "Label Distance" and "Triangle Size" inputs moved into the Display group with improved tooltips

### State Machine Fixes

- 🐛 SSA Phase 1 body-breakout invalidation now correctly uses `breakout_thresh` (1 pip) instead of bare `sp_lock` / `lp_lock`, preventing premature resets
- 🔗 SSA A-candle detection now disarms the corresponding S1 state machine, preventing conflicting signals on the same setup
- 🔗 SSA and S1 signals now cross-disarm each other on fire, ensuring only one schematic wins per session breakout
- 🏷️ Signal labels unified: "SSA B-C Long/Short" and "SSA C Long/Short" collapsed to "SSA Long/Short"; S1 labels follow the same pattern

## [0.2.0] — 2026-03-09

- 🛠️ Fixed state machine logic to correctly trail extremes (sp_min and lp_max)
  in Phase 2
- 🏷️ Added version string to Pine Script indicator definition
- 📝 Updated README for project documentation parity

## [0.1.0] — 2026-03-07

- 🚀 Initial release
- ✅ SSA schematic fully functional (B-C and C variants, long and short)
- ✅ Session engine with box visualization
- ✅ Complete input and visual signal system
