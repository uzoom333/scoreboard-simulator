# Scoreboard Simulator

An interactive visualization of the CDC 6600 Scoreboard dynamic-scheduling algorithm, including cycle-by-cycle execution and data-hazard detection.

## Project Overview

This project was developed for the Computer Architecture course (CMP 1059). It demonstrates the classic Scoreboard algorithm used in the CDC 6600 for dynamic instruction scheduling, following the concepts presented by Hennessy and Patterson.

For the included instruction sequence and configuration, the simulator reproduces the 67-cycle execution recorded with the [Koren Scoreboard Simulator](http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/).

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

```text
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

## Running Locally

Clone the repository and open `scoreboard_simulator.html` in a modern browser:

```bash
git clone https://github.com/uzoom333/scoreboard-simulator.git
cd scoreboard-simulator
```

No package installation, build step, or local server is required.

## Technologies

- HTML / CSS / JavaScript (vanilla, single file)
- No external dependencies

## References

- HENNESSY, J. L.; PATTERSON, D. A. *Computer Architecture: A Quantitative Approach.* 4th ed.
- STALLINGS, W. *Computer Organization and Architecture.* 8th ed.
- KOREN, I. [Scoreboard Simulator](http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/)

## Documentation

Read the [technical report](docs/REPORT.md) or download the original course document from [`docs/relatorio_trabalho2.docx`](docs/relatorio_trabalho2.docx).

## Author

Renato Morais Mundim Filho
