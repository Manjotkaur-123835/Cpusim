# Computer System Architecture: CPU Sim Lab

Practical lab work for **Computer System Architecture** (DSC02 / DSC03 / GE2c), Ramanujan College, University of Delhi.

All eleven practicals are done in **CPU Sim 4.0.11** on a machine built after M. Morris Mano's **Basic Computer**.

**Name:** _your name_  **Roll No:** _your roll number_

## Contents

| No. | Practical |
|-----|-----------|
| 1 | [Create a machine (Basic Computer architecture)](#practical-1-create-a-machine-basic-computer-architecture) |
| 2 | [Create the fetch routine of the instruction cycle](#practical-2-create-the-fetch-routine-of-the-instruction-cycle) |
| 3 | [ADD operation on two user-entered numbers](#practical-3-add-operation-on-two-user-entered-numbers) |
| 4 | [SUBTRACT operation on two user-entered numbers](#practical-4-subtract-operation-on-two-user-entered-numbers) |
| 5 | [Logical operations AND, OR, NOT, XOR, NOR, NAND](#practical-5-logical-operations-and-or-not-xor-nor-nand) |
| 6 | [Memory-reference instructions ADD, LDA, STA, BUN, ISZ](#practical-6-memory-reference-instructions-add-lda-sta-bun-isz) |
| 7 | [Register-reference instructions CLA, CMA, CME, HLT](#practical-7-register-reference-instructions-cla-cma-cme-hlt) |
| 8 | [Register-reference instructions INC, SPA, SNA, SZE](#practical-8-register-reference-instructions-inc-spa-sna-sze) |
| 9 | [Register-reference instructions CIR, CIL](#practical-9-register-reference-instructions-cir-cil) |
| 10 | [Sum of integers until a negative number is read](#practical-10-sum-of-integers-until-a-negative-number-is-read) |
| 11 | [Sum of integers until zero is read](#practical-11-sum-of-integers-until-zero-is-read) |

## Repository layout

```
csa-cpusim-lab/
├── README.md
├── machine/
│   └── practical.cpu          the saved Basic Computer machine
├── programs/
│   ├── P03_ADD.a
│   ├── P04_SUBTRACT.a
│   ├── P05_LOGIC.a
│   ├── P06_MEMORY_REFERENCE.a
│   ├── P07_REGISTER_REF_CLA_CMA_CME_HLT.a
│   ├── P08_REGISTER_REF_INC_SPA_SNA_SZE.a
│   ├── P09_REGISTER_REF_CIR_CIL.a
│   ├── P10_SUM_UNTIL_NEGATIVE.a
│   └── P11_SUM_UNTIL_ZERO.a
└── screenshots/
    ├── P03_output.png  ...  P11_output.png
```

## How to run a program

1. Start CPU Sim and open the machine: **File → Open machine** and choose `machine/practical.cpu`.
2. Open the program: **File → Open text** and choose the `.a` file.
3. Press **Ctrl+2** (Assemble & load). The machine code appears in the memory pane on the right.
4. Press **Ctrl+R** (Run). When the console turns yellow, type a number and press **Enter**.
5. The result appears in the console, followed by the message `EXECUTION HALTED NORMALLY`.

## The Basic Computer in one page

It is a 16-bit computer with one main working register (the accumulator) and a memory of 4096 words. Every instruction is one 16-bit word.

| Register | Bits | What it holds |
|----------|------|---------------|
| AC | 16 | Accumulator: the main working register, where all results appear |
| DR | 16 | Data register: holds the number just read from memory |
| AR | 12 | Address register: holds the memory address to read or write |
| PC | 12 | Program counter: the address of the next instruction |
| IR | 16 | Instruction register: the instruction being carried out |
| E | 1 | Extra bit: holds the carry out of an addition, and is used by CIR and CIL |
| S | 1 | Start/stop bit: HLT sets it to 1 and the machine stops |
| TMP | 1 | Scratch bit used inside CIR and CIL |

**Instruction codes (hex)**

| Instruction | Code | What it does |
|-------------|------|--------------|
| AND | 0xxx | AC = AC AND memory word |
| ADD | 1xxx | AC = AC + memory word (carry goes to E) |
| LDA | 2xxx | AC = memory word |
| STA | 3xxx | memory word = AC |
| BUN | 4xxx | jump to the address |
| ISZ | 6xxx | add 1 to the memory word; skip the next instruction if it becomes 0 |
| CLA | 7800 | AC = 0 |
| CLE | 7400 | E = 0 |
| CMA | 7200 | flip every bit of AC |
| CME | 7100 | flip E |
| CIR | 7080 | rotate E and AC right by one bit |
| CIL | 7040 | rotate E and AC left by one bit |
| INC | 7020 | AC = AC + 1 |
| SPA | 7010 | skip the next instruction if AC is zero or positive |
| SNA | 7008 | skip the next instruction if AC is negative |
| SZA | 7004 | skip the next instruction if AC is zero |
| SZE | 7002 | skip the next instruction if E is 0 |
| HLT | 7001 | stop |
| INP | F800 | AC = number typed by the user |
| OUT | F400 | show AC on the console |

**Reading the screenshots**

- The register pane is set to **Dec**, which shows each register as a signed number. A 1-bit register holding 1 is therefore shown as **-1**. So `S = -1` means S is set (the machine halted), and `E = -1` means E is 1.
- After `HLT`, **IR = 28673** (7001 in hex) and **AR = 1**, because AR always receives the low 12 bits of the instruction, and for HLT those are `001`.
- In the assembly programs, a `;` starts a comment, a name followed by a colon (like `LOOP:`) is a label, and `.data 1 n` reserves one memory word holding the value `n`.

---

## Practical 1: Create a machine (Basic Computer architecture)

**Aim:** Create, in CPU Sim, a machine based on the Basic Computer architecture: its registers, memory, microinstructions, instruction fields and machine instructions.

**Machine file:** `machine/practical.cpu`

### Step 1: Hardware

Registers (all start at 0): AC 16, DR 16, AR 12, PC 12, IR 16, E 1, TMP 1, S 1.

Condition bits:

| Name | Register | Bit | Halt |
|------|----------|-----|------|
| carry-E | E | 0 | no |
| halt-S | S | 0 | yes (setting this bit stops the program) |

RAM: name `M`, length 4096, cell size 16.

### Step 2: Microinstructions

A microinstruction is one small register step. CPU Sim counts bits from the left, so bit 0 is the leftmost bit.

| Type | Name | What it does |
|------|------|--------------|
| TransferRtoR | PC->AR | copy PC into AR (source start 0, dest start 0, 12 bits) |
| TransferRtoR | IR(0-11)->AR | copy the address part of IR into AR (source start 4, dest start 0, 12 bits) |
| TransferRtoR | AR->PC | copy AR into PC (12 bits) |
| TransferRtoR | DR->AC | copy DR into AC (16 bits) |
| TransferRtoR | AC(0)->TMP, AC(15)->TMP, E->AC(15), E->AC(0), TMP->E | move single bits between AC, E and TMP (used by CIR and CIL) |
| MemoryAccess | M[AR]->IR, M[AR]->DR | read the memory word at AR into IR or DR |
| MemoryAccess | AC->M[AR], DR->M[AR] | write AC or DR into the memory word at AR |
| Increment | PC+1->PC, DR+1->DR, AC+1->AC | add 1 to a register |
| Arithmetic | AC+DR->AC,E | add AC and DR into AC; the carry goes to E |
| Logical | AC^DR->AC, AC'->AC, E'->E | AND, and NOT (flip every bit) |
| Shift | shr AC, shl AC | shift AC right or left by 1 bit |
| Set | 0->AC, 0->E | put 0 into AC or E |
| Test | if(DR!=0)skip-1, if(AC(15)!=0)skip-1, if(AC(15)==0)skip-1, if(AC!=0)skip-1, if(E!=0)skip-1 | skip the next microinstruction when the test is true |
| Decode | decode-IR | pick the machine instruction that matches IR |
| SetCondBit | 1->S(halt) | set the halt bit |
| IO | input-int->AC, output-AC->int | read a whole number from the console into AC, or print AC |

### Step 3: Instruction fields

| Field | Bits | Used for |
|-------|------|----------|
| op | 4 | opcode of memory-reference instructions (hex digit 0 to 6) |
| addr | 12 | address part of memory-reference instructions |
| opcode | 16 | the complete 16-bit code of register-reference and I/O instructions |

### Step 4: Machine instructions

Memory-reference instructions use the fields `op` then `addr`. All others use the single field `opcode`. Every execute sequence ends with `End`.

| Name | Opcode | Execute sequence |
|------|--------|------------------|
| AND | 0x0 | M[AR]->DR, AC^DR->AC, End |
| ADD | 0x1 | M[AR]->DR, AC+DR->AC,E, End |
| LDA | 0x2 | M[AR]->DR, DR->AC, End |
| STA | 0x3 | AC->M[AR], End |
| BUN | 0x4 | AR->PC, End |
| ISZ | 0x6 | M[AR]->DR, DR+1->DR, DR->M[AR], if(DR!=0)skip-1, PC+1->PC, End |
| CLA | 0x7800 | 0->AC, End |
| CLE | 0x7400 | 0->E, End |
| CMA | 0x7200 | AC'->AC, End |
| CME | 0x7100 | E'->E, End |
| CIR | 0x7080 | AC(0)->TMP, shr AC, E->AC(15), TMP->E, End |
| CIL | 0x7040 | AC(15)->TMP, shl AC, E->AC(0), TMP->E, End |
| INC | 0x7020 | AC+1->AC, End |
| SPA | 0x7010 | if(AC(15)!=0)skip-1, PC+1->PC, End |
| SNA | 0x7008 | if(AC(15)==0)skip-1, PC+1->PC, End |
| SZA | 0x7004 | if(AC!=0)skip-1, PC+1->PC, End |
| SZE | 0x7002 | if(E!=0)skip-1, PC+1->PC, End |
| HLT | 0x7001 | 1->S(halt), End |
| INP | 0xf800 | input-int->AC, End |
| OUT | 0xf400 | output-AC->int, End |

### Step 5: Save

In **Execute → Options** the program counter is set to PC, and the machine is saved with **File → Save machine as**.

**Result:** A machine based on the Basic Computer architecture was created in CPU Sim and saved.

---

## Practical 2: Create the fetch routine of the instruction cycle

**Aim:** Create the fetch (and decode) routine of the instruction cycle.

Every instruction starts with the same fetch routine. It is set in **Modify → Fetch Sequence**.

| Order | Microinstruction | Meaning |
|-------|------------------|---------|
| 1 | PC->AR | copy the address of the next instruction into AR |
| 2 | M[AR]->IR | read that instruction from memory into IR |
| 3 | PC+1->PC | move PC on to the next instruction |
| 4 | IR(0-11)->AR | copy the address part of the instruction into AR |
| 5 | decode-IR | find out which instruction it is and start its execute sequence |

**Fetching the first instruction of Practical 3 (INP, F800 hex)**

| Micro-step | Microinstruction | AR | PC | IR |
|------------|------------------|----|----|----|
| start | none | 0 | 0 | 0 |
| 1 | PC->AR | 0 | 0 | 0 |
| 2 | M[AR]->IR | 0 | 0 | 63488 (F800) |
| 3 | PC+1->PC | 0 | 1 | 63488 |
| 4 | IR(0-11)->AR | 2048 (800) | 1 | 63488 |
| 5 | decode-IR | 2048 | 1 | 63488, so INP is selected |

**Result:** The fetch routine PC->AR, M[AR]->IR, PC+1->PC, IR(0-11)->AR, decode-IR was created and checked by stepping through the first instruction of a program.

---

## Practical 3: ADD operation on two user-entered numbers

**Aim:** Read two numbers from the user, add them and display the sum.

**Program:** `programs/P03_ADD.a`

```asm
        INP
        STA A
        INP
        ADD A
        OUT
        HLT
A:      .data 1 0
SUM:    .data 1 0
```

| Line | What it means |
|------|---------------|
| `INP` | Asks you to type a number. It goes into AC. |
| `STA A` | Copies the number in AC into the memory word `A`, so it is not lost. |
| `INP` | Asks for the second number. It goes into AC and replaces the old value. |
| `ADD A` | Adds the saved number in `A` to the number in AC. The answer stays in AC. |
| `OUT` | Shows the number in AC on the console. |
| `HLT` | Stops the program. |
| `A: .data 1 0` | Reserves one memory word, names it `A` and puts 0 in it. |
| `SUM: .data 1 0` | Reserves another memory word named `SUM`, also 0. This program does not use it. |

**Machine code after Assemble & load**

| Address | Code (hex) | Instruction |
|---------|-----------|-------------|
| 0 | F800 | INP |
| 1 | 3006 | STA A |
| 2 | F800 | INP |
| 3 | 1006 | ADD A |
| 4 | F400 | OUT |
| 5 | 7001 | HLT |
| 6 | 0000 | A |
| 7 | 0000 | SUM |

**Output:** inputs `5` and `3`, output `8`.

![Practical 3 output](screenshots/P03_output.png)

| Register | AC | AR | DR | E | IR | PC | S |
|----------|----|----|----|---|----|----|---|
| Final value | 8 | 1 | 5 | 0 | 28673 | 6 | -1 (halted) |

DR is 5 because `ADD A` read the first number from memory into DR.

**Result:** The program adds two user-entered numbers: 5 + 3 = **8**.

---

## Practical 4: SUBTRACT operation on two user-entered numbers

**Aim:** Read two numbers A and B and display A - B.

The Basic Computer has no subtract instruction, so it adds the two's complement of B:

```
A - B = A + (B' + 1)
```

`CMA` flips every bit of B (that is B'), and `INC` adds 1, which gives -B.

**Program:** `programs/P04_SUBTRACT.a`

```asm
        INP
        STA A
        INP
        CMA
        INC
        ADD A
        STA DIFF
        OUT
        HLT
A:      .data 1 0
DIFF:   .data 1 0
```

| Line | What it means |
|------|---------------|
| `INP` | Asks for the first number (A). It goes into AC. |
| `STA A` | Saves A in the memory word `A`. |
| `INP` | Asks for the second number (B). It goes into AC. |
| `CMA` | Flips every bit of AC, so AC now holds B'. |
| `INC` | Adds 1 to AC, so AC now holds -B. |
| `ADD A` | Adds A to AC, so AC holds A + (-B), which is A - B. |
| `STA DIFF` | Saves the result in the memory word `DIFF`. |
| `OUT` | Shows AC on the console. |
| `HLT` | Stops the program. |
| `A: .data 1 0` | Reserves a memory word named `A`, value 0. |
| `DIFF: .data 1 0` | Reserves a memory word named `DIFF`, value 0. |

**Machine code after Assemble & load**

| Address | Code (hex) | Instruction |
|---------|-----------|-------------|
| 0 | F800 | INP |
| 1 | 3009 | STA A |
| 2 | F800 | INP |
| 3 | 7200 | CMA |
| 4 | 7020 | INC |
| 5 | 1009 | ADD A |
| 6 | 300A | STA DIFF |
| 7 | F400 | OUT |
| 8 | 7001 | HLT |
| 9 | 0000 | A |
| 10 | 0000 | DIFF |

**Output:** inputs `9` and `3`, output `6`.

![Practical 4 output](screenshots/P04_output.png)

| Register | AC | AR | DR | E | IR | PC | S |
|----------|----|----|----|---|----|----|---|
| Final value | 6 | 1 | 9 | -1 (E = 1) | 28673 | 9 | -1 (halted) |

E = 1 means the addition had a carry out, which happens when A is greater than or equal to B.

**Result:** The program subtracts two numbers using the two's complement: 9 - 3 = **6**.

---

## Practical 5: Logical operations AND, OR, NOT, XOR, NOR, NAND

**Aim:** Perform AND, OR, NOT, XOR, NOR and NAND on two user-entered numbers.

The Basic Computer has only two logic instructions: `AND` and `CMA` (NOT). AND and NOT together are enough to build everything else:

| Operation | Built as |
|-----------|----------|
| A AND B | A . B |
| A OR B | (A' . B')' (De Morgan's law) |
| NOT A | A' |
| A XOR B | (A + B) . (A . B)' |
| A NOR B | (A + B)' |
| A NAND B | (A . B)' |

**Program:** `programs/P05_LOGIC.a`

```asm
        INP
        STA A
        INP
        STA B
; ---------------- AND ----------------
        LDA A
        AND B
        STA RAND
        OUT
; ---------------- OR ----------------
        LDA B
        CMA
        STA NB
        LDA A
        CMA
        STA NA
        AND NB
        CMA
        STA ROR
        OUT
; ---------------- NOT A, NOT B ----------------
        LDA NA
        OUT
        LDA NB
        OUT
; ---------------- XOR ----------------
        LDA RAND
        CMA
        STA NRAND
        LDA ROR
        AND NRAND
        STA RXOR
        OUT
; ---------------- NOR ----------------
        LDA ROR
        CMA
        STA RNOR
        OUT
; ---------------- NAND ----------------
        LDA RAND
        CMA
        STA RNAND
        OUT
        HLT

A:      .data 1 0
B:      .data 1 0
NA:     .data 1 0
NB:     .data 1 0
RAND:   .data 1 0
ROR:    .data 1 0
RXOR:   .data 1 0
RNOR:   .data 1 0
NRAND:  .data 1 0
RNAND:  .data 1 0
```

The seven outputs appear in this order: AND, OR, NOT A, NOT B, XOR, NOR, NAND.

**Reading the numbers**

| Line | What it means |
|------|---------------|
| `INP`, `STA A` | Ask for the first number and save it in `A`. |
| `INP`, `STA B` | Ask for the second number and save it in `B`. |

**AND section**

| Line | What it means |
|------|---------------|
| `LDA A` | Copies `A` into AC. |
| `AND B` | AC becomes AC AND `B`, which is A AND B. |
| `STA RAND` | Saves the result in `RAND`. |
| `OUT` | Shows the result (output 1). |

**OR section**

| Line | What it means |
|------|---------------|
| `LDA B`, `CMA`, `STA NB` | Take B, flip every bit, and save it as `NB` (NOT B). |
| `LDA A`, `CMA`, `STA NA` | Take A, flip every bit, and save it as `NA` (NOT A). |
| `AND NB` | AC becomes NOT A AND NOT B. |
| `CMA` | Flips every bit of AC, which gives A OR B. |
| `STA ROR` | Saves the result in `ROR`. |
| `OUT` | Shows the result (output 2). |

**NOT section**

| Line | What it means |
|------|---------------|
| `LDA NA`, `OUT` | Copy NOT A into AC and show it (output 3). |
| `LDA NB`, `OUT` | Copy NOT B into AC and show it (output 4). |

**XOR section**

| Line | What it means |
|------|---------------|
| `LDA RAND`, `CMA` | Take A AND B and flip it, which gives A NAND B. |
| `STA NRAND` | Saves that in `NRAND`. |
| `LDA ROR` | Copies A OR B into AC. |
| `AND NRAND` | AC becomes (A OR B) AND (A NAND B), which is A XOR B. |
| `STA RXOR` | Saves the result in `RXOR`. |
| `OUT` | Shows the result (output 5). |

**NOR section**

| Line | What it means |
|------|---------------|
| `LDA ROR` | Copies A OR B into AC. |
| `CMA` | Flips every bit, which gives A NOR B. |
| `STA RNOR` | Saves the result in `RNOR`. |
| `OUT` | Shows the result (output 6). |

**NAND section**

| Line | What it means |
|------|---------------|
| `LDA RAND` | Copies A AND B into AC. |
| `CMA` | Flips every bit, which gives A NAND B. |
| `STA RNAND` | Saves the result in `RNAND`. |
| `OUT` | Shows the result (output 7). |
| `HLT` | Stops the program. |

**Data lines:** each `.data 1 0` line reserves one memory word with the value 0. The names are `A`, `B`, `NA`, `NB`, `RAND`, `ROR`, `RXOR`, `RNOR`, `NRAND` and `RNAND`.

**Output:** inputs `1` and `0`.

![Practical 5 output](screenshots/P05_output.png)

| Operation | Result (decimal) | Result (hex) |
|-----------|------------------|--------------|
| A AND B | 0 | 0000 |
| A OR B | 1 | 0001 |
| NOT A | -2 | FFFE |
| NOT B | -1 | FFFF |
| A XOR B | 1 | 0001 |
| A NOR B | -2 | FFFE |
| A NAND B | -1 | FFFF |

NOT gives negative numbers because flipping all 16 bits of a small positive number sets the sign bit.

| Register | AC | AR | DR | E | IR | PC | S |
|----------|----|----|----|---|----|----|---|
| Final value | -1 | 1 | 0 | 0 | 28673 | 38 | -1 (halted) |

**Result:** All six logical operations were done using only AND and CMA. For A = 1 and B = 0 the outputs are AND = 0, OR = 1, NOT A = -2, NOT B = -1, XOR = 1, NOR = -2, NAND = -1.

---

## Practical 6: Memory-reference instructions ADD, LDA, STA, BUN, ISZ

**Aim:** Write a program that uses the memory-reference instructions ADD, LDA, STA, BUN and ISZ.

The program multiplies X = 5 by N = 3 using repeated addition. `CTR` starts at -3, and `ISZ` adds 1 to it on every pass. When `CTR` reaches 0, `ISZ` skips the `BUN` and the loop ends.

**Program:** `programs/P06_MEMORY_REFERENCE.a`

```asm
LOOP:   LDA PROD
        ADD X
        STA PROD
        ISZ CTR
        BUN LOOP
        LDA PROD
        HLT
X:      .data 1 5
CTR:    .data 1 -3
PROD:   .data 1 0
```

| Line | What it means |
|------|---------------|
| `LOOP: LDA PROD` | `LOOP:` is a label that marks this line. Copies the running product `PROD` into AC. |
| `ADD X` | Adds `X` (5) to AC. |
| `STA PROD` | Saves AC back into `PROD`. |
| `ISZ CTR` | Adds 1 to `CTR`. If `CTR` becomes 0, the next line is skipped. |
| `BUN LOOP` | Jumps back to the line labelled `LOOP`. |
| `LDA PROD` | Copies the final product into AC. |
| `HLT` | Stops the program. |
| `X: .data 1 5` | Reserves a memory word named `X` holding 5. |
| `CTR: .data 1 -3` | Reserves a memory word named `CTR` holding -3 (the loop counter). |
| `PROD: .data 1 0` | Reserves a memory word named `PROD` holding 0. |

**Machine code after Assemble & load**

| Address | Code (hex) | Instruction |
|---------|-----------|-------------|
| 0 | 2009 | LDA PROD |
| 1 | 1007 | ADD X |
| 2 | 3009 | STA PROD |
| 3 | 6008 | ISZ CTR |
| 4 | 4000 | BUN LOOP |
| 5 | 2009 | LDA PROD |
| 6 | 7001 | HLT |
| 7 | 0005 | X |
| 8 | FFFD | CTR |
| 9 | 0000 | PROD |

**How the loop runs**

| Pass | PROD after STA | CTR after ISZ | What happens next |
|------|----------------|---------------|-------------------|
| 1 | 5 | -2 | not zero, so BUN LOOP jumps back |
| 2 | 10 | -1 | not zero, so BUN LOOP jumps back |
| 3 | 15 | 0 | zero, so BUN LOOP is skipped and the program goes on to the last LDA |

**Output:** this program takes no input.

![Practical 6 output](scre
