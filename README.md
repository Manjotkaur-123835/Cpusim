# Computer System Architecture – CPU Sim Lab

**Name:** Manjot Kaur  
**Roll No:** 26570036

Practicals for **Computer System Architecture** (DSC02 / DSC03 / GE2c), Ramanujan College, University of Delhi.

All programs run on a machine built in **CPU Sim** that models **Mano's Basic Computer**: a 16-bit, single-accumulator computer with 4096 words of memory.

## Requirements

- Java 8 with JavaFX (for example Azul Zulu JDK FX 8)
- CPU Sim 4.0.11
- The machine file `BasicComputer.cpu` (built in Practical 1)

## How to run a program

1. Start CPU Sim and open the machine: **File → Open machine → `BasicComputer.cpu`**
2. Open the program: **File → Open text**
3. Assemble and load: **Ctrl+2**
4. Run with **Ctrl+R**, or step through with **Ctrl+D** (debug mode) and **Step by Instr**
5. For programs that use `INP`, type a number in the yellow console and press **Enter**

## Practicals

| No. | Practical | What it does |
|----|-----------|--------------|
| 1 | Create a machine | Builds the Basic Computer: registers, memory, microinstructions, fields and machine instructions |
| 2 | Fetch routine | Builds the fetch and decode steps of the instruction cycle |
| 3 | ADD | Reads two numbers, prints their sum |
| 4 | SUBTRACT | Reads two numbers, prints A − B using the 2's complement (`CMA`, `INC`, `ADD`) |
| 5 | Logical operations | AND, OR, NOT, XOR, NOR, NAND built from `AND` and `CMA` |
| 6 | Memory-reference instructions | `ADD`, `LDA`, `STA`, `BUN`, `ISZ`: multiplies 5 × 3 by repeated addition (result 15) |
| 7 | Register-reference: `CLA`, `CMA`, `CME`, `HLT` | Clears AC, complements AC, complements E, halts |
| 8 | Register-reference: `INC`, `SPA`, `SNA`, `SZE` | Increments AC and tests the skip instructions |
| 9 | Register-reference: `CIR`, `CIL` | Rotates E and AC right and left |
| 10 | Sum until a negative number | Adds numbers until a negative one is typed |
| 11 | Sum until zero | Adds numbers until zero is typed |

## Instruction set used

| Symbol | Hex | Meaning |
|--------|-----|---------|
| AND | 0xxx | AC ← AC ∧ M[addr] |
| ADD | 1xxx | AC ← AC + M[addr], E ← carry |
| LDA | 2xxx | AC ← M[addr] |
| STA | 3xxx | M[addr] ← AC |
| BUN | 4xxx | PC ← addr |
| ISZ | 6xxx | M[addr] ← M[addr] + 1, skip next if result is 0 |
| CLA / CLE | 7800 / 7400 | Clear AC / clear E |
| CMA / CME | 7200 / 7100 | Complement AC / complement E |
| CIR / CIL | 7080 / 7040 | Circulate right / left through E |
| INC | 7020 | AC ← AC + 1 |
| SPA / SNA | 7010 / 7008 | Skip if AC positive / negative |
| SZA / SZE | 7004 / 7002 | Skip if AC is zero / E is zero |
| HLT | 7001 | Halt |
| INP / OUT | F800 / F400 | Read an integer into AC / print AC |

## Registers

| Register | Bits | Purpose |
|----------|------|---------|
| AC | 16 | Accumulator |
| DR | 16 | Data register |
| AR | 12 | Address register |
| PC | 12 | Program counter |
| IR | 16 | Instruction register |
| E | 1 | Carry / extended bit |
| S | 1 | Start/stop flip-flop |
| TMP | 1 | Scratch bit used by `CIR` and `CIL` |

## Sample results

Practicals 3, 4, 5, 10 and 11 read numbers from the console. Practicals 6 to 9 need no input, so their results are the final register values.

| Practical | Input | Output |
|-----------|-------|--------|
| 3 | 5, 3 | 8 |
| 4 | 9, 3 | 6 |
| 5 | A = 1, B = 0 | 0, 1, −2, −1, 1, −2, −1 (AND, OR, NOT A, NOT B, XOR, NOR, NAND) |
| 6 | (none) | AC = 15, PROD = 15 |
| 7 | (none) | AC = −1 (65535), E = 1 |
| 8 | (none) | AC = 1, E = 0 |
| 9 | (none) | AC = 9, E = 0 |
| 10 | 4, 10, 6, 0, −3 | 20 |
| 11 | 8, 12, −5, 0 | 15 |

### Final register values after HLT

| Practical | AC | DR | AR | E | IR | PC |
|-----------|----|----|----|---|----|----|
| 3 | 8 | 5 | 1 | 0 | 28673 | 6 |
| 4 | 6 | 9 | 1 | 1 | 28673 | 9 |
| 6 | 15 | 15 | 1 | 0 | 28673 | 7 |
| 7 | −1 | 25 | 1 | 1 | 28673 | 5 |
| 8 | 1 | −2 | 1 | 0 | 28673 | 11 |
| 9 | 9 | 9 | 1 | 0 | 28673 | 6 |
| 10 | 20 | 20 | 1 | 0 | 28673 | 9 |
| 11 | 15 | 15 | 1 | 1 | 28673 | 10 |

Practical 5 is left out of this table because it prints seven outputs, which are listed above. A 1-bit register such as E or S shows as −1 when it holds 1 in **Dec** mode.

## Repository layout

```
.
├── README.md
├── machine/
│   └── BasicComputer.cpu
└── programs/
    ├── P03_ADD.a
    ├── P04_SUBTRACT.a
    ├── P05_LOGIC.a
    ├── P06_MEMORY_REFERENCE.a
    ├── P07_REGISTER_REF_CLA_CMA_CME_HLT.a
    ├── P08_REGISTER_REF_INC_SPA_SNA_SZE.a
    ├── P09_REGISTER_REF_CIR_CIL.a
    ├── P10_SUM_UNTIL_NEGATIVE.a
    └── P11_SUM_UNTIL_ZERO.a
```

## Notes

- Numbers are 16-bit two's complement, so the range is −32768 to +32767.
- In the Registers pane, **Dec** shows signed values and **Unsigned Dec** shows plain binary values. A 1-bit register holding 1 shows as −1 in **Dec**.
