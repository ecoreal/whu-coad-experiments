# Project Guide

[简体中文](PROJECT_GUIDE.zh-CN.md)

This document explains the structure and design intent of WHU COAD Experiment. It is intended for readers who want to understand how the Vivado projects relate to each other and how the single-cycle CPU experiment is organized.

## 1. Project Positioning

WHU COAD Experiment is a teaching-oriented Verilog HDL repository for Computer Organization and Architecture Design labs. The projects are designed around FPGA observation rather than a purely software-based simulator:

- Switches (`sw_i`) are used to choose display modes, execution speed, reset entry points, and data observation targets.
- The board clock is divided to generate a slower CPU clock suitable for manual observation.
- The eight-digit seven-segment display is used as the primary output device.
- Vivado distributed memory IP is used as instruction ROM, initialized from `.coe` files.

The repository keeps the source assets needed to reopen and regenerate the projects, while generated Vivado build outputs are excluded from version control.

## 2. Learning Flow

The projects can be read as a gradual path from board-level I/O to a CPU demonstration.

| Stage | Main Goal | Representative Paths |
| --- | --- | --- |
| Display and board I/O | Learn clock division, switch input, and seven-segment scanning. | `project_seg7to18/`, `11.24.2/` |
| ROM observation | Load fixed data or instructions from distributed memory and display them. | `11.24.3/`, `12.8/` |
| Register file and ALU | Observe register values, arithmetic results, and Zero flag behavior. | `12.8/`, `12.8.2/`, `12.8.3/` |
| Memory and CPU datapath | Combine instruction fetch, decode, execution, memory access, and write-back. | `12.21test/`, `COAD_test/` |
| Standalone source reading | Study the CPU modules without opening Vivado first. | `COAD_test/` |

## 3. Vivado Project Entries

Each project directory contains a `.xpr` file that can be opened directly in Vivado.

| Project | Top Module | Purpose |
| --- | --- | --- |
| `11.24/11.24.xpr` | `sccomp` | Early integrated experiment with display and ROM-related logic. |
| `11.24.2/11.24.2.xpr` | `snake` | Seven-segment display pattern experiment using switch-controlled display modes. |
| `11.24.3/11.24.3.xpr` | `sccomp` | ROM demonstration using `Test_8_Instr1.coe` and distributed memory IP. |
| `12.8/12.8.xpr` | `task1` | Register file and ROM observation task. |
| `12.8.2/12.8.2.xpr` | `task2` | ALU task with switch-provided operands and seven-segment output. |
| `12.8.3/12.8.3.xpr` | `task3` | Extended CPU-related observation task. |
| `12.21test/12.21test.xpr` | `SCPU_TOP` | Single-cycle CPU integration and instruction execution demonstration. |
| `project_guessdigital/project_guessdigital.xpr` | `test` / `test_tb` | Small digital logic project with a simulation testbench. |
| `project_seg7to18/project_seg7to18.xpr` | `top` | Seven-segment display conversion project. |

## 4. Single-Cycle CPU Architecture

The CPU experiment uses `SCPU_TOP` as the integration module. The design follows a classic single-cycle datapath structure:

1. `PC` provides the current instruction address.
2. The instruction ROM IP (`dist_mem_gen_10`) reads the instruction from `12.21test.coe`.
3. `SCPU_TOP` extracts opcode, function fields, register indices, and immediate fields from the instruction.
4. `ctrl` decodes the instruction and generates control signals.
5. `EXT` generates the selected immediate.
6. `RF` reads source registers and writes back results.
7. `alu` computes arithmetic results and the Zero flag.
8. `dm` handles data memory reads and writes.
9. `NPC` computes the next program counter.
10. `seg7x16` displays selected internal values for observation.

The design is intentionally compact and focused on classroom visibility. It emphasizes how CPU components are connected and how internal signals can be inspected on FPGA hardware.

## 5. CPU Module Responsibilities

### `SCPU_TOP`

`SCPU_TOP` is the top-level CPU experiment module. It performs clock division, connects all datapath modules, extracts instruction fields, selects display data, and maps switch inputs to observation behavior.

Important behavior:

