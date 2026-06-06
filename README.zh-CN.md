# WHU COAD Experiment

[English](README.md)

WHU COAD Experiment 是一个面向计算机组成与体系结构设计实验的 Vivado 与 Verilog HDL 项目集合。仓库包含多个 FPGA 实验项目、数码管显示模块、基于 ROM 的指令演示，以及一个简单单周期 CPU 实验。

项目使用 Xilinx Vivado 2018.3 创建，目标器件为 Xilinx Artix-7 `xc7a100tcsg324-1`。

![Vivado 项目预览](image.png)

## 项目指南

关于实验递进关系、模块职责、CPU 数据通路、显示交互方式与推荐阅读顺序，请查看 [项目指南](docs/PROJECT_GUIDE.zh-CN.md)。

## 项目特点

- 提供多个独立实验任务的 Vivado 工程入口文件 (`.xpr`)。
- 包含数码管显示、ROM 指令演示、开关与 LED 交互、CPU 数据通路等 Verilog HDL 实现。
- 单周期 CPU 源码包含 `PC`、`NPC`、`RF`、`alu`、`ctrl`、`dm`、`EXT`、`seg7x16` 与 `SCPU_TOP` 等模块。
- 保留 Vivado IP 配置文件 (`.xci`) 与 COE 存储器初始化文件，便于在 Vivado 中重新生成工程输出。

## 实验概览

本仓库按计算机组成实验的递进关系组织：

1. 基础 FPGA 输入输出与数码管显示控制。
2. 基于 ROM 的数据与指令显示。
3. 通过 FPGA 开关与数码管观察寄存器堆和 ALU。
4. 数据存储器访问与显示。
5. 将取指、控制逻辑、立即数扩展、寄存器堆、ALU、数据存储器、下一条 PC 逻辑和显示输出集成为单周期 CPU 演示。

后续 CPU 相关工程延续了同一套硬件交互方式：通过开关选择执行或显示模式，通过板载时钟分频生成 CPU 时钟，并使用八位数码管观察指令、寄存器值、ALU 值和存储器值等内部数据。

## 仓库结构

| 路径 | 说明 |
| --- | --- |
| `11.24/` | 早期 Vivado 实验项目，顶层模块为 `sccomp`。 |
| `11.24.2/` | Snake 实验项目，顶层模块为 `snake`。 |
| `11.24.3/` | ROM 演示项目，使用 `Test_8_Instr1.coe`。 |
| `12.8/` | CPU 相关任务项目，顶层模块为 `task1`。 |
| `12.8.2/` | CPU 相关任务项目，顶层模块为 `task2`。 |
| `12.8.3/` | CPU 相关任务项目，顶层模块为 `task3`。 |
| `12.21test/` | 单周期 CPU 测试项目，顶层模块为 `SCPU_TOP`。 |
| `COAD_test/` | 单周期 CPU 实验的独立 Verilog 源码集合。 |
| `project_guessdigital/` | 数字猜测与测试项目，包含仿真 testbench。 |
| `project_seg7to18/` | 数码管显示转换项目。 |
| `icf.xdc` | 多个 Vivado 项目共用的 FPGA 约束文件。 |
| `Test_8_Instr1.coe`, `12.21test.coe` | 分布式存储器 IP 使用的初始化文件。 |

## CPU 模块概览

CPU 实验以 `SCPU_TOP` 为核心，连接指令 ROM、控制器、立即数扩展、程序计数器、寄存器堆、ALU、数据存储器和数码管显示驱动。

| 模块 | 职责 |
| --- | --- |
| `SCPU_TOP` | 顶层集成、时钟分频、取指连线、显示选择与用户开关交互。 |
| `ctrl` | 解码指令字段，生成寄存器写、存储器写、ALU 操作、立即数类型、写回来源和下一条 PC 模式等控制信号。 |
| `PC` | 保存并更新程序计数器，并提供由开关控制的指令演示入口地址。 |
| `NPC` | 为顺序执行、分支、`jal` 和 `jalr` 计算下一条 PC。 |
| `EXT` | 为 I-type、S-type、B-type、J-type 指令生成符号扩展立即数。 |
| `RF` | 实现 32 个寄存器、两个读端口和一个写端口的寄存器堆。 |
| `alu` | 执行当前实验集合使用的算术运算，并生成 Zero 标志。 |
| `dm` | 实现按字节寻址的数据存储器，支持字、半字和字节访问形式。 |
| `seg7x16` | 以十六进制模式或原始段码模式驱动八位数码管。 |

## 环境要求

- Xilinx Vivado 2018.3 或兼容版本。
- 支持 `xc7a100tcsg324-1` 的 FPGA 器件库。
- Verilog HDL 与 Vivado 工程使用基础。

使用较新的 Vivado 版本打开项目时，Vivado 可能提示升级工程元数据或重新生成 IP 输出文件。

## 使用方式

克隆仓库：

```bash
git clone https://github.com/ecoreal/WHU_COAD_Experiment.git
cd WHU_COAD_Experiment
```

选择对应目录下的 `.xpr` 文件打开 Vivado 项目。例如：

```bash
vivado 12.21test/12.21test.xpr
```

也可以在 Vivado 图形界面中通过 **File > Open Project** 选择需要打开的 `.xpr` 文件。

打开工程后，如 Vivado 提示需要生成 IP 输出文件，可按提示重新生成，然后根据需要执行综合、实现与比特流生成。

## 项目资产

仓库保留重新打开与复现实验所需的源码和配置文件：

- 各项目 `*.srcs/sources_1/new/` 目录下的 Verilog 源码。
- Vivado 工程文件 (`.xpr`)。
- Vivado IP 配置文件 (`.xci`)。
- FPGA 约束文件 (`.xdc`)。
- 存储器初始化文件 (`.coe`)。

Vivado 生成的 `.runs`、`.cache`、`.sim`、`.hw`、`.ip_user_files`、报告、检查点与比特流等文件不纳入版本控制，可在本地通过 Vivado 重新生成。

## 开源许可

本项目采用 MIT License 开源，详见 [LICENSE](LICENSE)。

Vivado IP 核及其生成产物遵循对应的 Xilinx/Vivado 许可条款。
