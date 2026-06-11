This project is a first attempt by the team at MSEC, Bangalore to learn the process of RTL-GDSII flow using Tiny Tapeout Platform. 
The project team at MS Engineering College, Bangalore 
1.	Harshitha B R, harshithaharshitha32679@gmail.com, 8088935246
2.	Jayanth S, jayanthjayanth949@gmail.com, 8861183113
3.	Maheshwari, maheshwarikollar@gmail.com, 9008689695
4.	Mohammad Zaman, zamanpasha55@gmail.com,  8310007327
5.	Sai Prashanth, saragallasaiprashanth@gmail.com, 91130 22029
6.	Smitha S,  psmitha.s777@gmail.com, 8088847321
Project supervisor
1.	Dr. Sunitha P H, sunithaph.ece@msec.ac.in, 972368486
Sponsored and supported by 
2.	Shri. Sudarshan M. – Chairman, MSEC, Bangalore 
The work carried out by the students team involved understanding ASIC design flow using openlane and working with GF180nm design files. Before completing this project the team worked on several micro designs to get hands on expertise. Developed own code to be tested for functionality. Dr. Cyril Prasanna Raj P., Professor and CIT was able to provide necessary technical support to complete this task successfully. 
![GDS Badge](../../workflows/gds/badge.svg)
![Docs Badge](../../workflows/docs/badge.svg)
![Test Badge](../../workflows/test/badge.svg)
![FPGA Badge](../../workflows/fpga/badge.svg)

# TinyTapeout SPI Microcoded CPU

A compact **4-bit microcoded CPU** designed for **TinyTapeout (GF180MCU)**. Unlike traditional embedded CPUs, this design fetches and executes its program directly from **external SPI RAM**.

The current demonstration program implements a **4×4-bit multiplier**, where the operands are supplied through `ui_in` and the resulting 8-bit product is presented on `uo_out`.

## Quick Start

### Inputs

| Signal | Description |
|----------|-------------|
| `ui_in[7:4]` | Operand A (4-bit) |
| `ui_in[3:0]` | Operand B (4-bit) |

### Output

| Signal | Description |
|----------|-------------|
| `uo_out[7:0]` | Product (`A × B`) |

Example:

```text
A = 7
B = 9

uo_out = 63
```

---

## Features

- 4-bit microcoded CPU
- External SPI RAM program storage
- 16-instruction ISA
- Arithmetic and logic operations
- Shift-register based operations
- TinyTapeout compatible top-level wrapper
- Fully synthesizable Verilog RTL
- Cocotb and Verilog testbenches

---

## Architecture

The design consists of three primary blocks:

```text
tt_um_spi_cpu_top
├── spi_wrap
│   ├── spi_read_byte
│   └── ExecutionUnit
│       ├── InstructionDecoder
│       ├── RegisterFile
│       ├── ShiftRegister
│       ├── ArithmeticLogicUnit
│       └── Accumulator
```

### Top-Level Wrapper

`tt_um_spi_cpu_top`

Responsibilities:

- TinyTapeout I/O integration
- SPI signal routing
- Operand input interface
- Output register interface

### SPI Fetch Wrapper

`spi_wrap`

Responsibilities:

- Program Counter (PC)
- Instruction fetch FSM
- SPI read engine
- Instruction sequencing

Each byte fetched from SPI memory contains two instructions:

```text
+--------+--------+
| Opcode | Opcode |
+--------+--------+
 High      Low
Nibble    Nibble
```

The CPU executes the low-level micro-operations sequentially before advancing to the next memory byte.

### Execution Unit

The execution datapath contains:

- 4-bit Register A
- 4-bit Register B
- 8-bit Output Register O
- 8-bit Shift Register
- 8-bit Accumulator (ACC)
- Arithmetic Logic Unit (ALU)
- Instruction Decoder

---

## Instruction Set

| Opcode | Mnemonic | Description |
|----------|----------|-------------|
| `0000` | LDA  | Load operand A |
| `0001` | LDB  | Load operand B |
| `0010` | LDO  | Load output register |
| `0011` | LDSA | Load shift register from A |
| `0100` | LDSB | Load shift register from B |
| `0101` | LSH  | Shift left |
| `0110` | RSH  | Shift right |
| `0111` | CLR  | Clear accumulator |
| `1000` | SNZA | Skip if A equals zero |
| `1001` | SNZS | Skip if shift flag set |
| `1010` | ADD  | Addition |
| `1011` | SUB  | Subtraction |
| `1100` | AND  | Bitwise AND |
| `1101` | OR   | Bitwise OR |
| `1110` | XOR  | Bitwise XOR |
| `1111` | INV  | Bitwise inversion |

---

## Program Memory

The CPU fetches instructions from external SPI RAM using:

```text
READ Command : 0x03
Address Width: 16-bit
SPI Mode     : 0
Data Width   : 8-bit
```

Program memory capacity:

```text
4096 bytes
8192 instructions
```

The program counter directly maps to the lower 12 bits of the SPI address space.

---

## SPI Interface

| Signal | Direction | Description |
|----------|----------|-------------|
| CS_n | Output | Chip Select |
| MOSI | Output | Master Out Slave In |
| MISO | Input | Master In Slave Out |
| SCK | Output | SPI Clock |

### TinyTapeout Mapping

| Signal | Pin |
|----------|-----|
| CS_n | `uio_out[0]` |
| MOSI | `uio_out[1]` |
| MISO | `uio_in[2]` |
| SCK | `uio_out[3]` |

---

## Top-Level I/O

| Port | Direction | Width | Description |
|--------|-----------|--------|-------------|
| `ui_in` | Input | 8 | Operand input bus |
| `uo_out` | Output | 8 | Output register |
| `uio_in` | Input | 8 | SPI input signals |
| `uio_out` | Output | 8 | SPI output signals |
| `uio_oe` | Output | 8 | Output enables |
| `clk` | Input | 1 | System clock |
| `rst_n` | Input | 1 | Active-low reset |
| `ena` | Input | 1 | Chip enable |

---

## Testing

### Cocotb Tests

- Multiplication verification
- SPI activity verification
- Reset recovery verification
- I/O mapping verification
- ALU unit tests
- Instruction decoder tests

Run all tests:

```bash
make
```

Run an individual Verilog testbench:

```bash
iverilog -o tb_spi_read_byte.vvp \
tb_spi_read_byte.v \
spi_read_byte.v \
spi_ram_model.v

vvp tb_spi_read_byte.vvp
```

---

## Known Limitations

### Conditional Skip Instructions

`SNZA` and `SNZS` are currently non-functional.

The present implementation performs an additional ALU operation instead of modifying instruction flow. This limitation does not affect the supplied multiplication microprogram.

### No HALT Instruction

The CPU continuously fetches and executes instructions.

### SPI Clocking

The SPI master currently operates directly from the system clock. A divider may be required for real external memory devices.

---

## Current Demonstration

The included microcode implements a shift-and-add multiplier:

```text
Input:
  A = ui_in[7:4]
  B = ui_in[3:0]

Output:
  uo_out = A × B
```

---

## License

See the repository license file for licensing information.
