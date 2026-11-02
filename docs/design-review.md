# Design Review

An honest look at the program: what we think works well, where it falls short of a real system, and what we'd change next.

## What works well

- One master enable (`CR`) with a seal-in start/stop circuit
- Critical temperature, power failure and fire all have to last 5 s before the system reacts
- A critical shutdown blocks the backup sequence and the generator
- Mains and generator never overlap (break-before-make)
- On smoke, powered loads are isolated before the gas is released
- Consistent tag names and a layered structure that's easy to follow

## Where it falls short

| # | What we noticed | Why it matters |
|---|-----------------|----------------|
| 1 | `critical_shutdown`, `overheat` and `power_backup_mode` are plain outputs, not latched | A shutdown clears itself as soon as the sensor input clears, so an operator never gets a chance to review it |
| 2 | There's no manual reset or acknowledgement | Nobody has to confirm the fault is actually fixed |
| 3 | Fire protection depends on `CR` | If the system is stopped, the smoke alarm and suppression are disabled too |
| 4 | `stop` is examined as XIO | That only works with a normally open stop button. A fail-safe design would use a normally closed button with XIC, so a broken wire also stops the system |
| 5 | About 5 s passes between mains dropping (t = 5 s) and the generator closing (t = 10 s) | Loads go unpowered unless a UPS or battery covers the gap |
| 6 | Temperatures come from discrete inputs with no hysteresis | Outputs can chatter around a threshold |
| 7 | There's no sensor-fault detection | A failed or disconnected sensor looks like "normal" |
| 8 | There's no watchdog or fault-state handling | If the controller faults, nothing forces a known safe state |
| 9 | Standard, non-safety-rated logic controls the gas release | Not suitable for life-safety use without certified equipment |

## What we'd improve

**First**
1. Latch `critical_shutdown` with `OTL` and clear it only with an operator reset input.
2. Make the smoke and fire rungs independent of `CR`, so fire protection still works when the process is stopped.
3. Add a watchdog and a defined fail-safe output state.

**Next**
4. Switch to analog temperature inputs with setpoints and hysteresis, and add sensor-fault alarms.
5. Add a manual override and an abort or hold input for the suppression release.
6. Cover the transfer gap with a UPS or battery stage.
7. Add a generator feedback input (running or fault) instead of trusting the timer alone.

**Later**
8. Add an HMI/SCADA layer for monitoring, trends and alarm management.
9. Move repeated logic into Add-On Instructions and use structured tags (UDTs).

## Standards context

- IEC 61131-3 defines the ladder diagram language used in this project.
- A real installation would use a listed fire-alarm and suppression control panel for the discharge, not a general-purpose PLC. NFPA 72 covers fire alarm systems and NFPA 2001 covers clean-agent extinguishing systems.
- Any function that needs a safety integrity level would call for a safety-rated controller (such as Rockwell GuardLogix) and a formal risk assessment.
