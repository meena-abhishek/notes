timing and resynchronization penalty,

to change the clock frequency of a ddr channel, the memory controller must completely halt all incoming read/write requests, place the dram into a low-power self-refresh or idle state, change the clock speed, and wait for the DLL to reclock to the new frequency.
this can take several $\mu s$, which is a massive latency spike.
lets say we would dram channel into low frequency mode, when it needs to handle a sudden burst of traffic, then frequency need to be scaled up and will cause a latency spike due to reclocking, this speed penalty is not worth the energy savings.
bcoz when resync is happening, the cpu needs to stall its pipeline and need to sit idle burning leakage power without doing any useful work.
in HPC, traffic doesn't ramps up slowly, it arrives in burst.
the memory controller queue fills up and system becomes congested.

$E=P\times T$ ,
if we descrease the power, but time to complete a job increases, total energy consumed can actually go up.

in hpc, it is almost always energy-efficient to finish a work as fast as possible by running at max-speed and then putting chip into a sleep state.

before hbm, dram composed of only 10-15% of power in a server.
so cpu dvfs was more important.
since, dram was not scalable, we developed hbm.
a typicall, ai accelerator uses 6 to 8 HBM stacks. at 25w per stack, hbm alone can pull 150-200w. since, it represents a massive chunk of total power, we need to apply some power management techniques to hbm.
$$\text{P}_{\text{dynamic}} = \alpha \times C \times V^2 \times f$$
Where:
- **$\alpha$ (Activity Factor):** The percentage of transistors switching on any given clock cycle (ranges from 0 to 1). A heavy AVX-512 or AI workload has a high activity factor; idling has a very low one.
- **$C$ (Capacitance):** The physical capacitance of the transistors and wire traces on the silicon die (determined by the manufacturing node, e.g., 3nm vs 7nm).
- **$V$ (Voltage):** The operating voltage supplied to the silicon. **Crucially, this is squared ($V^2$).**
- **$f$ (Frequency):** The clock speed (in Hz).

Look at the $V^2$ relationship. If you lower the frequency ($f$) by 10%, power drops linearly by 10%. But if you can permanently drop the operating voltage ($V$) of DDR3 at 1.5V down to DDR5 at 1.1V (works at high frequency, because transistor size is reduced and operate fast at low voltage):
$$\left(\frac{1.1}{1.5}\right)^2 \approx 0.538$$
By simply dropping the standard voltage through architectural generation shifts, manufacturers **slashed dynamic power consumption by roughly 46%** across the board, completely eliminating the need to dynamically scale frequencies up and down during runtime.

since, hbm are physically close to xpu, there temperature can be high and thus leakage power can increase, we need DTM to reduce damage.

*3. interoperability between manufacturers*

idle , self-refresh state
instead of dvfs, because then there is need to co-operate between multiple manufacturers. coz if dram lower's its voltage without memory controller lowering the frequency, then there would be no synchronization between dram and memory controller.

DDR5 introduces FSP (frequency set points).
DDR5 uses a 16n prefetch, every single internal clock cycle, the core grabs 16 bits of data.

in hbm,
**Eliminating the Latency Penalty:** Because this logic die is built directly into the memory stack itself, it can manage the physical timing parameters and voltage rails of the DRAM layers with almost zero physical distance. It can orchestrate frequency shifts in real time without needing to send signals back and forth across a motherboard. The logic base die can isolate channels, meaning it can drop the frequency of one specific memory channel that isn't being heavily used while keeping other channels running at maximum speed. This completely eliminates the system-wide "latency freeze" that killed DVFS on traditional DDR memory.

since, hbm is places in same package as xpu and contributes 50% of power in servers for ai workloads, we need DTM for it.