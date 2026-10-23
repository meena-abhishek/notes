functional specification - 
describes what the circuit must do. describes inputs, outputs , relationship between them, operations etc.

timing specification -
describes when signals must change and how quickly the circuit must respond. setup/hold time, propogation delay, clock freq, clock-to-q delay, etc.
eg. a flip-flop might require data to be stable for 2ns before clock edge and 1ns after it, this is a timing specification.
because transistors take some time to determine whether the input is 0 or 1.

*microarchitecture* is the specific arrangement of registers, alus, memories, FSM and other logic building blocks needed to implement an architecture.

architecture is the internal organization of a circuit that implements its functional and timing specification.

a computer architecture is defined by its instruction set and architectural state.
based on the current architectural state, the processor executes a particular instruction with a particular set of data to produce a new architectural state.

microarchitecture contain non-architectural state to either simply logic or improve performance.

R-type (register-type) instructions uses three registers as operands: two as sources and one as destination.

In RISC-V, since many instructions share the same **opcode**, the ISA uses extra fields — `funct3` and `funct7` — to distinguish between different operations that would otherwise look identical at the opcode level. Think of the opcode as saying "this is an R-type instruction" and the funct fields as saying "...and specifically, it's an `add`."

*design process*

we divide our microarchitecture into two interacting parts:
- data path
- control unit

the control unit receives the current instruction from the datapath and tells the datapath how to execute the instruction.
specifically, it produces multiplexer select, register enable and memory write signals.

start designing with hardware that containing state elements:
- program counter, registers, instruction and data memories.

the *program counter* (PC) points to the current instruction. its input, PCNext, indicates the address of the next instruction.

![[riscv_state_element.png]]

the instruction memory, register file, and data memory are all read *combinationally*. in other words, if the address changes, then the new data appears at RD after some propagation delay; no clock is involved.
the clock controls writing only. these memories are written only on rising edge of the clock. in this fashion, the state of the system is changed only at the clock edge.

in most real processors, the instruction memory must be writable so that the operating system (OS) can load a new program into memory. The multicycle microarchitecture described in Section 7.4 is more realistic in that it uses a single memory that contains both instructions and data and that can be both read and written.

### Why keeping them separate is the right call either way

- Matches real hardware's split L1 structure (correctness win, not just simplicity).
- Avoids the structural hazard of fetch and data-access needing the same memory port in the same cycle (relevant even in multi-cycle, since some multi-cycle designs _do_ fetch and access memory in different cycles specifically to allow sharing — but keeping them separate is still more realistic and scales better toward a pipelined design later).
- If you later extend this into a pipelined CPU (5-stage), you'll be _forced_ to split them anyway, since IF and MEM stages can be active in the same cycle for different instructions — sharing one memory would create a structural hazard every single cycle.

32 entries × 32 bits is tiny. Combinational read here is fine and will map to distributed RAM (LUTRAM) — real RISC-V cores actually do exactly this for register files (small, low-latency, deliberately not BRAM).


the multicycle microarchitecture reduces the
hardware cost by reusing expensive hardware blocks, such as adders
and memories. For example, the adder may be used on different cycles
for several purposes while carrying out a single instruction.

Same physical adder, different job each cycle — controlled by muxes selecting its inputs based on the current FSM state. This is the whole point of having a control FSM in the first place: it's not just sequencing instruction phases, it's **steering shared hardware to different roles each cycle**.

pipelined microarchitecture applies pipelining to the single-cycle microarchitecture.

![[execution_time_eq.png]]

The cycles per instruction (CPI) is the number of clock cycles required to execute an average instruction.
It is the reciprocal of the throughput (instructions per cycle, or IPC).

the clock period is determined by the critical path through the logic in the processor.

the challenge of the microarchitect is to choose the desing that minimizes execution time while satisfying contraints on cost and/or power consumption.

the control unit contains combinational logic that generates the appropriate control signals based on the current instruction.

*single-cycle processor*

the program counter contains the address of the instruction to execute.

RISC-V is called a 32-bit architecture because it operates on 32-bit data.

Design Principle 1: Regularity supports simplicity.
Design Principle 2: Make the common case fast.
Design Principle 3: Smaller is faster.
Design Principle 4: Good design demands good compromises.

In addition to register operations, RISC-V instructions can use constant or immediate operands. These constants are called immediates because their values are immediately available from the instruction and do not require a register or memory access.

`lui` - load upper immediate, stores msb 20bits and 0 on lsb 12bits.

when using lui, we first create a 20bit word with last 12bits 0s, then we sign extend another 12bit immediate value to 32bits, then we add last 12bits of both numbers and first 20bits of both. if 12bit immediate was negative then we should be adding all 1s to upper immediate, so we need to add 1 coz all 1s is -1, in 2s complement. my doubt is when 12bit immd is negative and after adding one to upper immediate, so we add 12bit to upper 12bit as it is? like if its also 11111 , then we add these 1111.. to upper immediate?


