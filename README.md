# esphome-enail

**An open, Home Assistant-friendly controller for e-nail coil heaters.**
Take a cheap e-nail box, swap its controller for an ESP32 running [ESPHome](https://esphome.io),
and get a proper PID, timed sessions, a dial and screen on the box, and full control from
[Home Assistant](https://www.home-assistant.io).

> **Version 0.1.0** (2026-11-01). One box built and in use. Not yet built by anyone else.
> This is a mains (230 V) project. Read [SAFETY.md](SAFETY.md) before you open anything.

---

## What it is

An e-nail is a small electric heater: a coil wraps a quartz or titanium "nail", a K-type
thermocouple sits in the coil, and a box with a temperature controller keeps it hot. The boxes
sold online use a generic 48×24 mm PID module or a bare MCU board. They work, but you can't see
them from anywhere else, the tuning is a guess, and the timer is basic or missing.

This project replaces that controller with:

- an **ESP32** that runs the PID loop itself (ESPHome's `pid` climate),
- a **MAX6675** reading the coil's own thermocouple,
- an **SSR** switching the coil, with a **mechanical relay in series** as a master cut-off,
- a **rotary dial** and a small **OLED** on the box,
- a set of **Home Assistant packages and a dashboard** for sessions, ramps and stats.

The box works on its own. Home Assistant adds the extras but is never needed to heat safely or
to switch off.

## What it does

**On the box (firmware, `firmware/enail.yaml`)**

- PID temperature control, 1 s readings, time-proportioned SSR drive (2 s window).
- **Sessions**: press the dial to start; the box switches itself off after the session length (1–60 min).
- **Status**: Off → Heating → **Soaking** (holds a few minutes so heat reaches the quartz) → Ready → Done.
- **Dial**: 2 °F a click, faster spins step 10 or 20 °F. Long press swaps to session length.
- **Screen**: status line, live temperature, target, time left, heater power bar. Shows °F;
  everything inside stays in °C.
- **Screen saver** after 5 minutes idle, and a layout with no big lit areas (OLEDs burn in).
- **Safety, all on the device**: boots off; open or missing probe → off; over 480 °C → off;
  heater flat out for 2 minutes without a 10 °C rise → off ("no heat"); every session has a
  timer; the master relay drops on any stop.
- **Live PID tuning** from Home Assistant, plus a one-button autotune.
- Reports why it last restarted (brownout, crash, watchdog).

**In Home Assistant (`homeassistant/`)**

- `enail.yaml`: one-shot and stepped sessions, phone push on Ready, Done or Fault.
- `e_nail_target.yaml`: target in °F with 10 °F taps, safe for a shared tablet (it never starts a session).
- `e_nail_ramp.yaml`: **ramp session**, starts at one temperature and climbs to another by the end.
  Turning the dial takes over.
- `e_nail_pulls.yaml`: **pull counter**, spots the quick temperature dip when air is drawn
  over the nail. Per session, per day, all time.
- `dashboards/e-nail-view.yaml`: one tab with an on/off status card, target, ramp controls,
  a last-hour graph and the pull stats.

## What it uses

| Part | Job | Notes |
|---|---|---|
| Donor e-nail box + coil | Case, mains inlet, coil socket, coil with built-in K thermocouple | Its old controller board comes out |
| ESP32 (classic WROOM-32) | Runs everything | Built on a "MINI D1 ESP32" double-row board |
| MAX6675 module | Reads the K thermocouple | Chosen over MAX31855 on purpose, see [hardware](docs/hardware.md) |
| SSR-25DA (DC in, AC out) | Switches the coil | The coil draws under 1 A, no heatsink needed |
| 1-channel 5 V relay module | Master cut-off in series with the SSR | SSRs usually fail shorted. Jumper on high trigger |
| HLK-PM01 | 230 V AC → 5 V for the ESP32 and relay | |
| EC11 rotary encoder with push | Dial | Internal pull-ups, no resistors |
| 1.12" 128×128 SH1107 SPI OLED | Screen | Built with a Pimoroni Breakout Garden module |
| 3 lever connectors (Wago 221) | Live, neutral and 5 V joins | Replace the old board's terminals |
| Mains lead with a 3 A fuse | Protection | BS1362 **ceramic** 3 A in a UK/IE plug |

Software: ESPHome 2026.10.1 (tested), Home Assistant 2026.x, [Mushroom cards](https://github.com/piitaya/lovelace-mushroom) for two dashboard cards.

## Repository layout

```
esphome-enail/
├── firmware/
│   ├── enail.yaml               ESPHome config for the ESP32
│   └── secrets.example.yaml     copy to secrets.yaml
├── homeassistant/
│   ├── packages/                drop into /config/packages/
│   └── dashboards/e-nail-view.yaml
├── docs/
│   ├── hardware.md              parts, pin map, wiring diagrams
│   ├── bench-test.md            bring-up order, USB first, then mains
│   ├── pid-tuning.md            autotune vs hand-tuning, what we learned
│   ├── home-assistant.md        install, entities, dashboard
│   ├── troubleshooting.md       known traps and fixes
│   ├── prior-art.md             similar projects and credits
│   └── images/                  wiring diagrams (SVG)
├── .github/                     CI (ESPHome config check) and issue templates
├── SAFETY.md
├── CHANGELOG.md
└── LICENSE                      MIT
```

## Quick start

1. **Read [SAFETY.md](SAFETY.md).** Then [docs/hardware.md](docs/hardware.md) for the parts and wiring.
2. **Flash over USB, no mains.** Copy `firmware/enail.yaml` and `firmware/secrets.example.yaml`
   (as `secrets.yaml`) into your ESPHome folder, fill in the secrets, then
   `esphome run enail.yaml`.
3. **Bench-test on USB**, then on mains with the coil unplugged, then with the coil, following
   [docs/bench-test.md](docs/bench-test.md). Each step proves one safety stop.
4. **Tune the PID** for your coil ([docs/pid-tuning.md](docs/pid-tuning.md)). The shipped values
   fit one coil only.
5. **Add the Home Assistant side**: packages, then the dashboard ([docs/home-assistant.md](docs/home-assistant.md)).

## Status and limits

- One box built, in use since 2026-10-31. Treat it as a well-documented prototype.
- Once settled, the PID holds the coil reading within about ±1.5 °C of the target. Hot accuracy
  has not been checked against a reference thermometer.
- On the prototype the probe reads well below room temperature when cold (up to ~18 °C low).
  Only the working temperature matters, so no offset is applied.
- Below about 18 °C ambient the MAX6675 may report 0 °C at a cold start, which the firmware
  treats as a missing probe. See [troubleshooting](docs/troubleshooting.md).
- The box does **not** stop when Home Assistant or Wi-Fi drops. It keeps its target until
  the session timer ends (max 60 min). That is deliberate: it works away from home on the dial alone.
- The ramp session (fixed 2026-11-01) and the stepped session have not yet run end to end on a hot coil.
- No enclosure files or PCB yet: everything is point-to-point inside the donor box.

## Similar projects and credits

This stands on **ESPHome** (its `pid` climate, `max6675`, `slow_pwm`, `rotary_encoder`,
`ssd1306_spi` and `debug` components) and **Home Assistant**, plus **Mushroom cards** by
Paul Bottein (piitaya). No code from other projects is in this repo, but plenty of people solved
the same problems first. A nod to them (full list with links in [docs/prior-art.md](docs/prior-art.md)):

- [hardboiledfrog/e-nail](https://github.com/hardboiledfrog/e-nail): the same knob + OLED + timed
  session idea on an Arduino, with its own heat soak.
- [J-Cat/fc-enail](https://github.com/J-Cat/fc-enail): the most complete open e-nail found;
  its scripts "wait for the set point, then step", like our ramp.
- [kiln-controller/esphome-kiln-api](https://github.com/kiln-controller/esphome-kiln-api): the
  same three ESPHome blocks (`slow_pwm`, `max6675`, `pid`) running ramp/hold schedules on a kiln.
- [Saur0o0n/PIDKiln](https://github.com/Saur0o0n/PIDKiln): a mechanical relay in series with the
  SSR, for exactly our reason (SSRs fail closed), plus a no-heat check.
- [rancilio-pid/clevercoffee](https://github.com/rancilio-pid/clevercoffee): its espresso "brew
  detection" (a fast drop near the set point, then re-arm) is nearly our pull counter.
- [rocketscream/Reflow-Oven-Controller](https://github.com/rocketscream/Reflow-Oven-Controller)
  and [Ralim/IronOS](https://github.com/Ralim/IronOS): the reflow soak and the thermal-runaway
  check that our Soaking state and "no heat" fault echo.

Firmware, docs and tuning were worked out with [Claude Code](https://claude.com/claude-code).

## Licence

[MIT](LICENSE) for everything in this repo. Build it, change it, share it. You are responsible
for your own mains wiring. Note that ESPHome's C++ runtime is GPLv3, so a compiled firmware
image includes GPLv3 code ([ESPHome licence](https://github.com/esphome/esphome/blob/dev/LICENSE)).
