# 4-Bit RISC CPU with a 3-Stage Pipeline (Verilog HDL)

Stages: **IF** (fetch) → **ID** (decode) → **EX** (execute + memory + write-back).

- One instruction finishes every clock once the pipeline is full.
- A taken jump (`JMP`, or `JZ` with Zero = 1) flushes 2 instructions (2 extra clocks).

## Design files: `rtl/`

Add as **design sources** in Vivado.

| File | Contents |
|---|---|
| `alu.v` | ALU: `adder_subtractor` (one shared adder for ADD and SUB), `logic_unit`, `zero_detector`, `alu` |
| `register_file.v` | 4 registers R0-R3, 2 read ports, 1 write port |
| `program_counter.v` | 4-bit PC |
| `instruction_memory.v` | 16 x 12-bit program memory (loads a `.mem` file) |
| `instruction_register.v` | IF/ID pipeline register (the IR) |
| `pipeline_register.v` | ID/EX pipeline register |
| `control_unit.v` | `instruction_decoder` (ID stage) + `branch_unit` (EX stage) |
| `data_memory.v` | 16 x 4-bit data memory |
| `mux.v` | 2-input 4-bit multiplexer (used 3 times) |
| `flag_register.v` | Zero and Carry flags |
| `cpu_top.v` | **Top module**: connects everything, stage by stage |

## Testbenches: `testbench/`

Add as **simulation sources**, then *Set as Top* one at a time.

| File | Tests | Checks |
|---|---|---|
| `tb_alu.v` | ALU on its own | 11 |
| `tb_register_file.v` | register file on its own | 11 |
| `tb_control_unit.v` | decoder + branch unit | 22 |
| `tb_cpu.v` | complete CPU, 4 programs | 44 |
| `tb_cpu_extra.v` | extra CPU test (`prog_extra.mem`): 15 + 1 flags, LDI / MOV keep the flags, LOAD right after STORE, use of a register right after LOAD, PC wrap 15 → 0 | 16 |

The first four testbenches give the 88 checks of the report. `tb_cpu_extra.v` is separate from them.

## Program files: `programs/`

Also add as simulation sources (choose *All files* in the file dialog).

| File | Purpose |
|---|---|
| `program.mem` | demonstration program (project plan Appendix A) |
| `prog_arith.mem` | ADD, SUB, STORE, LOAD, flags |
| `prog_logic.mem` | AND, OR, XOR, MOV, NOT, LDI, Zero |
| `prog_jump.mem` | PC increment, JMP, JZ taken / not taken |
| `prog_extra.mem` | used by `tb_cpu_extra.v` |

## Constraints: `constraints/` (optional)

`cpu_timing.xdc` is a clock constraint (`create_clock`, 10 ns) for synthesis / implementation. Change the period to find the fastest clock with positive slack. It is not needed for simulation. Pin assignments for a board are not included.

## Run

1. In Vivado: *Simulation → Run Behavioral Simulation*, then type `run all` in the Tcl console.
2. Each testbench ends with a `PASSED` / `FAILED` summary line.
3. Close the simulation (*Close Simulation*, or Tcl `close_sim`) before setting another testbench as top.

Expected: `tb_alu` 11/11, `tb_register_file` 11/11, `tb_control_unit` 22/22, `tb_cpu` 44/44, `tb_cpu_extra` 16/16.

**Synthesis note:** `instruction_memory.v` loads `program.mem` with `$readmemh`. For synthesis / elaboration (not for simulation), also add `program.mem` as a **design** source, otherwise Vivado warns `[Synth 8-4445] could not open $readmemh data file`.

## Carry convention

- ADD: Carry = 1 when the sum does not fit in 4 bits.
- SUB: Carry = 1 means **no borrow** (a ≥ b).

Both come from the same adder: `A - B = A + ~B + 1`, and carry is the carry-out (bit 4).

## Documents: `docs/`

- `4bit_RISC_CPU_Pipeline_Report.pdf`: full project report (theory + practical)
- `code_explained/Explained_*.pdf`: one walkthrough for every Verilog file except `tb_cpu_extra.v` (15 files)

> **Note:** the report (Section 4.5 and the `alu.v` listing in Chapter 11) and `Explained_alu.pdf` describe the earlier `alu.v`, where SUB used a separate subtractor and an `a >= b` comparator. The current `alu.v` uses one shared adder (same behaviour, checked for all 2048 input combinations).

## Comments in the code

Every place where a file uses a module from another file has a comment `// CALLS: module X from file Y.v`. Each file header has a CONNECTIONS list (Calls / Called by).