- `sw_i[15]` selects CPU clock speed after division.
- `sw_i[1]` controls whether instruction stepping is enabled.
- `sw_i[14:11]` selects displayed internal data.
- `sw_i[13]`, `sw_i[12]`, and `sw_i[11]` step through register, ALU, and data-memory observation values.
- `sw_i[0]` selects raw display-pattern mode versus hexadecimal data display mode.

### `ctrl`

`ctrl` decodes instruction fields and generates datapath control signals:

- `RegWrite` enables register file write-back.
- `MemWrite` enables data memory writes.
- `EXTOp` selects immediate extension format.
- `ALUOp` selects the ALU operation.
- `ALUSrc` selects register or immediate as the ALU second operand.
- `DMType` selects byte, halfword, or word memory access format.
- `WDSel` selects write-back data from ALU, data memory, or PC+4.
- `NPCOp` selects sequential, branch, `jal`, or `jalr` next-PC behavior.

### `PC` and `NPC`

`PC` stores the current instruction address. On reset, selected switch values can map the PC to preset entry addresses, which is useful for observing different instruction demonstrations.

`NPC` computes the next address:

- `PC + 4` for normal sequential execution.
- `PC + immout` for branch and `jal`.
- `aluout` for `jalr`.

### `EXT`

`EXT` converts instruction immediates into 32-bit signed values. It supports I-type, S-type, B-type, and J-type immediate formats used by the CPU experiment.

### `RF`

`RF` is a 32-register file with two read ports and one write port. Register zero is read as zero, and writes to register zero are ignored through the write-address check.

### `alu`

`alu` performs arithmetic operations required by the experiment and produces the Zero flag. The Zero flag is consumed by the controller for branch decisions.

### `dm`

`dm` is a byte-addressed data memory. It supports word, halfword, byte, and sign/zero extension behavior for load-style reads. Store behavior writes the corresponding bytes into the memory array.

### `seg7x16`

`seg7x16` scans eight seven-segment digits. It supports two modes:

- Hexadecimal mode: each four-bit nibble is decoded to a display pattern.
- Raw segment mode: each byte is used directly as a segment pattern.

## 6. Display and Switch Interaction

The design uses the seven-segment display as an internal signal viewer. In the CPU project, the display source is selected by `sw_i[14:11]` when `sw_i[0] == 0`.

| Switch Pattern | Displayed Data |
| --- | --- |
| `sw_i[14:11] = 1000` | Current instruction word |
| `sw_i[14:11] = 0100` | Register file observation value |
| `sw_i[14:11] = 0010` | ALU observation value |
| `sw_i[14:11] = 0001` | Data memory observation value |
| Other values | Default display value |

When `sw_i[0] == 1`, `SCPU_TOP` displays predefined raw seven-segment patterns stored in `LED_DATA`.

## 7. Memory Initialization Files

The instruction ROMs use COE files:

- `Test_8_Instr1.coe` is used by ROM demonstration projects.
- `12.21test.coe` is used by the `12.21test` CPU project.

These files are checked in because they are part of the reproducible experiment setup. Vivado may regenerate IP products from the `.xci` configuration and these COE files when a project is opened.

## 8. Recommended Reading Order

For source-level study, read the modules in this order:

1. `COAD_test/seg7x16.v`
2. `COAD_test/RF.v`
3. `COAD_test/alu.v`
4. `COAD_test/dm.v`
5. `COAD_test/EXT.v`
6. `COAD_test/NPC.v`
7. `COAD_test/PC.v`
8. `COAD_test/ctrl.v`
9. `COAD_test/SCPU_TOP.v`

This order starts from independent utility modules and ends with top-level integration.

## 9. Rebuilding in Vivado

Open the desired `.xpr` file in Vivado 2018.3 or a compatible version. If Vivado reports missing generated products for an IP core, run **Generate Output Products** for the IP and then proceed with synthesis, implementation, and bitstream generation.

The repository intentionally does not include generated `.runs`, `.cache`, `.sim`, `.hw`, `.ip_user_files`, reports, checkpoints, or bitstreams. These files are environment-dependent and should be regenerated locally.
