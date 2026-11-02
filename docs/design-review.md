# Design Review

A critical review of the program: what it does well, where it falls short of a production system, and what to improve.

## Strengths

- Single master enable (`CR`) with a seal-in start/stop circuit
- Timer-confirmed critical temperature, power failure and fire events
- Interlocks prevent the backup sequence and generator during a critical shutdown
- Mains and generator never overlap (break-before-make)
- Smoke response isolates powered loads before gas discharge
- Clear, consistent tag naming and a layered structure

## Limitations

| # | Observation | Risk |
|---|-------------|------|
| 1 | `critical_shutdown`, `overheat` and `power_backup_mode` are non-latching (OTE) | A shutdown clears itself when the sensor input clears, so a transient reset can restore operation without operator review |
| 2 | No manual reset or acknowledgement | Operator is not required to confirm the fault is resolved |
| 3 | Fire protection is gated by `CR` | Pressing `stop` (or a stopped system) disables smoke alarm and suppression |
| 4 | `stop` is examined as XIO | Correct only for a normally-open stop button. A fail-safe design would use a normally-closed button with XIC so that a broken wire also stops the system |
| 5 | Roughly 5 s gap between mains dropping (t = 5 s) and generator closing (t = 10 s) | Loads are unpowered unless a UPS or battery bridges the gap |
| 6 | Discrete temperature inputs, no hysteresis | Outputs can chatter around a threshold |
| 7 | No sensor-fault detection | A failed or disconnected sensor reads as "normal" |
| 8 | No watchdog or fault-state handling | A controller fault does not force a known safe state |
| 9 | Standard (non-safety-rated) logic controls gas discharge | Not suitable for life-safety use without certified equipment |

## Recommended improvements

**High priority**
1. Latch `critical_shutdown` with `OTL` and clear it only with an operator reset input.
2. Make the smoke / fire rungs independent of `CR`, so fire protection works even when the process is stopped.
3. Add a watchdog and a defined fail-safe output state.

**Medium priority**
4. Use analog temperature inputs with setpoints and hysteresis, and add sensor-fault alarms.
5. Add a manual override and a suppression abort / hold input.
6. Add a UPS or battery bridging stage, or overlap-managed transfer, to cover the transfer gap.
7. Add a generator-feedback input (running / fault) instead of relying on the timer alone.

**Longer term**
8. Add an HMI / SCADA layer for monitoring, trends and alarm management.
9. Move repeated logic into Add-On Instructions and use structured tags (UDTs).

## Standards context

- **IEC 61131-3** defines the ladder diagram language used here.
- A real deployment would use a listed fire-alarm and suppression control panel (see NFPA 72 for fire alarm systems and NFPA 2001 for clean-agent extinguishing systems) rather than a general-purpose PLC for discharge.
- Safety functions that need a safety integrity level would use a safety-rated controller (for example Rockwell GuardLogix) and a formal risk assessment.
