# RV32I 5-Stage Pipelined RISC-V CPU

<p align="center">
  <img src="https://img.shields.io/badge/Language-SystemVerilog-blue?style=for-the-badge&logo=verilog" alt="SystemVerilog">
  <img src="https://img.shields.io/badge/Architecture-RISC--V-0066CC?style=for-the-badge" alt="RISC-V">
  <img src="https://img.shields.io/badge/Simulation-Icarus%20Verilog-green?style=for-the-badge" alt="Icarus Verilog">
  <img src="https://img.shields.io/badge/Synthesis-Yosys-orange?style=for-the-badge" alt="Yosys">
</p>

<p align="center">
  A 32-bit pipelined RISC-V processor designed in SystemVerilog, verified through simulation, and synthesized using Yosys.
</p>

<p align="center">
  <b>RTL Design | Digital Logic | Processor Architecture | Functional Verification | Logic Synthesis</b>
</p>

---
A 32-bit, five-stage pipelined RISC-V processor implemented in SystemVerilog, with a focus on instruction execution, pipeline control, data dependencies, and digital hardware verification.

This project explores how processor architecture translates into synthesizable RTL, from instruction decoding and ALU operations to pipeline stalls, control-flow redirection, simulation, and logic synthesis.

## Project Overview

The goal of this project is to develop a practical understanding of processor microarchitecture by implementing a pipelined CPU based on the RISC-V instruction set.

Instead of executing an entire instruction in one cycle, the processor divides instruction execution into five stages. This allows multiple instructions to occupy different stages of the pipeline simultaneously.

The implementation includes a register file, arithmetic and logic operations, branch and jump handling, word-level memory operations, and pipeline control logic.

I use **SystemVerilog for RTL development, Icarus Verilog for functional simulation, and Yosys for synthesis and structural checks.**

### Key Features

- 32-bit processor datapath based on the RISC-V architecture
- Five-stage instruction pipeline
- Register file with 32 architectural registers
- Arithmetic, logical, comparison, and shift operations
- Conditional branches and jump instructions
- Immediate generation and upper-immediate instructions
- Word-level load and store operations
- Load-use stall handling and control-flow redirection
- Directed testbenches for functional verification
- Synthesis flow using Yosys

> **Project scope:** This is an educational pipelined processor implementation. The supported instruction set and verification coverage are limited to the functionality implemented and tested in the RTL.

## Processor Architecture

The processor follows a conventional five-stage pipeline.

| Stage | Name | Function |
|---|---|---|
| IF | Instruction Fetch | Fetches the instruction using the program counter |
| ID | Instruction Decode | Decodes instruction fields and reads source registers |
| EX | Execute | Performs ALU operations, calculates addresses, and evaluates branches |
| MEM | Memory Access | Handles supported data-memory operations |
| WB | Write Back | Writes the selected result to the destination register |

### Architecture at a Glance

The main components include:

- **Program Counter:** Tracks instruction addresses and handles control-flow redirection.
- **Instruction Decode:** Extracts instruction fields and generates immediate values.
- **Register File:** Provides source operands and stores architectural register values.
- **Arithmetic Logic Unit:** Performs arithmetic, logical, shift, and comparison operations.
- **Pipeline Control:** Coordinates stalls, valid instructions, and branch or jump redirection.
- **Memory Interface:** Connects the processor to externally provided instruction and data memories.
- **Write-Back Logic:** Selects the result returned to the register file.

The instruction and data memories are external to the processor core, allowing the surrounding system or testbench to provide the required memory behavior.

## Implemented Instruction Support

The current RTL supports selected instructions from the RV32I base integer instruction set.

### Arithmetic and Logical Operations

- `ADD`, `SUB`
- `AND`, `OR`, `XOR`
- `SLT`, `SLTU`
- `SLL`, `SRL`, `SRA`

### Immediate Operations

- `ADDI`
- `ANDI`, `ORI`, `XORI`
- `SLTI`, `SLTIU`
- `SLLI`, `SRLI`, `SRAI`

### Control Flow

- `BEQ`, `BNE`
- `BLT`, `BGE`
- `BLTU`, `BGEU`
- `JAL`, `JALR`

### Upper-Immediate Instructions

- `LUI`
- `AUIPC`

### Memory Operations

- `LW` — Load Word
- `SW` — Store Word

Byte and halfword memory operations are not yet implemented in the documented load/store datapath. Support for the listed instructions should not be interpreted as complete RV32I compliance.

## Pipeline Control and Hazard Handling

One of the important aspects of a pipelined processor is maintaining correct execution when instructions depend on one another.

The implementation includes pipeline-valid signals, load-use stall handling, and control-flow redirection.

The main areas of interest are:

