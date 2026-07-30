# Scoreboard: Hazard Analysis and Dynamic Scheduling

This report accompanies the Scoreboard Simulator developed for CMP 1059 — Computer Architecture.

[← Back to the project README](../README.md)

## 1. Introduction

The project examines program execution under the Scoreboard dynamic-scheduling technique. Its purpose is to identify RAW (Read After Write), WAR (Write After Read), WAW (Write After Write), and structural hazards, and to discuss register renaming as a response to false dependencies.

The analysis uses the Scoreboard simulator maintained by Professor Israel Koren at UMass and the interactive HTML simulator in this repository. Both follow the classic CDC 6600 scheduling model described by Hennessy and Patterson.

## 2. Scoreboard Fundamentals

In a statically scheduled pipeline, a blocked instruction can prevent later independent instructions from advancing. Dynamic scheduling allows independent instructions to proceed when operands and functional units are available. Instructions are issued in order, but execution and completion may occur out of order.

### Four stages

1. **Issue:** checks whether the required functional unit is free and whether issuing the instruction would create a WAW hazard.
2. **Read Operands:** waits until source operands are available, resolving RAW dependencies.
3. **Execution:** performs the operation for the configured functional-unit latency.
4. **Write Result:** delays a write when an earlier instruction still needs to read the destination register, preventing WAR hazards.

### Hazard types

| Hazard | Meaning | Scoreboard response |
|---|---|---|
| RAW | A later instruction needs a value that has not been written | Delay operand reading |
| WAR | A later instruction would overwrite a value an earlier instruction still needs | Delay result writing |
| WAW | Two active instructions target the same register | Delay issue |
| Structural | A required functional unit is occupied | Delay issue |

## 3. Program and Configuration

```text
1. LD F6, 34+R2
2. LD F2, 45+R3
3. MULTD F0, F2, F4
4. SUBD F8, F6, F2
5. DIVD F10, F0, F6
6. ADDD F6, F8, F2
```

The configuration uses one Integer unit, two Multiply units, one Add unit, and one Divide unit. The configured latencies are one cycle for Integer/Load, two for floating-point Add/Subtract, ten for Multiply, and forty for Divide.

## 4. Simulation Results

| Instruction | Issue | Read Op | Exec Complete | Write Result |
|---|---:|---:|---:|---:|
| LD F6, 34+R2 | 1 | 2 | 3 | 4 |
| LD F2, 45+R3 | 5 | 6 | 7 | 8 |
| MULTD F0, F2, F4 | 9 | 10 | 20 | 21 |
| SUBD F8, F6, F2 | 10 | 22 | 24 | 25 |
| DIVD F10, F0, F6 | 11 | 26 | 66 | 67 |
| ADDD F6, F8, F2 | 26 | 27 | 29 | 30 |

The final instruction completion recorded by the simulation occurs at clock cycle 67.

### Observed dependencies

- The second load waits for the single Integer unit, demonstrating a structural hazard.
- `MULTD` waits for `F2` from the preceding load, while `DIVD` later waits for `F0` from `MULTD`; both are RAW dependencies.
- `ADDD` targets `F6`, which `DIVD` must first read. The Scoreboard preserves that ordering to avoid a WAR hazard.
- The included program has no explicit WAW hazard; WAW is discussed as a separate theoretical case.

## 5. WAW Example and Register Renaming

Consider two instructions that both target `F0`:

```text
MULTD F0, F2, F4
ADDD  F0, F6, F8
```

The second instruction cannot be issued until the first has written its result. Renaming the second destination to a temporary physical register removes the output dependency. The same principle removes WAR anti-dependencies when a later result is assigned a different physical register.

Register renaming does not remove true RAW dependencies. It removes name-based dependencies so that independent work can proceed.

## 6. Conclusion

The simulator provides a concrete view of in-order issue, out-of-order execution, functional-unit occupancy, and the Scoreboard's treatment of data hazards. The included sequence demonstrates RAW, WAR, and structural constraints, while the WAW example motivates register renaming.

## References

- Hennessy, John L., and David A. Patterson. *Computer Architecture: A Quantitative Approach*. 4th ed.
- Stallings, William. *Computer Organization and Architecture*. 8th ed.
- Israel Koren, [Scoreboard Simulator](http://www.ecs.umass.edu/ece/koren/architecture/scoreboard/).
