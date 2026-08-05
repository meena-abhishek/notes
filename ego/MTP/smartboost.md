boosting : 
lets a cpu core temporarily run at a higher clock speed than its base frequency.

since $P_{dyn} \propto CV^2f$ and reducing voltage means also reducing frequency, slight voltage drop means power drops cubicly.

silicon and package thermal RC time constants are in the ms-to-s, while workload phases change much faster. This mismatch means there's usually _thermal slack_ — a core can run hot briefly without the temperature actually reaching the limit yet, because heat takes time to accumulate. DVFS-based boosting exploits exactly this slack: ramp up during the thermal "lag," back off before the limit is reached.

TDP (Thermal Design Power) is the amount of heat the cooling solution needs to be able to dissipate so the chip can sustain its **base clock** indefinitely.

"If you want this chip to run at its rated base speed forever without overheating, your cooler needs to be capable of removing TDPw continuously."

Existing boosting techniques decide which core(s) to boost by looking at how sensitive an application's **performance** is to voltage/frequency

Dennard scaling said "smaller transistors = same power density, so clock speed can keep rising for free." Once voltage stopped scaling around the mid-2000s, power density started climbing with every node instead — which is the root cause of why chips became thermally constrained, why single-core frequency growth stalled, why multicore took over, and why sophisticated DVFS/boosting (like what SmartBoost proposes) became a central research problem instead of an afterthought.



IPS (instructions per second) = IPC × clock frequency. So low IPS at a fixed frequency means **low IPC** (high CPI — many cycles per instruction). But high CPI can come from _two very different root causes_:

1. **Memory-bound reason**: the core is stalled waiting on cache misses / DRAM — pipeline sits idle, no useful compute happening. This is the classic "low IPS = not compute intensive" case.
2. **Compute-bound reason**: the instructions themselves are inherently expensive to execute — long-latency functional units (e.g., floating-point divide, square root, transcendental functions like sin/cos/exp), or long dependency chains where each instruction must wait for the previous result (low instruction-level parallelism), or wide SIMD/vector operations that take many cycles to complete even though they're 100% compute, not memory-stalled.

In case 2, the core isn't stalled on memory at all — the **execution units are fully busy**, drawing significant dynamic power, generating heat — but throughput in _instructions/sec_ is still low because each instruction takes many cycles to complete or instructions can't be issued back-to-back due to true data dependencies.

- Memory-bound low-IPS app → boosting frequency barely helps (still bottlenecked on memory latency).
- Compute-bound low-IPS app (long dependency chains/expensive ops) → boosting frequency helps _a lot_, since the execution units themselves are the bottleneck.

Radix has the lowest absolute IPS
but the highest sensitivity of the performance. Canneal has a similar
absolute IPS but a low sensitivity of the performance. Hence, the
sensitivity of the performance is the important metric to be included
into the boosting decision instead of the absolute IPS.
lu is high ips workload.

In summary,
three metrics need to be included into the boosting optimization: the
application-dependent V/f sensitivities of the performance and power,
and the core-dependent sensitivity of the temperature.

https://claude.ai/share/18e820cf-461b-4247-bd39-66959e498e8d

This is exactly why they derive **one single combined metric**, boostability: b=sperfBih−1⋅spwb = $\dfrac{s^{perf}}{B^{-1}_{ih}\cdot s^{pw}}​$ — performance sensitivity in the numerator, (thermally-weighted) power sensitivity in the denominator. This directly encodes the tradeoff as a single number: "percent performance gained per degree of hotspot temperature rise." Now applications can be ranked on one consistent scale that already accounts for the contradiction, instead of needing arbitrary tie-breaking rules between separately-ranked metrics.

NN predicts sensitivities → sensitivities feed into the boostability formula → boostability drives the SmartBoost algorithm's boost/throttle decisions every 1ms epoch.

This is $B^{-1}_{ih}$ — how much a change in power on core ii i translates into a temperature change at the hotspot core h.

Boostability captures: "if I raise this application's frequency, (a) how much faster does it run [performance sensitivity], (b) how much more power does it draw [power sensitivity], and (c) given _where this specific core physically sits on the die_, how much does that extra power actually raise the hotspot temperature [temperature sensitivity]?"
So the more precise version of your summary: boostability = (performance gain per GHz) ÷ (temperature rise at the hotspot per GHz), where that temperature rise itself is computed as power-sensitivity × thermal-conductivity-to-hotspot. All three physical effects (performance response, power response, and thermal propagation) are captured, but the last two are combined multiplicatively rather than being three separate additive terms.

since DVFS in this model appears to be applied per-core (per-thread), but you generally don't want an application's threads running at wildly different frequencies (footnote reference [24], "IPC Considered Harmful for Multiprocessor Workloads," warns about synchronization stalls if communicating threads run at different speeds). That's the reasoning behind Eq. (6): use the _minimum_ thread sensitivity across the application when deciding whether to boost (since the slowest-benefiting thread caps the real speedup you'd get by boosting the whole application), and the _maximum_ thread sensitivity when deciding whether to throttle (since that's the thread that would be hurt most by slowing down).

it's "how much does changing the frequency of _this specific thread_ (running on core ii i) raise the temperature at the _hotspot_ (core hh h)

A ratio, however, cancels the GHz in both numerator and denominator, leaving %/°C — **"performance gained per degree of thermal budget spent."**

**Step 3: Generate training examples by pairing slices across frequencies**

This is the key trick for getting _sensitivity_ labels (which, remember, are derivatives — you can't measure a derivative from a single frequency's trace alone; you need to compare two nearby points).

For a given thread, application, and parallelism setting, they take the _same_ execution phase (i.e., the same slice index, so it's comparing "the same chunk of work") recorded at **two different frequencies** — call them f1f_1 f1​ and f2f_2 f2​. Since everything else is held fixed (same application, same parallelism, same phase of execution), the _only_ difference between the two traces is the frequency, so any difference in IPS or power can be attributed to that frequency change. From this pair, they compute the empirical sensitivities:

$sperf\approx \frac{IPS(f_2) - IPS(f_1)}{f_2 - f_1} \cdot \frac{1}{IPS(f_1)}, \quad s^{pw} \approx \frac{P(f_2) - P(f_1)}{f_2 - f_1}​$

We select the frequency as the first feature because it has
the highest impact on the sensitivities of performance and power.
A crucial characteristic of a thread is its memory-intensiveness. The
more memory-intense a thread is, the lower are its sensitivities of the
performance and power. We select the ratio of all memory-related
CPI stack parts over the total CPI as a second feature.

