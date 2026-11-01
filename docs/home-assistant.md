# Home Assistant

The box runs on its own. Home Assistant adds remote control, ramps, stats and notifications.

## Install

1. Adopt the ESPHome device (name `enail`, friendly name `E-nail`, as in the firmware).
2. Turn on packages if you haven't, in `configuration.yaml`:
   ```yaml
   homeassistant:
     packages: !include_dir_named packages
   ```
3. Copy the files from `homeassistant/packages/` into `/config/packages/`. Each one stands
   alone; take only the ones you want (the dashboard expects all four).
4. In `enail.yaml`, change `notify.mobile_app_your_phone` to your phone's notify action.
5. Restart Home Assistant (or reload scripts, automations, templates and input helpers).
6. If HA named the °F sensor `sensor.e_nail_temperature_degf`, rename it to
   `sensor.e_nail_temperature_f` (Settings → Entities). The dashboard graph uses `_f`.
7. Install [Mushroom cards](https://github.com/piitaya/lovelace-mushroom) (HACS), then add
   `homeassistant/dashboards/e-nail-view.yaml` as a view (instructions at the top of the file).

## Entities from the box

| Entity | What it is |
|---|---|
| `climate.e_nail` | Live temperature and target (°C) |
| `switch.e_nail_session` | Start / stop a session |
| `number.e_nail_session_length` | 1–60 min. Changing it mid-session moves the end |
| `number.e_nail_soak_time` | Soak before Ready, 0–10 min (config) |
| `sensor.e_nail_status` | Off, Heating, Soaking, Ready, Done, Stopped, Autotune, Fault: … |
| `sensor.e_nail_temperature` | Coil reading, °C |
| `sensor.e_nail_heat_output` | PID output, % |
| `sensor.e_nail_time_left` | Minutes left in the session |
| `number.e_nail_pid_kp/_ki/_kd/_max_integral` | Live PID tuning, not saved (config) |
| `button.e_nail_pid_autotune` | Runs an autotune (config) |
| `sensor.e_nail_reset_reason` | Why the board last restarted (diagnostic) |
| `sensor.e_nail_dial_position`, `binary_sensor.e_nail_dial_button` | Dial, for testing (diagnostic) |

## What each package adds

**`enail.yaml`: sessions and alerts**
- `script.enail_session`: one temperature (°C) for N minutes.
- `script.enail_stepped_session`: a list of steps, e.g. `[{temp: 230, minutes: 5}, {temp: 260, minutes: 10}]`.
  Each step waits for the coil to get within 5 °C, then holds.
- An automation pushes to your phone on Ready, Done or any Fault.

**`e_nail_target.yaml`: °F target**
- `number.e_nail_target`: 212–842 °F in 10 °F taps. It only changes the target and never starts
  a session, so it's safe on a shared tablet. It shows the nearest 5 °F preset when within 1 °F
  (Home Assistant only gets the target in whole °C).
- `sensor.e_nail_temperature_f`: the coil in °F for graphs.

**`e_nail_ramp.yaml`: ramp session**
- `input_number.e_nail_ramp_from` / `_to` (°F) and `script.e_nail_ramp_session`.
- Starts the session at From, waits for `Ready`, then moves the target in 2 °F steps so it
  reaches To one minute before the session ends. **The ramp time is the session length.**
- Moving the dial or the target by more than 1.5 °C takes over: the ramp stops, the session
  carries on. Switching the session off ends the ramp.
- If Home Assistant goes away mid-ramp, the box holds the last target until its timer ends.

**`e_nail_pulls.yaml`: pull counter**
- `sensor.e_nail_pulls`: all-time total (`total_increasing`, so HA's statistics give per-day
  bars), with attributes `session` and `today`.
- A pull = the coil drops ≥ 4 °C within 12 s while holding. Armed 90 s after the coil first
  gets within 3 °C of the target, at most one per 25 s.
- Autotune runs don't count. Turn `input_boolean.e_nail_tuning` on while you test, so test
  sessions don't count either.
- Thresholds were set on one coil and one PID tune. Check them against your own sessions.

## Privacy

Session history and pull counts are personal data. Home Assistant has no per-entity privacy:
anyone with a login can find them in History. What you can do:
- Put the view on a dashboard only you can see (`visible:` with your user id).
- Don't expose `sensor.e_nail_pulls` to Assist, voice assistants or cloud integrations.
- Keep the Pulls section off shared tablets. The target tile is fine to share.
