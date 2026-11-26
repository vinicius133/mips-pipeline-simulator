# ⚙️ MIPS 5-Stage Pipeline & Hazard Simulator

![Language](https://img.shields.io/badge/Language-C-blue.svg)
![Build](https://img.shields.io/badge/Build-Makefile%20%7C%20GCC-brightgreen.svg)
![Course](https://img.shields.io/badge/IFB-Computer%20Architecture%20I-orange.svg)
![Status](https://img.shields.io/badge/Status-In%20Development-yellow.svg)

A cycle-accurate **5-stage MIPS pipeline simulator and hazard analyzer** built from scratch in **C**. Designed to parse raw MIPS Assembly (`.asm`) source files, detect data and control hazards automatically, insert pipeline stalls, bubbles, and flushes, and generate comprehensive execution reports with performance metrics (CPI).

Developed as the final project for the **Computer Architecture** course at **Instituto Federal de Brasília (IFB)**.

---

## 📋 Table of Contents

- [Overview & Pipeline Model](#-overview--pipeline-model)
- [Features](#-features)
- [Supported Instruction Set](#-supported-instruction-set)
- [Project Architecture](#-project-architecture)
- [Getting Started](#-getting-started)
  - [Prerequisites](#prerequisites)
  - [Compilation](#compilation)
  - [Usage](#usage)
- [Example Output](#-example-output)
- [Team & Responsibilities](#-team--responsibilities)
- [Git Workflow](#-git-workflow)

---

## 🧠 Overview & Pipeline Model

Rather than emulating full arithmetic execution or physical memory, this tool focuses on **instruction flow, dependency resolution, and hazard mechanics** across a classic 5-stage RISC pipeline (`IF` -> `ID` -> `EX` -> `MEM` -> `WB`).

### Architectural Rules Implemented
| Pipeline Characteristic | Behavior in Simulator |
| :--- | :--- |
| **Execution Order** | Strictly in-order across 5 stages (`IF`, `ID`, `EX`, `MEM`, `WB`) |
| **Data Forwarding** | **Disabled** (hazards are resolved strictly via stalls until `WB`) |
| **Register File Timing** | Write-Back (`WB`) occurs in the first half of the cycle, allowing Instruction Decode (`ID`) to read updated registers in the same cycle |
| **Branch Resolution** | Resolved in the Execute (`EX`) stage |
| **Taken Branch Penalty** | Flushes (`DESCARTE`) the two younger instructions in `IF` and `ID` |
| **Memory Architecture** | Harvard model (separate instruction and data memories. No structural hazards) |

---

## ✨ Features

- **Dynamic `.asm` Lexical & Syntax Parser:** Strips comments (`#`), ignores blank lines and assembler directives (`.text`, `.globl`), maps standalone and inline labels, and validates syntax with line-accurate error reporting.
- **Automatic Hazard Detection:**
  - **Data Hazards (RAW):** Detects register dependencies between producer and consumer instructions, stalling `ID`/`IF` and injecting `BOLHA` (NOPs) into `EX` until `WB` completes.
  - **Load-Use Hazards:** Separately tracks and categorizes immediate dependencies following `LW` instructions without double-counting standard RAW stalls.
  - **Control Hazards:** Detects conditional branches (`BEQ`), flushes speculative instructions when taken, and redirects the Program Counter (`PC`) to the target label.
- **Dual Reporting (CLI + File Export):** Prints formatted cycle-by-cycle pipeline tables and hazard breakdowns directly to the terminal while automatically exporting an identical copy to `relatorio_simulacao.txt`.
- **Performance Metrics:** Computes total cycles, completed instructions, discarded instructions, stall cycles, injected bubbles, and average **CPI** (`CPI = Total Cycles / Completed Instructions`).

---

## 🛠️ Supported Instruction Set

The parser strictly enforces the five core MIPS instructions defined in the specification. Any unsupported instruction triggers a descriptive error with the exact line number and halts execution safely.

| Instruction | Format | Description | Destination | Source(s) |
| :--- | :--- | :--- | :--- | :--- |
| **`ADD`** | `ADD $rd, $rs1, $rs2` | Register addition | `$rd` | `$rs1`, `$rs2` |
| **`SUB`** | `SUB $rd, $rs1, $rs2` | Register subtraction | `$rd` | `$rs1`, `$rs2` |
| **`LW`** | `LW $rd, offset($rs1)` | Load word from memory | `$rd` | `$rs1` (base) |
| **`SW`** | `SW $rs2, offset($rs1)` | Store word to memory | *None* | `$rs2`, `$rs1` |
| **`BEQ`** | `BEQ $rs1, $rs2, LABEL` | Branch if equal | *None* | `$rs1`, `$rs2` |

---

## 📂 Project Architecture

The codebase is modularized into decoupled components with clear header contracts to allow parallel development and isolated testing:

```text
mips-pipeline-simulator/
├── Makefile                   # Build automation (GCC with -Wall -Wextra)
├── README.md                  # Project documentation
├── .gitignore                 # Ignores binaries, object files, and generated reports
├── docs/
│   └── technical_report.md    # Technical overview of data structures and algorithms
├── tests/
│   └── sample_program.asm     # Official benchmark Assembly test file
├── include/
│   ├── types.h                # Shared structs, enums, and constants
│   ├── parser.h               # Assembly reader & tokenizer interface
│   ├── simulator.h            # Pipeline state machine & hazard engine interface
│   └── report.h               # Terminal & file report formatter interface
└── src/
    ├── main.c                 # CLI entry point and module orchestrator
    ├── parser.c               # .asm parsing, label resolution, and validation
    ├── simulator.c            # 5-stage simulation loop and hazard counters
    └── report.c               # Table rendering and relatorio_simulacao.txt output
```

---

## 🚀 Getting Started

### Prerequisites
- **GCC**
- **GNU Make**

### Compilation
Clone the repository and build the executable using `make`:

```bash
git clone https://github.com/vinicius133/mips-pipeline-simulator.git
cd mips-pipeline-simulator
make
```

To clean compiled object files and binaries:
```bash
make clean
```

### Usage
Run the simulator by passing the path to a `.asm` file as a command-line argument:

```bash
./simulator tests/sample_program.asm
```

Alternatively, run `./simulator` without arguments to enter the file path interactively when prompted. Upon completion, check the terminal output or open the generated **`relatorio_simulacao.txt`** file in the root directory.

---

## 📊 Example Output

When running the official benchmark (`tests/sample_program.asm`), the simulator produces a cycle-by-cycle trace and performance summary:

```text
========================================================================
                 SIMULATION REPORT - MIPS 5-STAGE PIPELINE
========================================================================
Analyzed File: tests/sample_program.asm
Instructions Identified: 11 | Completed: 9 | Discarded: 2 | Total Cycles: 19

Cycle | IF             | ID             | EX       | MEM      | WB
------+----------------+----------------+----------+----------+---------
  1   | I1             |                |          |          |
  2   | I2             | I1             |          |          |
  3   | I3             | I2             | I1       |          |
  4   | I4             | I3             | I2       | I1       |
  5   | I5             | I4             | I3       | I2       | I1
  6   | I6 (ESPERA)    | I5 (ESPERA)    | I4       | I3       | I2
  7   | I6 (ESPERA)    | I5 (ESPERA)    | BOLHA    | I4       | I3
  8   | I6             | I5             | BOLHA    |          | I4
  ...
 14   | I10 (DESCARTE) | I9 (DESCARTE)  | I8       | I7       |
 15   | I11            |                |          | I8       | I7
  ...
 19   |                |                |          |          | I11

Performance Summary:
- Data Hazard (RAW) Stalls : 2 cycles
- Load-Use Hazard Stalls   : 2 cycles
- Bubbles Injected (EX)    : 4
- Discarded Instructions   : 2
- Average CPI              : 2.11 (19 / 9)
```

---

## 👥 Team & Responsibilities

| Member | Branch | Core Module | Responsibilities |
| :--- | :--- | :--- | :--- |
| **Vinícius** | `vinicius` | `simulator` | 5-stage pipeline state machine, RAW/Load-Use/Control hazard detection, stall/bubble/flush logic, and cycle metrics |
| **Patrícia** | `patricia` | `parser` | `.asm` file ingestion, comment/directive sanitization, label mapping, operand extraction, and syntax error handling |
| **José** | `jose` | `report` & `main` | CLI entry point, module orchestration, terminal & `.txt` table formatting, CPI calculation, and technical documentation |

---

<p align="center">
  <em>A final project in computer architecture for the bachelor’s degree program in computer science • 2026</em>
</p>