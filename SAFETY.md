# Safety

This project works on **mains voltage (230 V AC)** and builds a **heater that reaches 450 °C**.
Either can kill you or burn your house down. Only build it if you are competent with mains
wiring, and follow your local electrical rules. You build and use it at your own risk.

## Rules the design depends on

1. **Fuse the lead small.** The coil draws under 1 A. Use a **3 A** fuse: in a UK/Irish plug
   that is a **BS1362 ceramic** (sand-filled, opaque, "BS1362" printed). A glass fuse will not
   seat properly and can shatter on a short. A 13 A fuse protects the cable, not your box.
2. **Earth the coil socket and any metal case.** Keep the lead's earth wire on the coil
   socket's shell.
3. **Two switches in series.** The SSR does the PID switching. A mechanical relay in series
   cuts the coil whenever the firmware stops a session. SSRs usually fail **shorted**: without
   the relay a dead SSR means a coil heating flat out.
4. **Relay jumper on high-level trigger**, so a floating or booting ESP32 pin can't close it.
5. **Never USB and mains at once.** Unplug the USB cable before the mains lead goes in.
6. **Keep low voltage away from mains.** At least a finger's width between the 5 V/3.3 V
   wiring and anything on the mains side. Every mains joint in a lever connector, a screw
   terminal or heat-shrink.
7. **Beep test before first power-up.** Multimeter on continuity: ESP32 GND against live,
   neutral and both heater wires must be silent. The plug's earth pin against the coil socket's
   metal must beep.
8. **Don't put the ESP32 Mini in a breadboard.** Its inner and outer pin rows land in the same
   strip and short each other (it boots into download mode and stays there).

## What the firmware does for you

All of these run on the ESP32. None needs Wi-Fi or Home Assistant.

| Condition | Action |
|---|---|
| Power-up | Heater and relay off, PID off |
| Probe open, unreadable, or reads ≤ 0 °C | Stop: `Fault: probe` (a missing MAX6675 reads exactly 0 °C) |
| Reading above 480 °C | Stop: `Fault: too hot` |
| Heater ≥ 90 % for 2 min without a 10 °C rise | Stop: `Fault: no heat` (probe off the coil, dead coil or SSR) |
| Session timer runs out (max 60 min) | Stop: `Done` |
| Any stop | PID off **and** master relay open |

## What it does **not** do

- It does **not** stop when Home Assistant or Wi-Fi goes away. It holds its last target until
  the session timer ends. That is on purpose (the box must work on its own), so always set a
  session length you are happy with.
- It can't protect against a probe that has slipped out of the coil but still reads a
  plausible temperature beyond the 2-minute no-heat check.
- A firmware update (OTA) blocks the main loop while it uploads: the SSR stays in whatever state
  it was. **Switch the session off before flashing.**
- It is not a certified appliance. There is no thermal fuse unless you add one.

## Before you leave it

Switch the session off. The coil stays dangerously hot for several minutes after: the
dashboard's status card says "cooling off" until it is under 60 °C. Let it cool before you
touch it or pack it away.