- **Data dependencies:** Handling situations in which an instruction requires a result produced by an earlier instruction.
- **Load-use hazards:** Managing dependencies involving values loaded from data memory.
- **Control hazards:** Preventing younger instructions from executing incorrectly after a taken branch or jump.
- **Pipeline validity:** Tracking which instructions should be considered valid as execution progresses.

These mechanisms are important because incorrect pipeline control can produce errors even when the individual ALU and register-file components work correctly.

## Verification and Simulation

Functional verification is performed using SystemVerilog testbenches and Icarus Verilog.

The existing testbenches exercise selected processor operations, including:

- Arithmetic and logical execution
- Register write-back and zero-register behavior
- Word load and store operations
- Load-use behavior
- Conditional branch decisions
- `JAL` and `JALR` control flow
- Selected pipeline flush scenarios

The testbench reports expected and observed values and provides pass/fail results for the checks it implements.

### Run the Simulation

Run the following commands from the repository root in a WSL/Linux terminal:

```bash
chmod +x run_sim.sh
./run_sim.sh
```

The supplied script runs the configured simulation flow. The actual results should be checked against the testbench output.

A passing directed test indicates that the corresponding checks succeeded; it does not establish exhaustive correctness or complete instruction-set compliance.

## Waveform Debugging

Simulation waveforms provide a way to inspect how instructions progress through the pipeline and how control signals respond to different instructions.

The testbenches generate VCD files when configured to do so.

For example:

```bash
gtkwave sim/rv32i_control.vcd
```

Useful signals to inspect include:

- `pc`
- `imem_addr`
- `idex_instr`
- `ex_alu_result`
- `dmem_addr`
- `dmem_wdata`
- `dmem_we`
- Pipeline-valid and stall signals

Waveform inspection is particularly useful for debugging register dependencies, memory accesses, pipeline stalls, and branch or jump redirection.

## RTL Synthesis with Yosys

The processor uses Yosys to process the RTL and generate a synthesized representation of the design.

Run:

```bash
yosys -s synth_run_yosys.ys
```

The configured synthesis flow includes RTL processing, optimization, structural checks, and generation of a JSON representation.

The expected output path is:

```text
synth/rv32i_core.json
```

Synthesis provides an additional way to inspect the hardware structure inferred from the RTL.

Generic synthesis results should not be confused with final ASIC area, FPGA resource utilization, or measured operating frequency. Those require appropriate target-specific implementation and analysis.

## Repository Structure

```text
RV32I-5-Stage-Pipelined-CPU/
│
├── rtl/
│   └── rv32i_core.sv
│
├── tb/
│   ├── tb_rv32i_core.sv
│   └── tb_rv32i_control.sv
│
├── sim/
│   └── Generated simulation outputs
│
├── synth/
│   └── Generated synthesis outputs
│
├── run_sim.sh
├── run_sim.bat
├── synth_run_yosys.ys
├── .gitignore
└── README.md
```

## Tools and Technologies

| Technology | Application |
|---|---|
| SystemVerilog | Processor RTL implementation |
| Icarus Verilog | Simulation and functional testing |
| GTKWave | Waveform inspection and debugging |
| Yosys | Logic synthesis and structural checks |
| Visual Studio Code | RTL development |
| WSL/Linux | Open-source hardware development environment |
| Git and GitHub | Version control and project documentation |

## Current Limitations

The project is a work in progress, with several opportunities for deeper verification and further hardware development.

- Complete RV32I compliance has not been established.
- Byte and halfword load/store operations remain future work.
- Broader randomized and formal verification has not been completed.
- FPGA implementation and board-level validation have not been performed.
- Target-specific timing analysis and ASIC physical design have not been completed.
- Interrupts, exceptions, caches, and a complete peripheral subsystem are outside the current implementation.

## Planned Improvements

### 1. Verification and Reliability

- Expand tests for consecutive dependent instructions.
- Investigate and verify forwarding behavior across pipeline stages.
- Add directed tests for load-use hazards and control-flow corner cases.
- Introduce automated regression tests and stronger result checking.
- Compare processor behavior against a trusted reference model where practical.

### 2. Instruction Set Expansion

- Add byte and halfword load/store instructions.
- Improve coverage across supported instruction encodings.
- Validate immediate generation, signed arithmetic, and branch conditions.

### 3. Performance Analysis

- Measure instruction throughput and cycles per instruction.
- Investigate pipeline stalls and control-hazard penalties.
- Examine synthesis statistics and timing under a defined target flow.
- Compare the pipelined design with a suitable baseline implementation.

### 4. Hardware System Integration

- Integrate instruction and data RAM.
- Add a memory-mapped address decoder.
- Develop UART and GPIO peripherals.
- Execute small assembly programs on the integrated system.
- Explore FPGA implementation if suitable hardware is available.

## What This Project Explores
<p align="center">
  <b>Built with SystemVerilog. Verified through simulation. Synthesized with Yosys.</b>
</p>

