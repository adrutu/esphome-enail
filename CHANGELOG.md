# Changelog

Format: [Keep a Changelog](https://keepachangelog.com/en/1.1.0/). Versions: [SemVer](https://semver.org/).

## [0.1.0] - 2026-11-01

First packaged release of a working, in-use prototype.

### Firmware
- ESPHome `pid` climate on an ESP32 with a MAX6675 K-type probe and an SSR (2 s time-proportioned drive).
- Mechanical master relay in series with the SSR, dropped on every stop.
- Device-side safety: boot off, probe fault (NaN or ≤ 0 °C), 480 °C cutoff, 2-minute no-heat
  watchdog, session timer (1–60 min).
- Status flow Off → Heating → Soaking → Ready → Done, with a configurable soak time.
- EC11 dial: 2 °F a click with acceleration (10/20 °F), long press swaps to session length,
  short press starts/stops.
- 1.12" SH1107 OLED in °F with status, target, time left and a heater bar; burn-in-safe layout and 5-minute screen saver.
- One-button autotune at 30 % heat; live PID tuning numbers; hand-tuned values for the prototype coil.
- Reset reason, Wi-Fi signal and uptime diagnostics. ESPHome `project:` version 0.1.0.

### Home Assistant
- Session and stepped-session scripts; phone push on Ready, Done or Fault.
- °F target number (10 °F taps) and °F temperature sensor.
- Ramp session script (From → To over the session length, dial takes over).
- Pull counter (per session, per day, all time).
- Dashboard view: on/off status card, target with 470–510 °F preset chips, ramp controls,
  last-hour graph, pull stats. The °F target snaps to the 5 °F presets.

### Docs
- Hardware, wiring diagrams, bench-test order, PID tuning log, troubleshooting, safety.
- Prior art: 15 similar open projects (e-nail, kiln, espresso, reflow, soldering) and what we share with them.

## Before 0.1.0

Built 2026-10-21 → 2026-11-01 inside a private home-lab repo: firmware and HA package (10-21),
OLED fixes (10-21), probe and SSR changes (10-21), autotunes and °F UI (10-31), first mains
run and box build (10-31), pull counter, °F target, new coil and hand-tune (11-01), ramp
session, status card, soak state and reset reason (11-01).
