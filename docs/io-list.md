# I/O List and Tag Table

Everything the program reads, writes and keeps track of. Addresses follow the Logix format `Local:<slot>:<I|O>.Data.<bit>`, and they're taken from the ladder diagrams in `images/`.

## Digital inputs

| Tag | Address | What it is | Rung |
|-----|---------|-----------|------|
| `start` | `Local:1:I.Data.0` | Start push-button | 0 |
| `stop` | `Local:1:I.Data.1` | Stop push-button (examined as XIO) | 0 |
| `temp_high` | `Local:1:I.Data.2` | High temperature sensor | 2 |
| `temp_overheat` | `Local:2:I.Data.2` | Overheat temperature sensor | 3 |
| `critical_temp` | `Local:1:I.Data.3` | Critical temperature sensor | 6 |
| `power_fail` | `Local:1:I.Data.4` | Mains failure signal | 9 |
| `smoke_detector` | `Local:2:I.Data.1` | Smoke detector | 17, 18, 19 |

## Digital outputs

| Tag | Address | What it is | Rung |
|-----|---------|-----------|------|
| `AC_unit` | `Local:1:O.Data.0` | Air-conditioning unit | 4 |
| `cooling_fans` | `Local:1:O.Data.2` | Primary cooling fans | 1 |
| `generator_light` | `Local:1:O.Data.4` | Generator-running light | 16 |
| `secondary_cooling_fan` | `Local:1:O.Data.5` | Secondary cooling fan | 2 |
| `yellow_light` | `Local:2:O.Data.0` | Backup-mode warning light | 11 |
| `gas_based_fire_suppression` | `Local:2:O.Data.1` | Gas suppression release | 20 |
| `mains` | `Local:2:O.Data.2` | Mains supply contactor | 12 |
| `generator` | `Local:2:O.Data.3` | Generator supply contactor | 15 |
| `smoke_alarm` | `Local:2:O.Data.4` | Smoke alarm | 17 |
| `overheat_light` | `Local:2:O.Data.5` | Overheat light | 5 |
| `critical_light` | `Local:2:O.Data.6` | Critical shutdown light | 8 |

## Internal BOOL tags (no physical address)

| Tag | Set in rung | What it does |
|-----|-------------|--------------|
| `CR` | 0 | Master enable, latched by the start/stop circuit |
| `overheat` | 3 | Overheat status, used for the AC unit and light |
| `critical_shutdown` | 7 | Critical shutdown command |
| `power_backup_mode` | 10 | The system is running in backup mode |
| `generator_ready` | 14 | Generator warm-up is finished |
| `power_relay` | 18 | Smoke-triggered isolation relay. Its NC contacts sit in rungs 1, 2, 4 and 10-16 |

## Timers

All four are `TON` timers with `PRE` = 5000 ms.

| Timer | Rung | Runs when | Done bit used in rung |
|-------|------|-----------|-----------------------|
| `critical_T1` | 6 | `CR` AND `critical_temp` | 7 |
| `backup_timer` | 9 | `CR` AND `power_fail` AND NOT `critical_shutdown` | 10 |
| `generator_warmup_del` | 13 | In backup mode with no critical shutdown (full condition in the logic description) | 14 |
| `fire_suppression_del` | 19 | `CR` AND `smoke_detector` | 20 |
