# Logic Description

Notation: `·` AND, `+` OR, `!` NOT (normally closed contact). `power_relay` appears as `!power_relay` in most rungs, so asserting it disables those rungs.

All rungs are non-latching (OTE coils), so every output follows its input conditions on each scan.

## Block A: Start / stop (rung 0)

```
CR = (start + CR) · !stop
```

A classic seal-in circuit. `start` energises `CR`, and the parallel `CR` contact holds it on after release. `stop` is examined as XIO and breaks the circuit. `CR` is the master enable for all later rungs.

## Block B: Thermal management (rungs 1-5)

| Rung | Equation | Purpose |
|------|----------|---------|
| 1 | `cooling_fans = !power_relay · CR` | Baseline cooling whenever the system is running |
| 2 | `secondary_cooling_fan = !power_relay · CR · temp_high` | Extra cooling only when needed |
| 3 | `overheat = CR · temp_overheat` | Internal overheat status bit |
| 4 | `AC_unit = !power_relay · CR · overheat` | Start AC on overheat |
| 5 | `overheat_light = CR · overheat` | Operator indication |

## Block C: Critical shutdown (rungs 6-8)

| Rung | Equation | Purpose |
|------|----------|---------|
| 6 | `critical_T1 (TON 5 s) enable = CR · critical_temp` | Confirm the condition persists |
| 7 | `critical_shutdown = CR · critical_T1.DN` | Shutdown command |
| 8 | `critical_light = CR · critical_shutdown` | Operator indication |

The 5 s timer means a brief sensor spike does not cause a shutdown.

## Block D: Power backup (rungs 9-16)

| Rung | Equation | Purpose |
|------|----------|---------|
| 9 | `backup_timer (TON 5 s) enable = CR · power_fail · !critical_shutdown` | Ignore flickers, and no backup during critical shutdown |
| 10 | `power_backup_mode = !power_relay · CR · backup_timer.DN` | Enter backup mode |
| 11 | `yellow_light = !power_relay · CR · power_backup_mode` | Warn operators |
| 12 | `mains = !power_relay · CR · !power_backup_mode · !critical_shutdown` | Mains contactor drops in backup mode or critical shutdown |
| 13 | `generator_warmup_del (TON 5 s) enable = !power_relay · CR · power_backup_mode · !critical_shutdown` | Let the generator stabilise |
| 14 | `generator_ready = !power_relay · CR · generator_warmup_del.DN` | Warm-up complete |
| 15 | `generator = !power_relay · CR · generator_ready` | Close generator contactor |
| 16 | `generator_light = !power_relay · CR · generator` | Indication (reads the generator output) |

Timeline after a mains failure: **t = 0** `power_fail`, **t = 5 s** backup mode (mains drops, yellow light), **t = 10 s** generator online. The mains drops before the generator closes, so the two sources never overlap (break-before-make).

## Block E: Fire safety (rungs 17-20)

| Rung | Equation | Purpose |
|------|----------|---------|
| 17 | `smoke_alarm = CR · smoke_detector` | Immediate alarm |
| 18 | `power_relay = CR · smoke_detector` | Energise the isolation relay, so every `!power_relay` rung drops out |
| 19 | `fire_suppression_del (TON 5 s) enable = CR · smoke_detector` | Pre-discharge delay |
| 20 | `gas_based_fire_suppression = CR · fire_suppression_del.DN` | Release suppression agent |

When smoke is detected, rung 18 removes cooling fans, AC, mains and generator outputs through the `!power_relay` contacts. Power is isolated before the gas discharge.

## Scan-order note

`power_relay` is read in rungs 1-16 and written in rung 18. Because the PLC scans top to bottom, those rungs see the value from the previous scan. The one-scan (milliseconds) delay is negligible here, but it is worth remembering when changing the logic.
