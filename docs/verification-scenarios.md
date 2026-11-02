# Verification Scenarios

These are the scenarios we use to check the program. For each one, force the listed inputs (in Logix Emulate or on hardware) and compare the result with the expected behaviour. Unless a row says otherwise, `CR` needs to be on.

| # | Scenario | Inputs | Expected behaviour |
|---|----------|--------|--------------------|
| 1 | Power-up, not started | None | All outputs off |
| 2 | Start | Pulse `start` | `CR` latches and `cooling_fans` turns on. It stays on after the button is released |
| 3 | Stop | Pulse `stop` | `CR` drops and all outputs turn off |
| 4 | High temperature | `temp_high` on | `secondary_cooling_fan` turns on, and `cooling_fans` stays on |
| 5 | Overheat | `temp_overheat` on | `overheat` is set, and `AC_unit` and `overheat_light` turn on |
| 6a | Critical temperature, brief | `critical_temp` on for under 5 s | The timer resets and nothing shuts down |
| 6b | Critical temperature, sustained | `critical_temp` on for 5 s or more | `critical_shutdown` and `critical_light` turn on, and `mains` turns off |
| 7a | Power flicker | `power_fail` on for under 5 s | No change, and `mains` stays on |
| 7b | Power failure | `power_fail` on for 5 s | `power_backup_mode` and `yellow_light` turn on, and `mains` turns off |
| 7c | Generator online | `power_fail` on for 10 s | `generator_ready`, `generator` and `generator_light` turn on |
| 8 | Mains restored | `power_fail` off | Backup mode drops, `mains` turns on and the generator turns off |
| 9 | Critical shutdown during a power failure | `critical_shutdown` on, then `power_fail` on | Backup timer and generator warm-up are blocked, and `mains` stays off |
| 10a | Smoke, brief | `smoke_detector` on for under 5 s | `smoke_alarm` turns on and `power_relay` is energised, but no gas is released |
| 10b | Smoke, sustained | `smoke_detector` on for 5 s or more | Gas suppression turns on, and powered loads (fans, AC, mains, generator) are off |
| 11 | Stop during a fire | `smoke_detector` on, then `stop` | `CR` drops, so the alarm and suppression drop out too (see the design review) |

## Scenario 10b timeline

| Time | What happens |
|------|--------------|
| 0 s | `smoke_alarm` turns on, `power_relay` is energised, and fans, AC, mains and generator turn off |
| 5 s | `fire_suppression_del` finishes and `gas_based_fire_suppression` turns on |
