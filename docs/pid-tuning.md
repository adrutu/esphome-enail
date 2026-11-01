# PID tuning

The values in `firmware/enail.yaml` fit **one coil** (the prototype's second, bigger coil).
Start with them, but expect to tune your own.

## The short version

1. Press **PID autotune** in Home Assistant once, at your usual working temperature.
2. Read the result from the ESPHome log. Start from its **"No Overshoot"** set, not the
   Ziegler-Nichols one.
3. Step-test it (below). If it still overshoots or swings, hand-tune with the live numbers.
4. Copy what works into `substitutions:` (`pid_kp`, `pid_ki`, `pid_kd`, `pid_max_i`) and reflash.

## How the autotune button works

It sets a 30-minute session, starts it if needed, then runs ESPHome's
[`climate.pid.autotune`](https://esphome.io/components/climate/pid.html) at **30 % heat**
(`positive_output: 0.3`). At full power this coil heats about 5× faster than it cools (on
~11 s, off ~55 s), which makes the autotune's oscillation lopsided. 30 % evens it out.

**Trap:** after an autotune, ESPHome keeps running the raw Ziegler-Nichols values until the
next reboot. On the prototype those swung ±20 °C. Always switch the session off and reboot or
reflash after a tune.

## Live tuning

Four config numbers in Home Assistant change the running PID at once: `PID kp`, `PID ki`,
`PID kd`, `PID max integral`. They are **not saved**: a reboot goes back to the substitutions.

## Step test, done right

Cool the coil with the session **off**, then start a **fresh session** at the target. Watch the
peak, how long it takes to settle, and the swing once settled.

**Trap:** cooling to a lower target *inside* a session winds the integral down to
−max_integral. The next climb then looks slow and overshoot-free, which is fake. Real use always
starts a fresh session, which resets the integral.

## What happened on the prototype

| Run | Where | Result |
|---|---|---|
| Untuned | 275 °C target | held ~327 °C |
| Autotune #1, full power | 320 °C | Pu ≈ 75 s. Z-N kp 0.01863, ki 0.0005, kd 0.17422. Peaked 287 °C at a 254 °C target, then a slow 232↔251 °C swing |
| Autotune #2, 30 % heat | 249 °C | Z-N set swung 233–273 °C for 27 min. **No Overshoot** set kp 0.01019, ki 0.00088, kd 0.07356 + `max_integral: 0.35`: +4.5 °C peak, then 248–252 °C |
| New, bigger coil. Autotune #3, 30 % heat | 254 °C | No Overshoot kp 0.01091, ki 0.00101, kd 0.07367: **+14 °C overshoot, ±6 °C on a ~70 s cycle** |
| **Hand-tuned** (shipped) | 254 °C | kp **0.015**, ki **0.0003**, kd **0.3**, max_integral **0.2**: reaches target in ~72 s from 184 °C, **+3.8 °C** peak, 8-minute hold 253.0–255.25 °C (mean 254.40 for 254.44), heater ~11 % |

**Lesson:** the short 30 % autotunes measured a ~22 s cycle, but the real one on this coil is
~70–75 s (as the full-power autotune found). Their ki came out about 3× too quick. Dividing ki
by 3, raising kd ×4 and capping the integral at a bit above the holding power (~11 % → 0.2)
fixed it. If your autotune's period looks short, trust a full-power run's period.

`max_integral` is the anti-wind-up limit. Set it a little above the heat output your coil
needs to hold temperature (watch `Heat output` during a steady hold).

Not yet measured on the shipped values: a start from room temperature, and how fast it recovers
after a pull.
