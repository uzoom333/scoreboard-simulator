# ⚡ Scoreboard Simulator

Interactive simulator of the **CDC 6600 Scoreboard** dynamic scheduling algorithm. Visualizes cycle-by-cycle instruction execution, hazard detection and resolution in real time.

## About

This project was developed for the **Computer Architecture** course (CMP 1059). It simulates the classic Scoreboard algorithm used in the CDC 6600 supercomputer for dynamic instruction scheduling, as described by Hennessy & Patterson.

The simulation matches the actual output of the [Koren Simulator](http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/) (Clock Cycle 67).

## Features

- **Step-by-step execution** with forward/backward navigation
- **Auto-play mode** with adjustable speed
- **Instruction Status Table** — Issue, Read Operands, Exec Complete, Write Result
- **Functional Unit Status** — Busy, Op, Fi, Fj, Fk, Qj, Qk, Rj, Rk
- **Register Result Status** — tracks which unit writes to each register
- **Hazard detection and visualization:**
  - **RAW** (Read After Write) — true data dependency
  - **WAR** (Write After Read) — anti-dependency
  - **WAW** (Write After Write) — output dependency
  - **Structural** — functional unit conflict

## Instruction Set

```
LD F6, 34+R2
LD F2, 45+R3
MULTD F0, F2, F4
SUBD F8, F6, F2
DIVD F10, F0, F6
ADDD F6, F8, F2
```

## Functional Unit Latencies

| Unit | Latency |
|------|---------|
| Integer / Load | 1 cycle |
| FP Add / Sub | 2 cycles |
| FP Multiply | 10 cycles |
| FP Divide | 40 cycles |

## Execution Results

| Instruction | Issue | Read Op | Exec Complete | Write Result |
|---|---|---|---|---|
| LD F6, 34+R2 | 1 | 2 | 3 | 4 |
| LD F2, 45+R3 | 5 | 6 | 7 | 8 |
| MULTD F0, F2, F4 | 9 | 10 | 20 | 21 |
| SUBD F8, F6, F2 | 10 | 22 | 24 | 25 |
| DIVD F10, F0, F6 | 11 | 26 | 66 | 67 |
| ADDD F6, F8, F2 | 26 | 27 | 29 | 30 |

**Total execution: 67 clock cycles.**

## Hazards Identified

| Type | Stage | Example |
|------|-------|---------|
| RAW | Read Operands | MULTD waits for F2 from LD |
| WAR | Write Result | ADDD waits for DIVD to read F6 |
| Structural | Issue | LD F2 waits for Integer unit (4 cycles) |

## How to Run

Just open `scoreboard_simulator.html` in any browser. No dependencies, no build step, no server needed.

## Tech

- HTML / CSS / JavaScript (vanilla, single file)
- No external dependencies

## References

- HENNESSY, J. L.; PATTERSON, D. A. *Computer Architecture: A Quantitative Approach.* 4th ed.
- STALLINGS, W. *Computer Organization and Architecture.* 8th ed.
- KOREN, I. [Scoreboard Simulator](http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/)

## Documentation

The full technical report is available in `docs/relatorio_trabalho2.docx`.

---

*CMP 1059 — Computer Architecture | Built with the help of Claude Opus 4.6*
