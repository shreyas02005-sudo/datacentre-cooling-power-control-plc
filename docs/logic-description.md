# Logic Description

A walk through the program, rung by rung.

Notation: `·` is AND, `+` is OR and `!` is NOT (a normally closed contact). Most rungs include `!power_relay`, so energising `power_relay` switches those rungs off.

Every coil is a plain output (OTE), so outputs follow their conditions on every scan. Nothing is latched except `CR`.

## Block A: Start / stop (rung 0)

```
CR = (start + CR) · !stop
```

This is a standard seal-in circuit. Pressing `start` energises `CR`, and the parallel `CR` contact keeps it on after the button is released. `stop` is examined as XIO, so pressing it breaks the circuit. Every later rung includes `CR`, which makes it the master enable.

## Block B: Thermal management (rungs 1-5)

| Rung | Equation | What it does |
|------|----------|--------------|
| 1 | `cooling_fans = !power_relay · CR` | Baseline cooling whenever the system is running |
| 2 | `secondary_cooling_fan = !power_relay · CR · temp_high` | Extra cooling only when it's needed |
| 3 | `overheat = CR · temp_overheat` | Internal overheat flag |
| 4 | `AC_unit = !power_relay · CR · overheat` | Starts the AC on overheat |
| 5 | `overheat_light = CR · overheat` | Shows the operator there's an overheat |

## Block C: Critical shutdown (rungs 6-8)

| Rung | Equation | What it does |
|------|----------|--------------|
| 6 | `critical_T1 (TON 5 s) enable = CR · critical_temp` | Checks the condition lasts |
| 7 | `critical_shutdown = CR · critical_T1.DN` | Issues the shutdown |
| 8 | `critical_light = CR · critical_shutdown` | Shows the operator |

The 5 s timer means a short sensor spike won't cause a shutdown.

## Block D: Power backup (rungs 9-16)

| Rung | Equation | What it does |
|------|----------|--------------|
| 9 | `backup_timer (TON 5 s) enable = CR · power_fail · !critical_shutdown` | Ignores brief flickers, and doesn't start a backup during a critical shutdown |
| 10 | `power_backup_mode = !power_relay · CR · backup_timer.DN` | Enters backup mode |
| 11 | `yellow_light = !power_relay · CR · power_backup_mode` | Warns operators |
| 12 | `mains = !power_relay · CR · !power_backup_mode · !critical_shutdown` | Drops the mains contactor in backup mode or critical shutdown |
| 13 | `generator_warmup_del (TON 5 s) enable = !power_relay · CR · power_backup_mode · !critical_shutdown` | Gives the generator time to stabilise |
| 14 | `generator_ready = !power_relay · CR · generator_warmup_del.DN` | Warm-up is complete |
| 15 | `generator = !power_relay · CR · generator_ready` | Closes the generator contactor |
| 16 | `generator_light = !power_relay · CR · generator` | Shows the generator is running (it reads the generator output) |

Here's the timeline after a mains failure:

- **t = 0:** `power_fail` goes on
- **t = 5 s:** backup mode starts, mains drops and the yellow light comes on
- **t = 10 s:** the generator comes online

Mains drops before the generator closes, so the two sources never overlap (break-before-make).

## Block E: Fire safety (rungs 17-20)

| Rung | Equation | What it does |
|------|----------|--------------|
| 17 | `smoke_alarm = CR · smoke_detector` | Immediate alarm |
| 18 | `power_relay = CR · smoke_detector` | Energises the isolation relay, so every `!power_relay` rung drops out |
| 19 | `fire_suppression_del (TON 5 s) enable = CR · smoke_detector` | Pre-discharge delay |
| 20 | `gas_based_fire_suppression = CR · fire_suppression_del.DN` | Releases the suppression agent |

When smoke is detected, rung 18 switches off the cooling fans, AC, mains and generator through the `!power_relay` contacts. That means power is isolated before the gas is released.

## A note on scan order

`power_relay` is read in rungs 1-16 but written in rung 18. The PLC scans from top to bottom, so those earlier rungs see the value from the previous scan. The delay is a few milliseconds and doesn't matter here, but it's worth keeping in mind if the logic is changed.