Using a series of two instructions—one of the “multiply high” instructions followed by the mul instruction—will place the entire 64-bit result of the 32-bit multiplication in the two registers designated by the user.

*branching*
Branch instructions modify the flow of the program so that the processor can fetch instructions that are not in sequential order in memory. They modify the PC to skip over sections of code or to repeat previous code. Conditional branch instructions per- form a test and branch only if the test is TRUE. Unconditional branch instructions, called jumps, always branch.

![[if_else_riscv_ass.png|700]]



the main function calls the simple function by executing `jal` simple, which performs two tasks: it jumps to the target instruction located at simple (0x0000051C) and it stores the return address, the address of the instruction after jal (in this case, 0x00000304) in the return address register (ra). The programmer can specify which register gets written with the return address, but the default is ra. So, jal simple is equivalent to `jal ra, simple` and is the preferred style. The simple function returns immediately by executing the instruction jr ra, which jumps to the instruction address held in ra.

j and jr are pseudoinstructions.
They are not part of the
instruction set but are
convenient for programmers
to use. The RISC-V assembler
replaces them with actual
RISC-V instructions. The
assembler replaces j target
with jal zero, target,
which jumps and discards the
return address by writing it
to the zero register; and the
assembler replaces jr ra with
jalr zero, ra, 0.
The jump and link
register instruction (jalr)
is like jal, but it takes the
destination address from a
register, optionally added to a
12-bit signed immediate. For
example, jalr ra, s1, 0x4C
jumps to address s1 + 0x4C
and puts PC+4 in ra.

*instructions*

R-type (register) instructions, such as add s0, s1, s2, operate on
three registers. I-type (immediate) instructions, such as addi s3, s4, 42,
and S/B-type (store/branch) instructions, such as sw a0, 4(sp) or
beq a0,a1,L1, operate on two registers and a 12- or 13-bit signed
immediate. U/J-type (upper immediate/jump) instructions, such as
jal ra, factorial, operate on one register and a 20- or 21-bit immediate

![[r-type.png|500]]

![[i-type.png|500]]


Latch inference is a concern **only for combinational logic** (`always @(*)` blocks). Your write-guard code is inside a **clocked** block:
This is `always @(posedge clk)`, not `always @(*)`. In a clocked block, an `if` without an `else` doesn't create a latch — it creates exactly what you want: **a register that holds its value when not written**. This is normal, correct, and extremely common — it's literally how every conditional register update in synchronous design works (e.g., your PC register: `if (pc_write) pc <= pc_next;` with no else — same pattern, same correctness).

Verilog's `reg`/`wire` types are unsigned by default unless declared with the `signed` keyword — `$signed()` is a way to temporarily treat a value as signed _without_ having to declare the whole signal as `signed` everywhere it's used.

`>>`Logical right shiftAlways fills with **0**, regardless of sign`>>>`Arithmetic right shiftFills with the **sign bit** (MSB) — but _only_ if the operand is `signed`

#synthesis_directive
**`(* rom_style = "block" *)`**  
This is a synthesis attribute (directive) that tells the synthesis tool (e.g., Xilinx/Vivado) how to implement the memory that follows. `rom_style = "block"` instructs the tool to map this memory onto dedicated **Block RAM (BRAM)** resources on the FPGA, rather than implementing it using distributed logic (LUTs). This is useful for larger memories where BRAM is more efficient than LUT-based storage.

Good follow-up — these are two of the fundamental building blocks inside an FPGA. Here's the difference:

## LUT (Look-Up Table)

A LUT is the basic building block used to implement **logic functions** in an FPGA.

- Think of it as a tiny truth table stored in memory. For example, a 6-input LUT (common in modern FPGAs) has 2^6 = 64 possible input combinations, and for each combination, it stores a pre-computed 1-bit output.
- Instead of building logic gates (AND, OR, XOR, etc.) out of transistors like an ASIC would, an FPGA implements _any_ logic function by just storing the correct output values in this little memory table.
- LUTs are usually paired with a flip-flop (register) to form a **Slice** or **Logic Element (LE)** — the basic repeating unit of FPGA fabric.
- When you write ordinary combinational logic (`assign`, `always @(*)`) or use `reg` for small amounts of storage, synthesis tools typically implement it using LUTs (and flip-flops for the registers).

**Use case:** general logic, small state machines, small memories/registers, control logic.

## BRAM (Block RAM)

BRAM is a **dedicated, hardened memory block** built into the FPGA fabric — separate from the LUT-based logic.

