reducing supply voltage below a certain point introduces bit errors in the data.
the gap between 0 and 1 shrinks
slower charging/discharging, lower voltage reduces transistor drive current.
leakage becomes more significant relative to stored charge

these errors can be avoided by increasing latency of activation, restoration and precharge operations.

DRAM: dram array and peripheral circuitry

array-> circuitry -> channel -> controller

voltron,
observed that we can reliably access data when DRAM supply voltage is lowered below the nominal voltage, until a certain voltage value $V_{min}$.

we can prevent these errors below $V_{min}$ if we increase latency of operations.
when the supply voltage is reduced, the capacitor charge takes a longer time to change, causing these DRAM operations to become slower to complete. Errors are introduced into the data when the memory controller does not account for this slowdown.

voltron found that if memory controller allocated extra time for these operations when supply voltage is below $V_{min}$ errors no longer occur.

when only small number of errors occur due to reduced supply voltage, these errors tend to cluster physically. so only these regions latency needs to be increased.

supply voltage does not impact data retention gaurantee, coz capacitor leakage is so slow that no data is lost during the 64ms refresh interval.
- array voltage scaling
- performance-aware voltage control

$power \propto {Voltage}^2\times{frequency}$
