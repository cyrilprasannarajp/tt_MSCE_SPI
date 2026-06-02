![GDS Badge](../../workflows/gds/badge.svg)
![Docs Badge](../../workflows/docs/badge.svg)
![Test Badge](../../workflows/test/badge.svg)
# TinyTapeout SPI Microcoded CPU

A tiny **4-bit CPU** for **TinyTapeout (GF180MCU)** that executes its program from **external SPI RAM**. In the current demo, the microcode implements a **4x4-bit to 8-bit multiplier**, where the two 4-bit operands come in through `ui_in` and the 8-bit product is driven on `uo_out`.

- `ui_in[7:4] = A`
- `ui_in[3:0] = B`
- `uo_out[7:0] = A x B`

More detailed design notes are provided in [`docs/info.md`](docs/info.md).

## Project summary

This project explores a small microcoded CPU architecture suitable for TinyTapeout-style experiments with simple datapaths, external program storage, and SPI-based instruction fetch.

The CPU does not store its program internally. Instead, it reads instruction bytes from an external SPI memory device. In the current demo setup, that external device can be emulated by a microcontroller such as an RP2040 behaving like a simple SPI RAM.

## Demo behavior

The current microcode image is written to perform multiplication of two 4-bit operands:

- Operand `A` is provided on `ui_in[7:4]`
- Operand `B` is provided on `ui_in[3:0]`
- The 8-bit product is returned on `uo_out[7:0]`

Example:

- If `A = 4'b0011` and `B = 4'b0101`, then `uo_out = 8'b00001111`

## Architecture overview

The design is split into three main blocks:

1. **TinyTapeout wrapper**: `tt_um_spi_cpu_top`
2. **CPU wrapper + SPI fetch logic**: `spi_wrap`
3. **Execution datapath**: `ExecutionUnit`

### 1. TinyTapeout wrapper

The top-level module connects the TinyTapeout pad interface to the internal CPU and the external SPI-facing signals. It accepts the two 4-bit input operands on `ui_in`, returns the 8-bit result on `uo_out`, and routes SPI-related signals through the `uio_*` pins.

### 2. SPI fetch wrapper

`spi_wrap` is responsible for program sequencing and SPI fetch control. It contains:

- The program counter
- A small instruction-fetch state machine
- A byte-oriented SPI read engine
- The control interface to the execution unit

Each SPI byte contains **two 4-bit micro-operations**:

- Lower nibble executes first
- Upper nibble executes second
- The program counter then advances to the next byte

### 3. Execution datapath

`ExecutionUnit` contains the internal data path and control logic used to execute the micro-operations. At a high level, it includes:

- A, B, and O registers
- An 8-bit accumulator
- A shift register and associated status/flag logic
- A small ALU
- Decode/control logic

The instruction set is intentionally compact and is designed to be sufficient for simple arithmetic, logical, load, and shift operations needed by the multiplication demo.

## Execution flow

At a high level, execution proceeds as follows:

1. Reset initializes the control logic and program counter.
2. The CPU issues an SPI read for the current program byte.
3. The lower 4-bit micro-operation is decoded and executed.
4. The upper 4-bit micro-operation is decoded and executed.
5. The program counter increments.
6. The sequence repeats until the programmed operation completes.

## Pin mapping

> Replace any placeholder signal names below with the exact names used in your RTL.

| Signal | Direction | Description |
|---|---|---|
| `ui_in[7:4]` | Input | 4-bit operand A |
| `ui_in[3:0]` | Input | 4-bit operand B |
| `uo_out[7:0]` | Output | 8-bit result output |
| `uio_in[...]` | Input | SPI input-side signals from external memory model |
| `uio_out[...]` | Output | SPI output-side signals to external memory model |
| `uio_oe[...]` | Output | Output-enable control for bidirectional user IO pins |

If you have fixed signal assignments for SPI, expand this section with explicit names such as chip select, serial clock, MOSI, and MISO.

## Repository structure

- `src/` - Verilog source files for the TinyTapeout wrapper, SPI wrapper, and execution unit
- `test/` - Testbench and verification files
- `docs/` - Additional project documentation
- `info.yaml` - TinyTapeout project metadata
- `README.md` - Front-page overview of the project

## Current limitations

This repository currently demonstrates a specific microcoded multiply use case rather than a general-purpose programmable CPU environment.

Known limitations include:

- Program storage is external and depends on an SPI memory model
- Public documentation does not yet include a full opcode table
- Public documentation does not yet include a complete signal-level SPI interface description
- The current demo is focused on 4-bit multiplication rather than a broader instruction-programming workflow

## Next steps

Planned or recommended improvements:

- Publish the full micro-instruction encoding
- Document exact SPI signal mapping and timing
- Add a worked multiply example with state progression
- Summarize verification coverage in the documentation
- Extend the microcode and datapath for additional arithmetic operations