- It's a fixed-size chunk of true RAM (e.g., 18Kb or 36Kb per block on Xilinx devices), physically laid out on the die as its own resource, much like a small hardware RAM chip embedded in the FPGA.
- Because it's a dedicated hardware structure, it's far more efficient (in area and power) at storing larger amounts of data than building an equivalent-sized memory out of LUTs would be.
- Each FPGA has a limited number of BRAM blocks available (a fixed resource budget), separate from the LUT count.

**Use case:** larger memories — FIFOs, buffers, lookup tables with lots of entries, your `stimulus_mem` array, caches, etc.

## Why it matters for your code

Your `stimulus_mem` array is 17 × 139 bits ≈ 2,363 bits total.

- If synthesized using **LUTs** ("distributed RAM"), each bit of storage eats into your general logic fabric — fine for small memories, but wasteful and fabric-consuming for anything sizeable.
- If synthesized using **BRAM** (which the `rom_style = "block"` attribute forces), it instead gets mapped onto one of those dedicated memory blocks, freeing up LUTs for actual logic and usually being faster/denser for storage purposes.

![[lut-structure.png|400]]

### Practical takeaway

This is exactly why, in real hardware design, **reducing the number of inputs to critical logic paths matters for timing** — a 20-input `assign` statement will get chopped into a tree of LUTs several levels deep, and that adds up to real propagation delay. Tools report this in **critical path / logic level analysis** during timing closure, and designers sometimes manually restructure wide logic (e.g., using a case statement, breaking into stages, or pipelining) specifically to reduce LUT depth.

This is the key insight: **once configured, each SRAM cell just sits there holding a static 0 or 1, continuously driving its output pin.** It behaves like a tiny latch with a fixed value — not like a cell that needs to be "read" via a sensing mechanism.

--- theory ---

Instruction cache (I-cache) and data cache (D-cache) exist at the **L1 level** in modern processors.

**L1 Cache (split)**

- L1 is divided into two separate caches: L1i (instruction cache) and L1d (data cache)
- Each is usually small and fast, commonly 32KB–64KB per core (sizes vary by CPU)
- They're kept separate at this level for a practical reason: the CPU needs to fetch an instruction _and_ read/write data in the same clock cycle. Having two separate caches with independent access ports lets both happen simultaneously without contention. This is sometimes called a "Harvard architecture" style split, even though main memory itself is unified (von Neumann style).

Great follow-up. The key insight: **it's not that L2 inspects the data and figures out "this is code" vs "this is data."** Instead, the _request itself_ already carries that information, from the moment it's generated.

Here's the flow:

**1. The request originates from a specific place**

- The **fetch unit** generates instruction-fetch requests (based on the program counter)
- The **load/store unit** generates data requests (based on load/store instructions executing)

These are two structurally separate parts of the pipeline, each with its own connection down toward L2.

**2. On an L1 miss, the request is tagged with its origin**  
When L1i misses, it sends a fill request to L2 that's tracked in a **Miss Status Handling Register (MSHR)** — this entry records _which cache and which specific line_ needs to be filled once the data comes back. Same for an L1d miss — its own MSHR entry, its own return path.

**3. L2 doesn't classify content — it just answers the request**  
L2 is unified and stores lines without caring what they "are." When it responds, it just sends the requested cache line back along the same path/tag that made the request. The MSHR entry says "this response goes to L1i, line X" or "this response goes to L1d, line Y." No content inspection needed.

**A subtlety this reveals:** since main memory is unified (von Neumann), the exact same address _could_ end up cached in both L1i and L1d at different times — nothing at the memory level marks a byte as "code" vs "data." This actually causes a real problem: **self-modifying code or JIT-compiled code**, where you write to an address as data (D-cache) but later try to execute it as an instruction (I-cache). Since the two caches are independent, the I-cache might hold a stale copy. CPUs handle this with explicit I-cache invalidation mechanisms (or software has to flush/sync manually) when this pattern happens.

Want me to dig into how that self-modifying code / instruction-cache-coherency problem is actually solved in hardware?

`lw x6, -4(x9)`

i-type instruction, base register is in rs1 field (Instr 19:15)
these bits connect to A1 address input of the register file.

the offset is stored in 12-bit immediate field of the instruction (Instr 31:20)
this value is signed to it is signed extended.
ImmExt 31:12 = Instr 31, ImmExt 11:0 = Instr 31:20

ALUControl 2:0 specified which operation to perform to SrcA & ScrB.

this addr is passed to data memory and data is put onto readdata bus which is connected to WD3 port on register file, destination register address Instr 11:7 is passed to A3 port of register file, a control signal RegWrite is connected to WE3 port on register file.
the write takes place on the rising edge of the clock cycle at the end of the cycle.

the new address is written into the program counter on the next rising edge of the clock.

