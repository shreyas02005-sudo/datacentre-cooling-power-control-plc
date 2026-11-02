# Verification Scenarios

Each scenario lists the inputs to force and the behaviour expected from the ladder logic. Use them to test in Logix Emulate or on hardware. Every output below also requires `CR` = ON unless stated.

| # | Scenario | Inputs | Expected behaviour |
|---|----------|--------|--------------------|
| 1 | Power-up, not started | none | All outputs OFF |
| 2 | Start | Pulse `start` | `CR` latches, `cooling_fans` ON, stays ON after release |
| 3 | Stop | Pulse `stop` | `CR` drops, all outputs OFF |
| 4 | High temperature | `temp_high` ON | `secondary_cooling_fan` ON, `cooling_fans` stays ON |
| 5 | Overheat | `temp_overheat` ON | `overheat` set, `AC_unit` ON, `overheat_light` ON |
| 6a | Critical temp, transient | `critical_temp` ON for less than 5 s | Timer resets, no shutdown |
| 6b | Critical temp, sustained | `critical_temp` ON for 5 s or more | `critical_shutdown` ON, `critical_light` ON, `mains` OFF |
| 7a | Power flicker | `power_fail` ON for less than 5 s | No change, `mains` stays ON |
| 7b | Power failure | `power_fail` ON for 5 s | `power_backup_mode` ON, `yellow_light` ON, `mains` OFF |
| 7c | Generator online | `power_fail` ON for 10 s | `generator_ready`, `generator` and `generator_light` ON |
| 8 | Mains restored | `power_fail` OFF | Backup mode drops, `mains` ON, generator OFF |
| 9 | Critical shutdown during power failure | `critical_shutdown` ON, then `power_fail` ON | Backup timer and generator warm-up inhibited, `mains` OFF |
| 10a | Smoke, brief | `smoke_detector` ON for less than 5 s | `smoke_alarm` ON, `power_relay` energised, no gas release |
| 10b | Smoke, sustained | `smoke_detector` ON for 5 s or more | Gas suppression ON, powered loads (fans, AC, mains, generator) OFF |
| 11 | Stop during fire | `smoke_detector` ON, then `stop` | `CR` drops, so alarm and suppression drop out (see design review) |

## Scenario 10b: expected timeline

| Time | Event |
|------|-------|
| 0 s | `smoke_alarm` ON, `power_relay` energised, fans / AC / mains / generator OFF |
| 5 s | `fire_suppression_del` done, `gas_based_fire_suppression` ON |
