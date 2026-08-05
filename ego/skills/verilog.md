$HDL$ - hardware definition language

a synthesizer translates verilog into a configuration of the FPGA's actual physical building blocks.

use ASIC for a custom job, coz reconfigurable routing of FPGA adds overhead.

sized literal - `<size>'<base><value>`

![[verilog_module.png|500]]

wires are directional.

continous assignments (`assign`) describe connection between things and not copying value.

`always @(*) begin`
- whenever any input to this logic changes, recompute everything inside immediately (no clock, no memory) only combinational logic.
- `@(a or b or c)` sensitivity list

cases, ifs can only be used inside `always` or `initial` blocks

There are three types of assignments in Verilog:

- **Continuous** assignments (assign x = y;). Can only be used when **not** inside a procedure ("always block").
- Procedural **blocking** assignment: (x = y;). Can only be used inside a procedure.
- Procedural **non-blocking** assignment: (x <= y;). Can only be used inside a procedure.

In a **combinational** always block, use **blocking** assignments. In a **clocked** always block, use **non-blocking** assignments

- Non-blocking (`<=`) schedules the update to happen at the end of the time step, so all flip-flops in a design update "simultaneously" based on the _old_ values — this correctly models real hardware, where every flop's D input is sampled at the same clock edge.
- Blocking (`=`) executes immediately in sequence, which can cause simulation to give wrong results if you chain multiple flip-flops (a later flop might see an already-updated value instead of the old one, which wouldn't happen in real hardware).

- `posedge sig` → "the instant sig rises from 0 to 1"
- `negedge sig` → "the instant sig falls from 1 to 0"
- No edge keyword, just `sig` in sensitivity list (e.g. `@(sel)` or `@(*)`) → "any change in value, level-sensitive," used for combinational logic, not edges.

sensitive list: level/ edge

"=" executes code sequentially inside a begin / end, whereas nonblocking "<=" executes in parallel.

*assign* -
An `assign` statement is a _continuous assignment_. 
`assign one = 1`
this isn't a one-time assignment like in software. The `assign` statement creates a permanent connection — think of it as literally wiring `one` to a constant 1V rail in hardware. If anything on the right-hand side ever changes, the left-hand side updates instantly and continuously, with no clock involved.

`assign out = a ~^ b` - behavioral expression
`xor (out,a,b)` - structural/gate-level modeling

*vectors*
Vectors are used to group related signals using one name to make it more convenient to manipulate. For example, wire [7:0] w; declares an 8-bit vector named w that is functionally equivalent to having 8 separate wires.
`assign {o2, o1, o0} = vec;`

type [upper:lower] vector_name;

Implicit nets are often a source of hard-to-detect bugs. In Verilog, net-type signals can be implicitly created by an `assign` statement or by attaching something undeclared to a module port. Implicit nets are always one-bit wires and causes bugs if you had intended to use a vector. Disabling creation of implicit nets can be done using the `` `default_nettype none `` directive.
```verilog
wire [2:0] a, c; // Two vectors 
assign a = 3'b101; // a = 101 
assign b = a; // b = 1 implicitly-created wire 
assign c = b; // c = 001 <-- bug 
my_module i1 (d,e); // d and e are implicitly one-bit wide if not declared.   // This could be a bug if the port was intended to be a vector.
```
```verilog
reg [7:0] mem [255:0];   // 256 unpacked elements, each of which is a 8-bit packed vector of reg.
reg mem2 [28:0];         // 29 unpacked elements, each of which is a 1-bit reg.
```

Earlier, we mentioned that there are bitwise and logical versions of the various boolean operators (e.g., [norgate](https://hdlbits.01xz.net/wiki/norgate "norgate")). When using vectors, the distinction between the two operator types becomes important. A bitwise operation between two N-bit vectors replicates the operation for each bit of the vector and produces a N-bit output, while a logical operation treats the entire vector as a boolean value (true = non-zero, false = zero) and produces a 1-bit output.


*packed vs unpacked*

```verilog
reg [7:0] byte_val;        // packed: 8 contiguous bits, one 8-bit vector
wire [3:0][7:0] four_bytes; // packed 2D: still one contiguous 32-bit vector
reg [7:0] mem [0:255];     // unpacked: 256 separate 8-bit elements (like memory)
wire [3:0] data [0:15];    // unpacked: 16 separate 4-bit elements
```

|             | packed                           | unpacked                                      |
| ----------- | -------------------------------- | --------------------------------------------- |
| Storage     | Contiguous bits, one word        | Array of separate elements                    |
| Arithmetic  | Can do directly (`byte_val + 1`) | Can't operate on the whole array at once      |
| Typical use | Registers, buses, opcodes        | Memories, register files, arrays of registers |

*reduction operator*
```verilog
assign out_and = &in; // ANDs all 4 bits together 
assign out_or = |in; // ORs all 4 bits together 
assign out_xor = ^in; // XORs all 4 bits together
```

*concatenation operator*
```verilog
input [15:0] in;
output [23:0] out;
assign {out[7:0], out[15:8]} = in;         // Swap two bytes. Right side and left side are both 16-bit vectors.
assign out[15:0] = {in[7:0], in[15:8]};    // This is the same thing.
assign out = {in[7:0], in[15:8]};       // This is different. The 16-bit vector on the right is extended match the 24-bit vector on the left, so out[23:16] are zero. In the first two examples, out[23:16] are not assigned.
```
unsized constants are not allowed in concatenations.

*replication operator*
```verilog
{5{1'b1}}           // 5'b11111 (or 5'd31 or 5'h1f)
{2{a,b,c}}          // The same as {a,b,c,a,b,c}
{3'd5, {2{3'd6}}}   // 9'b101_110_110. It's a concatenation of 101 with
                    // the second vector, which is two copies of 3'b110.
assign out = {{24{in[7]}},in[7:0]};
```
modules
```verilog
mod_a instance1 ( wa, wb, wc );
mod_a instance2 ( .out(wc), .in1(wa), .in2(wb) );

    wire a,b;
    my_dff dff_1(.clk(clk),.d(d),.q(a));
    my_dff dff_2(.clk(clk),.d(a),.q(b));
    my_dff dff_3(.clk(clk),.d(b),.q(q));
```

A note on *wire vs. reg*: The left-hand-side of an assign statement must be a _net_ type (e.g., wire), while the left-hand-side of a procedural assignment (in an always block) must be a _variable_ type (e.g., reg).

- **Continuous** assignments (assign x = y;). 
- Procedural **blocking** assignment: (x = y;).
- Procedural **non-blocking** assignment: (x <= y;)

In a combinational always block, use blocking assignments. In a clocked always block, use non-blocking assignments.

always @(*) begin
    tmp = a & b;
    q   = tmp | c;
end

Blocking assignments execute **sequentially, immediately, in program order**—just like statements in a normal programming language. Each line finishes before the next one starts, so `tmp` is fully updated before it's used to compute `q`.

If you used non-blocking (`<=`) here instead, all the right-hand sides would be evaluated using **old** values from before the block executed, and updates would be scheduled to happen only after the block finishes. That would make `q` use the _old_ value of `tmp`, not the one just computed—introducing an unwanted one-step delay that doesn't exist in real combinational hardware

An if statement usually creates a 2-to-1 multiplexer, selecting one input if the condition is true, and the other input if the condition is false.

*reduction operator*
& a[3:0]   // AND: a[3]&a[2]&a[1]&a[0]. Equivalent to (a[3:0] == 4'hf)
| b[3:0]   // OR:  b[3]|b[2]|b[1]|b[0]. Equivalent to (b[3:0] != 4'h0)
^ c[2:0]   // XOR: c[2]^c[1]^c[0]
~& d[7:0]


if else,
conditions don't overlap, mux
conditiosn overlap, priority encoder


`timescale 1ns/1ps
Sets the simulation time unit to **1 nanosecond** and the time precision to **1 picosecond**. This means delays like `#10` are interpreted as 10 ns, and the simulator can resolve timing down to 0.001 ns.

Testbenches conventionally have no ports (no `input`/`output`)—they're the top-level module in simulation, driving signals into the design under test (DUT) directly.
```verilog
reg I0, I1, I2, I3;
reg S1, S0;
wire Y;
```
- `I0`–`I3` (data inputs) and `S1`, `S0` (select lines) are declared `reg` because the testbench will **drive** them procedurally inside an `initial` block.
- `Y` (the mux output) is declared `wire` because it's driven by the DUT, not by the testbench—`wire` is used to _read_ values the DUT produces.

`always @(*)` is used for **combinational logic** because it automatically infers the correct sensitivity list—the block re-evaluates whenever _any_ signal read inside it changes. This matches the fundamental nature of combinational circuits: the output should update immediately whenever _any_ input changes, since there's no clock or memory involved.

- **In simulation**: the block only re-evaluates when `a` or `sel` changes. If `b` changes while `sel = 0` (so `q` should now depend on `b`), the block **won't re-run**, and `q` keeps its old, now-incorrect value until `a` or `sel` happens to change again.
- **In synthesis**: real combinational hardware doesn't care about your sensitivity list—the physical logic gates respond to _any_ input change instantly, regardless of what you wrote. Synthesis tools build the actual mux/gate-level circuit based on the logic expressions themselves, not the sensitivity list.


latch inference bug,
if not every output is assigned in every branch (e.g., missing `else` or missing `default` in a `case`), the tool infers a latch to hold the old value—again causing simulation/hardware mismatches and creating an actual undesired sequential element (a latch) where you intended pure combinational logic.

The `` `timescale `` directive tells the simulator how to interpret time values used in your Verilog code (like `#10`)

With `1ps` precision, the simulator can represent delays down to 1 picosecond accuracy (i.e., time values are rounded to the nearest 0.001 ns). So you could technically write a delay like `#2.5` ns, and it would be tracked precisely (2500 ps) instead of getting rounded off to a whole nanosecond.

`` `timescale `` requires **both** a time unit and a time precision
`timescale 1ns/1ns

Here `posedge rst` is directly in the sensitivity list, so the block wakes up and clears `q` **the instant** `rst` rises — no waiting for a clock edge at all.

`posedge rst`                             plain  `rst`
Triggers onRising edge only (0→1)      Any change (0→1 or 1→0)

In a **Moore machine**, the output is a function of the **current state only** (not the inputs), so you get the output as soon as you **reach** (enter) that state — and it stays valid for the entire duration you remain in that state, until you transition to a different state.
```
PS=S2, X=0 → NS=S0, Y=1
```
This means: **"While currently in S2 (before this transition happens), the output is 1."** The fact that the machine is _about to_ transition to `S0` on this clock edge doesn't change what the output was _during_ S2. The `Y=1` is not describing the output _after_ moving to `S0` — it's describing the output _for the state you're currently in_ (S2), which happens to also be the row where you're computing what the next state will be.

**LVCMOS33** = Low Voltage CMOS at 3.3V
###### The Four Values

| Value | Meaning              | Real-world analogy                                                                 |
| ----- | -------------------- | ---------------------------------------------------------------------------------- |
| **0** | Logic low            | Signal driven to ground (0V)                                                       |
| **1** | Logic high           | Signal driven to supply voltage (e.g., 3.3V)                                       |
| **X** | Unknown / don't care | Signal state can't be determined — conflict, uninitialized, or genuinely ambiguous |
| **Z** | High impedance       | Signal is not being driven at all — floating, disconnected                         |


In a Moore machine, output is a function of the present state only, and it's "available" during the clock cycle the machine is _in_ that state — not during the transition into it.

|Modeling Style|Describes|Example|
|---|---|---|
|Behavioral|Algorithm/behavior|`always`, `if`, `case`|
|Dataflow|Boolean equations|`assign y = a & b;`|
|Gate-level|Individual logic gates|`and`, `or`, `not`, `nand`|
|Switch-level|Transistors|`nmos`, `pmos`|