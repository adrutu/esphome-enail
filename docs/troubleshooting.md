# Troubleshooting and known traps

Everything here happened on the prototype at least once.

## `Fault: probe` when starting from cold in a cold room

The MAX6675 can't report below 0 °C and clamps to 0.0. On the prototype the coil's probe also
reads well under room temperature when cold. In a room below about 18 °C the reading can be
0.0, which the firmware treats as a missing probe. Warm the room or the coil a little, or
(your call) let the firmware accept 0.0 for the first few seconds of a session. The worst case
of that change, if the probe really is loose: about 5 s at full power before the stop.

## New firmware vanished after a flash ("OTA rollback")

ESPHome's safe mode marks a new build as good only after it has run for 60 s. If the board
resets inside that window, the bootloader goes back to the **old** firmware ("OTA rollback
detected" in the log) and the new build's entities disappear from Home Assistant.

**Flash with the session off, and keep it off for over a minute** (check `Uptime` > 60 s)
before starting a session. Also don't open a serial log (which toggles reset) right after an OTA.

## The SSR stayed on during a flash

An OTA upload blocks ESPHome's main loop, so the SSR output stays where it was. Switch the
session off before every flash.

## The box restarts by itself

`sensor.e_nail_reset_reason` says why after each boot: brownout (power dip), panic or watchdog
(crash). On the prototype, unexplained resets were seen mid-session and once with the session
off. Suspects are relay/SSR switching noise on the 5 V rail and the exposed OLED header. Open
item: if you see it, note the reset reason and what was happening.

## The OLED shows garbage, flashes, or a ghost image

- Garbled or flashing over jumper wires: `data_rate: 1MHz` and `spi_mode: MODE0` (both set).
- Upside down: `rotation:` (180 on the prototype, header at the bottom).
- A faint picture you never drew: **burn-in**. The prototype's panel came with a ghost of an
  old demo. The layout avoids big lit areas and the screen sleeps after 5 minutes idle.
  The first press or turn on a dark screen only wakes it.

## The dial does nothing on the first press

The screen was asleep. The first input wakes it; the next one acts.

## The ESP32 is stuck in download mode

`boot:0x1` or `0x21` in the serial log, nothing runs: IO0/IO15 are being pulled low. On the
double-row ESP32 Mini this happens when it sits in a breadboard (inner and outer rows short).
Take it out and wire it with jumpers.

## The temperature swings or overshoots

See [pid-tuning.md](pid-tuning.md). After an autotune, reboot: ESPHome runs the raw autotune
values until then.

## Home Assistant lost the box and the coil is still hot

Expected. The box doesn't stop on a disconnect. It holds the target until the session timer
ends, then switches off. Use the dial to stop it sooner.

## Template errors with tiny numbers in scripts

A Jinja value like `0.000012` renders as `1.2e-05`, which Home Assistant can keep as **text**,
and the next multiplication fails. The ramp script rounds (`| round(4)`) for this reason. Do
the same in your own scripts.