reads are asynchronous, only writes are clocked.

Good question — this is a classic point of confusion in single-cycle datapath design. The short answer: **PC, register file, and dmem don't update at different times — they all use the _same_ clock edge, simultaneously.** The confusion usually comes from thinking there's a sequence like "PC changes, then a moment later regfile/dmem latch." That's not what happens.

Let me break down why it works.

## The key trick: reads are asynchronous, only writes are clocked

- **Instruction memory read** — combinational (not clocked). Feed in an address, the instruction appears after some propagation delay. No edge needed.
- **Register file read** — also combinational/asynchronous. Feed in register numbers, the values appear on the read ports continuously.
- **Register file write** and **data memory write** — these ARE clocked (edge-triggered). They only commit on a clock edge.
- **PC** — clocked. Only changes on a clock edge.

So during a cycle, nothing is "waiting" for PC to update — the current PC value just sits there, continuously driving instruction memory, and everything downstream (decode, register read, ALU, memory access) is pure combinational logic settling into steady values.

## What actually happens across one cycle

Think of cycle N like this:

1. **At the start of cycle N** (just after a clock edge), PC holds value `P_N` — set by the _previous_ edge.
2. This value ripples through combinational logic the whole cycle: instruction fetch → register read → ALU → data memory access → write-back mux. All of this just settles, like water finding its level — no clock involved in any of these steps individually.
3. In parallel, separate combinational "next PC" logic computes `P_(N+1)` from `P_N` and the branch/jump control signals. This is also just settling during the cycle.
4. **At the _next_ clock edge** (end of cycle N / start of cycle N+1), three things latch **at the exact same instant**:
    - PC captures `P_(N+1)`
    - Register file write port commits the result computed _during cycle N_ (if the instruction writes a register)
    - Data memory write port commits the store data computed _during cycle N_ (if it's a store)

So the register file/dmem writes use data that was fully computed using the _old_ PC value (`P_N`) — the new PC value only becomes "visible" starting next cycle, to fetch the next instruction. There's no leftover race between "PC changing" and "regfile/dmem latching" because they're not sequenced relative to each other at all — they're all sequenced relative to the _same_ clock edge, using values prepared beforehand.

## Why the clock period being long matters

This is exactly why single-cycle CPUs need a long clock period: it must be at least as long as the **worst-case combinational path** through the entire datapath — PC-to-Q delay + instruction memory access + register read + ALU + data memory access + mux delays + setup time for the write ports. Typically this worst case is a `load` instruction, since it touches every stage.

As long as the period is sized this way, by the time the next clock edge arrives, every combinational signal (including the write-back data and the next-PC value) has already settled to its correct, stable value. Nothing is "invalid" — it's just slow.


`sw x6, 8(x9)`

S-type instruction,
12-bit signed immediate is stored at Instr 31:25, 11:7
the extend unit receives all the bits of Instr 31:7
ImmSrc decides, which bits to choose , lw (31:20) or sw(31:25, 11:7).
source reg, rs2 Instr 24:20 connected to A2.

this RD2 is connected to WD of memory, MemWrite = WE = 1
RD1 + ImmExt = ALUResult = A.
ReadData is ignored because RegWrite = 0.

"Latching" means **capturing and holding a value at a specific moment**, so it stays fixed even after the input changes.

Think of it like a photograph vs. a live video feed:

- **Combinational logic (no latching)** — like a live video feed. The output constantly tracks the input. If the input changes, the output changes too, continuously, with just a small propagation delay.
- **A latch/flip-flop (latching)** — like taking a photo. At one specific instant (the clock edge), it "snaps" the current value and holds it steady — even if the input keeps changing right after. The stored value only updates again at the _next_ clock edge.

**Concretely, in hardware:**  
A flip-flop (the basic building block of registers like the PC or register file) has an input D, an output Q, and a clock. Right when the clock edge arrives:

- It looks at whatever value is on D _at that instant_
- It copies that value into Q
- Q holds that value steady until the next clock edge, ignoring any further changes on D in between

**Why this matters in your single-cycle question:**  
When I said the register file or dmem "latches" a value on the clock edge, I meant: the combinational logic feeds them a computed value all cycle long (this value may still be settling/changing as different inputs arrive), but at the exact clock edge, the flip-flops inside the register file/dmem freeze whatever value is present at that moment into storage. That's why it matters that the value has already _stabilized_ by the time the edge hits — if it hadn't, you'd latch garbage or a half-settled value.

So "latching" = the act of a clocked storage element grabbing and holding a value at a clock edge, as opposed to combinational logic which just continuously reflects its input.

*R-Type Instructions*

`slt` - set less than, rd = rs < rt ? 1 : 0

ALUSrc, select signal to mux selecting between RD2 (0) & ImmExt (1).

if Result is written back to register file, then if ResultSrc = 0 then ALUResult is written else ReadData from memory.

Observe that our hardware computes all the possible answers needed by different instructions (e.g., ALUResult and ReadData) and then uses a multiplexer to choose the appropriate one based on the instruction. This is an important design strategy. Throughout the rest of this chapter, we will add multiplexers to choose the desired answer. One of the major differences between software and hardware is that software operates sequentially, so we can compute just the answer we need. Hardware operates in parallel; therefore, we often compute all the possible answers and then pick the one we need. For example, while executing an R-type instruction with the ALU, the memory still receives an address and reads data from this address even though we don’t care what that data might be.

Let me correct a few mix-ups in your example first (I-type doesn't have rs2, and opcode is bits 6:0 — 7 bits, not "6-7") — then here's the full set.

All RISC-V instructions are 32 bits, and opcode is **always bits [6:0]**, funct3 is **always bits [14:12]**, rd is **always bits [11:7]**, rs1 is **always bits [19:15]**, rs2 is **always bits [24:20]** — the fields stay in fixed positions across formats so decode hardware can start reading rs1/rs2 before it even knows the format. What changes is what's _not_ used, and where immediate bits get packed.

## R-type (register-register, e.g. `add`, `sub`, `and`)

|31:25|24:20|19:15|14:12|11:7|6:0|
|---|---|---|---|---|---|
|funct7|rs2|rs1|funct3|rd|opcode|

## I-type (immediate, e.g. `addi`, loads `lw`, `jalr`)

|31:20|19:15|14:12|11:7|6:0|
|---|---|---|---|---|
|imm[11:0]|rs1|funct3|rd|opcode|

- **Note:** no rs2 field — the second operand is the immediate itself.
- For loads specifically (`lw`, `lb`, `lh`, etc.), rs1 is the base address register, imm[11:0] is the offset, and rd is where the loaded value goes. Same layout as any other I-type.

## S-type (store, e.g. `sw`, `sb`, `sh`)

|31:25|24:20|19:15|14:12|11:7|6:0|
|---|---|---|---|---|---|
|imm[11:5]|rs2|rs1|funct3|imm[4:0]|opcode|

- Here rs2 is the value **being stored**, rs1 is the base address.
- Immediate is **split** into two pieces (imm[11:5] and imm[4:0]) so that rs1 and rs2 stay in the same bit positions as R-type — this is the clever part of RISC-V's encoding.
- Notice there's **no rd** — makes sense, a store doesn't write back to a register.

## B-type (branch, e.g. `beq`, `bne`, `blt`)

|31|30:25|24:20|19:15|14:12|11:8|7|6:0|
|---|---|---|---|---|---|---|---|
|imm[12]|imm[10:5]|rs2|rs1|funct3|imm[4:1]|imm[11]|opcode|

- Compares rs1 and rs2, branches if condition (given by funct3) holds.
- Immediate bits are **scrambled/reordered** on purpose — imm[0] is never encoded (it's always 0, since branch targets are 2-byte aligned minimum, and actually implicitly even), and the bits are arranged so the sign bit (imm[12]) stays at position 31 — same place as every other immediate's sign bit, so sign-extension hardware is reused across all formats.
- No rd (branches don't write a register).

## U-type (upper immediate, e.g. `lui`, `auipc`)

|31:12|11:7|6:0|
|---|---|---|
|imm[31:12]|rd|opcode|

- Just loads a 20-bit immediate into the upper bits of rd (lower 12 bits become 0). Used to build large constants — usually paired with an I-type instruction (like `addi`) to fill in the lower 12 bits.

## J-type (jump, e.g. `jal`)

|31|30:21|20|19:12|11:7|6:0|
|---|---|---|---|---|---|
|imm[20]|imm[10:1]|imm[11]|imm[19:12]|rd|opcode|

- rd gets the **return address** (PC+4).
- Immediate is scrambled similarly to B-type, again keeping the sign bit at position 31.

## Quick summary table

|Format|Used by|rs2?|rd?|Immediate|
|---|---|---|---|---|
|R|add, sub, and, or, slt...|Yes|Yes|none|
|I|addi, loads, jalr|No|Yes|12-bit, contiguous|
|S|stores (sw, sb, sh)|Yes|No|12-bit, split|
|B|branches (beq, bne...)|Yes|No|13-bit (imm[0]=0), split & scrambled|
|U|lui, auipc|No|Yes|20-bit, upper bits only|
|J|jal|No|Yes|21-bit (imm[0]=0), scrambled|

Want me to explain _why_ the immediate bits get scrambled the way they do in B-type and J-type (it's a neat trick for hardware simplicity), or how the immediate generator ("imm gen") unit in the datapath reconstructs these into a clean sign-extended 32-bit value?


(SrcA - SrcB == 0 ) = PCSrc == 1 ? PCTarget : PCNext

in case of branch instruction offset from Extender is added to current PC.

![[single_cycle_processor.png]]


**use ALU decoder, main decoder truth tables to code the controller.**

Note that data does flow through the nonhighlighted
paths, but the value of that data is disregarded. For example, the immediate is
extended and a value is read from memory, but these values do not influence the
next state of the system.

`jal` - stores PC+4 in Rd and sets PC to PC+Imm 

![[single_cycle_controller.png|350]]

![[lw_critical_path.png]]

in most implementation technologies, the ALU, memory, and register
file are substantially slower than other combinational blocks. Therefore,
the critical path is through the register file—not through the decoder
(controller), Extend unit, and multiplexer.

## Why the immediate bits are scrambled

The core design principle in RISC-V: **keep the fields that are shared across formats in the exact same bit positions**, so decode hardware doesn't need to know the instruction format before it starts reading fields. This applies to opcode, rd, funct3, rs1, rs2 — and it also applies to _how immediates are built_, as much as possible.

Two specific goals drive the scrambling:

### 1. The sign bit always lives at bit 31

Every format's immediate is sign-extended, and sign-extension logic just needs to grab bit 31 and replicate it upward. So no matter which format the instruction is:

- I-type: imm[11] (the MSB of a 12-bit immediate) sits at instruction bit 31
- S-type: imm[11] also sits at bit 31
- B-type: imm[12] sits at bit 31
- J-type: imm[20] sits at bit 31
- U-type: imm[31] sits at bit 31 naturally (it's just the top bits directly)

This means **one piece of hardware** — "take bit 31, replicate it into all the upper bits of the 32-bit immediate" — works for _every_ format. You don't need format-specific sign-extension logic.

### 2. rs1/rs2 positions are preserved even when an immediate needs those bits

S-type and B-type both need rs2 (to compare against, or to store), so rs1 and rs2 must stay at their usual positions (19:15 and 24:20). But those formats also need to encode an immediate, and the _only_ leftover bits are 31:25 and 11:7. So the immediate literally gets **split into two chunks** and stuffed into whatever bits are left over — it's not scrambled arbitrarily, it's scrambled specifically to avoid colliding with rs1/rs2/rd/funct3/opcode.

Similarly for J-type: rd must stay at 11:7, so the 20-bit immediate has to be packed into bits 31:12 in whatever order fits — and again, the designers chose an order that keeps imm[20] (sign bit) at bit 31, matching every other format.

**The tradeoff:** this makes the _immediate generator hardware_ slightly more complex (it needs some rewiring/muxing to reassemble the bits in the right order) — but in exchange, the _far more critical_ decode paths (register file read ports, ALU control) are simplified and made uniform. This is a deliberate hardware-complexity tradeoff: pay a little cost in one small unit (imm gen) to simplify a path that's used by literally every instruction (register/opcode decode).

## The Immediate Generator (Imm Gen) unit

In the datapath, "Imm Gen" is a combinational block that:

- **Input:** the full 32-bit instruction
- **Output:** a clean, sign-extended 32-bit immediate value

It works like this conceptually:

1. Look at the **opcode** (bits 6:0) to determine which format this instruction is (this is really the only thing needed — opcode alone determines I/S/B/U/J for a given instruction, since opcode encodes the instruction class).
2. Based on the format, **reassemble** the scattered immediate bits from the instruction into their correct logical order. For example, for B-type:
    
    ```
    imm[12]   = instr[31]imm[11]   = instr[7]imm[10:5] = instr[30:25]imm[4:1]  = instr[11:8]imm[0]    = 0            (always, since branches are 2-byte aligned min)
    ```
    
3. **Sign-extend**: replicate instr[31] into all bits above the immediate's own MSB, filling up to a full 32-bit value.
4. Output this reassembled, sign-extended 32-bit immediate to whatever needs it (usually one input of the ALU, via a mux that picks between register value and immediate).

### Why this is just wiring, not real "logic"

A key insight: **Imm Gen does no computation** — no adders, no ALU operations. It's purely bit selection and replication:

- Each output bit of the immediate is wired directly to some specific input bit (or to a copy of instr[31] for sign extension)
- A small mux (controlled by the opcode-derived format signal) selects _which_ wiring pattern to use, since different formats need different rearrangements

So it's fast — just multiplexers and wires, no propagation delay from arithmetic — and it can run in parallel with other decode-stage work (like reading the register file), since it only depends on the instruction bits themselves, not on any register values.

### Where it fits in the single-cycle datapath

```
Instruction ---> [Imm Gen] ---> sign-extended imm ---> ALU input B (via mux)
             |
             +--> [Register File] ---> rs1, rs2 values
             |
             +--> [Control Unit] ---> control signals (including imm-gen format select, ALU op, etc.)
```

All of this happens **combinationally and in parallel** within the same cycle — consistent with everything we discussed earlier about single-cycle timing. Imm Gen's output just needs to be stable and correct by the time it reaches the ALU input mux, well before the clock edge that ends the cycle.

multi-cycle processor

However, ImmExt is a combinational function of Instr and will not change while the current
instruction is being processed, so there is no need to dedicate a register
to hold the constant value.

The main difference is that the instruction is executed in sev-
eral steps. Nonarchitectural registers are inserted to hold the results of
each step. In this way, the memory can be shared for instructions and
data and the ALU can be reused several times, thus reducing hardware
costs. 

varying cycles for different instructions

Unfortunately, this example shows that the multicycle processor is slower
than the single-cycle processor, given the assumptions of CPI and circuit
element delays. The fundamental problem is that even though the slow-
est instruction, lw, was broken into five steps, the multicycle processor
cycle time was not nearly improved fivefold. This is partly because not
all of the steps are exactly the same length and partly because the 90-ps
sequencing overhead of the register clock-to-Q and setup time must now
be paid on every step, not just once for the entire instruction. In general,
engineers have learned that it is difficult to exploit the fact that some
computations are faster than others unless the differences are large.

*Pipelined Processor*

Because each
stage has only one-fifth of the entire logic, the clock frequency is approxi-
mately five times faster. So, ideally, the latency of each instruction is
unchanged, but the throughput is five times better. Microprocessors exe-
cute millions or billions of instructions per second, so throughput is more
important than latency. Pipelining introduces some overhead, so the
throughput will not be as high as we might ideally desire, but pipelining
nevertheless gives such great advantage for so little cost that all modern
high-performance microprocessors are pipelined.

Reading and writing the memory and register file and using the ALU
typically constitute the biggest delays in the processor. We choose five
pipeline stages so that each stage involves exactly one of these slow steps.
Specifically, we call the five stages Fetch, Decode, Execute, Memory, and
Writeback. They are similar to the five steps that the multicycle processor
used to perform lw. In the Fetch stage, the processor reads the instruc-
tion from instruction memory. In the Decode stage, the processor reads
the source operands from the register file and decodes the instruction to
produce the control signals. In the Execute stage, the processor performs
a computation with the ALU. In the Memory stage, the processor reads
or writes data memory, if applicable. Finally, in the Writeback stage, the
processor writes the result to the register file, if applicable.

In the pipelined processor, the register file is used twice in every cycle: it is written in the first part of a cycle and read in the second part, as suggested by the shading. This way, data can be
written by one instruction and read by another within a single cycle.

A central challenge in pipelined systems is handling hazards that
occur when one instruction’s result is needed by a subsequent instruction
before the former instruction has completed.

The error is in the register file write logic, which should operate in
the Writeback stage. The data value comes from ResultW, a Writeback
stage signal. But the destination register comes from RdD (InstrD11:7),
which is a Decode stage signal. In the pipeline diagram of Figure 7.48,
during cycle 5, the result of the lw instruction would be incorrectly writ-
ten to s5 rather than s2.

![[RAW_hazard.png]]

insert nops before the subsequent instruction. 
increases runtime.
reorder independent instructions if available, inplace of nop.

On closer inspection, observe from Figure 7.52 that the sum from
the add instruction is computed by the ALU in cycle 3 and is not strictly
needed by the and instruction until the ALU uses it in cycle 4. In princi-
ple, we should be able to forward the result from one instruction to the
next to resolve the RAW hazard without waiting for the result to appear
in the register file and without slowing down the pipeline. In other situ-
ations explored later in this section, we may have to stall the pipeline to
give time for a result to be produced before the subsequent instruction
uses the result.


Hazards are classified as data hazards or control hazards. A data
hazard occurs when an instruction tries to read a register that has not
yet been written back by a previous instruction. A control hazard occurs
when the decision of what instruction to fetch next has not been made
by the time the fetch takes place.

Forwarding is necessary when an instruction in the Execute stage
has a source register matching the destination register of an instruction
in the Memory or Writeback stage.

Hazard Unit should forward from a stage if that stage will write a des-
tination register and the destination register matches the source register.
However, x0 is hardwired to 0 and should never be forwarded. If both
the Memory and Writeback stages contain matching destination reg-
isters, then the Memory stage should have priority because it contains
the more recently executed instruction.

![[forwarding_equation.png]]

Forwarding is sufficient to solve RAW data hazards when the result
is computed in the Execute stage of an instruction because its result
can then be forwarded to the Execute stage of the next instruction.
Unfortunately, the lw instruction does not finish reading data until
the end of the Memory stage, so its result cannot be forwarded to the
Execute stage of the next instruction. We say that the lw instruction
has a two-cycle latency because a dependent instruction cannot use its
result until two cycles later. Figure 7.56 shows this problem. The lw
instruction receives data from memory at the end of cycle 4, but the
and instruction needs that data (the value in s7) as a source operand
at the beginning of cycle 4. There is no way to solve this hazard with
forwarding.

A solution is to stall the pipeline, holding up operation until the data
is available. Figure 7.57 shows stalling the dependent instruction (and)
in the Decode stage. and enters the Decode stage in cycle 3 and stalls
there through cycle 4. The subsequent instruction (or) must remain in the
Fetch stage during both cycles as well because the Decode stage is full.
In cycle 5, the result can be forwarded from the Writeback stage
of lw to the Execute stage of and. Also, in cycle 5, source s7 of the
or instruction is read directly from the register file, with no need for
forwarding

ote that the Execute stage is unused in cycle 4. Likewise, Memory
is unused in cycle 5 and Writeback is unused in cycle 6. This unused
stage propagating through the pipeline is called a bubble, which behaves
like a nop instruction. The bubble is introduced by zeroing out the
Execute stage control signals during a Decode stage stall so that the
bubble performs no action and changes no architectural state.

In summary, stalling a stage is performed by disabling its pipeline
register (i.e., the register to the left of a stage) so that the stage’s inputs
do not change. When a stage is stalled, all previous stages must also be
stalled so that no subsequent instructions are lost. The pipeline register
directly after the stalled stage must be cleared (flushed) to prevent bogus
information from propagating forward. Stalls degrade performance, so
they should be used only when necessary.

![[LW_hazard.png]]

1. A load word is in the Execute stage (indicated by ResultSrcE0 = 1)
and
2. The load’s destination register (RdE) matches Rs1D or Rs2D, the
source operands of the instruction in the Decode stage
Stalls are supported by adding enable inputs (EN) to the Fetch and
Decode pipeline registers and a synchronous reset/clear (CLR) input to
the Execute pipeline register. When a load word (lw) stall occurs, StallD
and StallF are asserted to force the Decode and Fetch stage pipeline reg-
isters to retain their existing values. FlushE is also asserted to clear the
contents of the Execute stage pipeline register, introducing a bubble. 
The Hazard Unit lwStall (load word stall) signal indicates when the pipeline
should be stalled due to a load word dependency. Whenever lwStall is
TRUE, all of the stall and flush signals are asserted. Hence, the logic to
compute the stalls and flushes is:

![[LW_hazard_equation.png]]

control hazards,

the beq instruction presents a control hazard: the pipelined processor
does not know what instruction to fetch next because the branch deci-
sion has not been made by the time the next instruction is fetched.
One mechanism for dealing with this control hazard is to stall the
pipeline until the branch decision is made (i.e., PCSrcE is computed).
Because the decision is made in the Execute stage, the pipeline would
have to be stalled for two cycles at every branch. This would severely
degrade the system performance if branches occur often, which is typi-
cally the case.

An alternative to stalling the pipeline is to predict whether the branch
will be taken and begin executing instructions based on the predic-
tion. Once the branch decision is available, the processor can throw out
the instructions if the prediction was wrong. In the pipeline presented so
far (Figure 7.58), the processor predicts that branches are not taken and
simply continues executing the program in order until PCSrcE is asserted to
select the next PC from PCTargetE instead. If the branch should have been
taken, then the two instructions following the branch must be flushed
(discarded) by clearing the pipeline registers for those instructions. These
wasted instruction cycles are called the branch misprediction penalty.

![[flush_branch_taken.png]]

Finally, we must work out the stall and flush signals to handle
branches and PC writes. When a branch is taken, the subsequent two
instructions must be flushed from the pipeline registers of the Decode
and Execute stages. Thus, we add a synchronous clear input (CLR) to
the Decode pipeline register and add the FlushD output to the Hazard
Unit. (When CLR = 1, the register contents are cleared, that is, become
0.) When a branch is taken (indicated by PCSrcE being 1), FlushD and
FlushE must be asserted to flush the Decode and Execute pipeline regis-
ters. Figure 7.60 shows the enhanced pipelined processor for handling
control hazards. The flushes are now calculated as:

FlushD = PCSrcE
FlushE = lwStall | PCSrcE

![[full_pipeline.png]]

![[hazard_forwarding.png]]

The pipelined processor ideally would have a CPI of 1 because a new
instruction is issued—that is, fetched—every cycle. However, a stall or a
flush wastes 1 to 2 cycles, so the CPI is slightly higher and depends on
the specific program being executed.

