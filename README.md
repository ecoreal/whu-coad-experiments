# WHU COAD Experiment

[简体中文](README.zh-CN.md)

WHU COAD Experiment is a Vivado and Verilog HDL project collection for Computer Organization and Architecture Design experiments. The repository contains FPGA-oriented lab projects, seven-segment display modules, ROM-based instruction demonstrations, and a simple single-cycle CPU experiment.

The projects were created with Xilinx Vivado 2018.3 and target the Xilinx Artix-7 device `xc7a100tcsg324-1`.

![Vivado project preview](image.png)

## Project Guide

For a detailed explanation of the experiment flow, module responsibilities, CPU datapath, display workflow, and recommended reading order, see [Project Guide](docs/PROJECT_GUIDE.md).

## Highlights

- Multiple Vivado project entries (`.xpr`) for independent lab tasks.
- Verilog HDL implementations for display control, ROM demos, switch/LED interaction, and CPU datapath modules.
- A simple CPU source set including `PC`, `NPC`, `RF`, `alu`, `ctrl`, `dm`, `EXT`, `seg7x16`, and `SCPU_TOP`.
- Reproducible Vivado IP configuration through `.xci` files and checked-in COE memory initialization files.

## Experiment Overview

The repository is organized as a progressive set of computer organization experiments:

1. Basic FPGA I/O and seven-segment display control.
2. ROM-based data and instruction display.
3. Register file and ALU observation on FPGA switches and displays.
4. Data memory access and display.
5. Integration of instruction fetch, control logic, immediate extension, register file, ALU, data memory, next-PC logic, and display output into a single-cycle CPU demonstration.

The later CPU-oriented projects reuse the same hardware interaction style: switches select execution or display modes, the CPU clock is divided from the board clock, and the eight-digit seven-segment display is used to observe internal data such as instructions, register values, ALU values, and memory values.

## Repository Structure

| Path | Description |
| --- | --- |
| `11.24/` | Early Vivado experiment project with `sccomp` as the top module. |
| `11.24.2/` | Snake experiment project with `snake` as the top module. |
| `11.24.3/` | ROM demonstration project using `Test_8_Instr1.coe`. |
| `12.8/` | CPU-related task project with `task1` as the top module. |
| `12.8.2/` | CPU-related task project with `task2` as the top module. |
| `12.8.3/` | CPU-related task project with `task3` as the top module. |
| `12.21test/` | Single-cycle CPU test project with `SCPU_TOP` as the top module. |
| `COAD_test/` | Standalone Verilog source set for the CPU experiment. |
| `project_guessdigital/` | Digital guessing/test project with a simulation testbench. |
| `project_seg7to18/` | Seven-segment display conversion project. |
| `icf.xdc` | Shared FPGA constraint file used by the Vivado projects. |
| `Test_8_Instr1.coe`, `12.21test.coe` | Memory initialization files for distributed memory IP cores. |

## CPU Module Summary

The CPU experiment centers on `SCPU_TOP`, which connects the instruction ROM, control unit, immediate extension unit, program counter, register file, ALU, data memory, and seven-segment display driver.

| Module | Responsibility |
| --- | --- |
| `SCPU_TOP` | Top-level integration, clock division, instruction fetch wiring, display selection, and user switch interaction. |
| `ctrl` | Decodes instruction fields and generates control signals such as register write, memory write, ALU operation, immediate type, write-back source, and next-PC mode. |
| `PC` | Holds and updates the program counter, with switch-controlled reset entry points for instruction demonstrations. |
| `NPC` | Computes the next program counter for sequential execution, branch, `jal`, and `jalr` paths. |
| `EXT` | Produces sign-extended immediates for I-type, S-type, B-type, and J-type instructions. |
| `RF` | Implements a 32-entry register file with two read ports and one write port. |
| `alu` | Performs arithmetic operations used by the current experiment set and generates the Zero flag. |
| `dm` | Implements byte-addressed data memory with word, halfword, and byte load/store behavior. |
| `seg7x16` | Drives the eight-digit seven-segment display in hexadecimal or raw segment mode. |

## Requirements

- Xilinx Vivado 2018.3 or a compatible Vivado release.
- FPGA device support for `xc7a100tcsg324-1`.
- Basic familiarity with Verilog HDL and Vivado project workflows.

Newer Vivado versions may ask to upgrade project metadata or regenerate IP output products when a project is opened.

## Usage

Clone the repository:

```bash
git clone https://github.com/ecoreal/WHU_COAD_Experiment.git
cd WHU_COAD_Experiment
```

Open any Vivado project from the corresponding `.xpr` file. For example:

```bash
vivado 12.21test/12.21test.xpr
```

In the Vivado GUI, the same projects can be opened through **File > Open Project** and selecting the desired `.xpr` file.

After opening a project, regenerate IP output products if Vivado prompts for them, then run synthesis, implementation, and bitstream generation as needed.

## Project Assets

The repository keeps source files and configuration files required to reopen the projects:

- Verilog source files under each `*.srcs/sources_1/new/` directory.
- Vivado project files (`.xpr`).
- Vivado IP configuration files (`.xci`).
- FPGA constraints (`.xdc`).
- Memory initialization files (`.coe`).

Generated Vivado artifacts such as `.runs`, `.cache`, `.sim`, `.hw`, `.ip_user_files`, reports, checkpoints, and bitstreams are excluded from version control and should be regenerated locally.

## License

This project is licensed under the MIT License. See [LICENSE](LICENSE) for details.

Vivado IP cores and generated products are subject to the applicable Xilinx/Vivado license terms.
