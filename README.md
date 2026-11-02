# PLC Ladder Logic Project - Allen Bradley Studio 5000

Main routine ladder logic (Rungs 0-20) for a power/cooling/fire-safety control system, built in Allen Bradley Studio 5000.

## Files
- `PLC_ladder_rungs_0-10.png` - ladder diagram, rungs 0-10
- `PLC_ladder_rungs_11-20.png` - ladder diagram, rungs 11-20
- `PLCPROJ.docx` - project report/documentation

## Demo video
https://drive.google.com/file/d/1kpZR0ttCyFPmK2jvLmLAvJLmI5sSEqdf/view?usp=sharing

## Logic overview
| Rungs | Function |
|-------|----------|
| 0 | Start/stop seal-in circuit driving the control relay (CR) |
| 1-2 | Cooling fans and secondary cooling fan (temp_high) |
| 3-5 | Overheat detection, AC unit, overheat light |
| 6-8 | Critical temperature TON timer (5 s) -> critical shutdown + light |
| 9-10 | Power fail backup timer (5 s) -> power_backup_mode |
| 11-12 | Yellow light (backup mode) and mains indication |
| 13-16 | Generator warm-up timer (5 s) -> generator ready -> generator + light |
| 17-18 | Smoke detector -> smoke alarm and power_relay |
| 19-20 | Fire suppression delay timer (5 s) -> gas-based fire suppression |

## Author
Shreyas R - B.E. Electronics and Instrumentation Engineering, M.S. Ramaiah Institute of Technology (USN: 1MS23EI049)
