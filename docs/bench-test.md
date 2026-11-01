# Bench test: bring-up order

Each step proves one thing before the next adds risk. Don't skip ahead. The times and results
in *italics* are what the prototype did.

## 0. Flash over USB

Mains lead unplugged, ESP32 on USB only.

```sh
cp firmware/enail.yaml firmware/secrets.example.yaml ~/esphome/   # your ESPHome folder
mv ~/esphome/secrets.example.yaml ~/esphome/secrets.yaml           # then fill it in
esphome run ~/esphome/enail.yaml
```

Adopt the device in Home Assistant (Settings → Devices → ESPHome). With the device named
`enail` and friendly name `E-nail`, the entities come out as `climate.e_nail`,
`switch.e_nail_session`, `sensor.e_nail_temperature` and so on, which is what the packages expect.

## 1. USB: screen, dial, probe

- The OLED shows `Off`, the live temperature (°F) and the Set/Time rows.
- Turning the dial moves the Set row; a long press moves the marker to Time.
- Pinch the probe tip: the reading rises. *Prototype: it fell at first; T+/T− swapped.*
- With no probe or no MAX6675, a press shows `Fault: probe` and refuses to start. That's the lock working.

## 2. USB: session and no-heat stop

Press the dial. The relay clicks and the SSR's LED lights (status `Heating`). Nothing heats on
USB, so after 2 minutes the box must stop itself with **`Fault: no heat`** and the relay must
click off. *Prototype: 12:22:08 start → 12:24:08 `Fault: no heat`.*

## 3. Isolation check, unplugged

Multimeter on continuity:

- ESP32 GND against Wago 1, Wago 2 and both heater wires: **silent** on every one.
- Plug earth pin against the coil socket's metal: **beep**.

## 4. Mains, coil unplugged

USB out, lead in. The ESP32 should come up on the HLK-PM01 alone and show in Home Assistant.
Start a session: relay clicks, and after 2 minutes `Fault: no heat` again.
*Prototype: the first try gave no 5 V at the ESP32 although the HLK-PM01 measured 5 V at its own
pins; a joint was re-made.*

## 5. Coil in, low first

Set 150 °C (302 °F), start, and watch it reach `Soaking` then `Ready`. Your phone gets the
push if the Home Assistant package is in. *Prototype, untuned: ~6 °C/s heat-up and a big
overshoot (target 275 °C held ~327 °C). That's normal before tuning.*

## 6. Tune the PID

See [pid-tuning.md](pid-tuning.md). Don't run long sessions on untuned values.

## 7. Into the box, then a cold boot on the real lead

Fix every module down, keep low voltage away from mains, close the lid. Then switch off at
the wall, switch on again, and check: the box boots `Off`, shows in Home Assistant, and a
dial press starts a session. *Prototype: power-cycled in the closed box, back online in
~33 s, booted Off, session started from the dial.*
