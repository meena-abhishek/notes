The technique used to push performance within a thermal budget is called **boosting** — temporarily raising the voltage/frequency (V/f) of cores above their nominal operating point (like Intel TurboBoost/AMD Turbo Core).

The core question SmartBoost asks: **given a fixed thermal budget, which cores/applications should you boost (and which should you throttle) to maximize overall system performance?**


For example, roughly:
**5 V → 3.3 V → 1.8 V → 1.2 V → ~1 V**

That was fantastic because lowering voltage reduced switching energy dramatically.
But eventually you hit physical limitations.
You can't keep reducing voltage indefinitely because transistors need enough voltage to distinguish **ON** from **OFF** and to overcome various noise, leakage, and threshold-voltage constraints.
So transistor dimensions kept shrinking, but **voltage stopped shrinking proportionally**.
That's the point where Dennard scaling broke down.

*And this creates the modern problem*
Suppose you shrink a transistor so its area becomes **1/4**.
You'd like its power to also become **1/4**.
But if voltage can't fall as much, you might only get, say, **1/2 the power per transistor**.
Now you can fit 4× as many transistors:
> 4 × 1/2 = **2× the power**


| Approach                                      | What it considers                                                                                                                                     | What it misses                                                                                                                                                            |
| --------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Intel TurboBoost / AMD Turbo Core**         | Nothing about individual apps — boosts/throttles everything uniformly if there’s thermal margin                                                       | Ignores that different applications benefit very differently from boosting                                                                                                |
| **IPS-based boosting (e.g., Procrustes)**     | Boosts high-IPS (instructions/sec) applications, throttles low-IPS ones                                                                               | Absolute IPS is a poor proxy — a low-IPS app can still be extremely _sensitive_ to frequency changes (i.e., benefit a lot from boosting)                                  |
| **Performance-sensitivity-based techniques**  | Uses ∂performance/∂frequency (a much better metric)                                                                                                   | Ignores that boosting one app costs _power_, and power costs vary drastically between apps. Doesn’t consider _where_ on the chip the heat lands (temperature sensitivity) |
| **DTPM (state-of-the-art, Bhat et al. 2017)** | Partially combines performance, power, temperature — but power/temperature are only used to decide _when_ to boost, not _which_ application to select | Application selection is based on performance sensitivity alone, ignoring that boosting a low-power app is “cheaper” thermally than boosting a high-power app             |

#### The Key Idea / Contribution
SmartBoost introduces a **single unified metric called "boostability"** that mathematically combines:

1. **Performance sensitivity** (s^perf) — how much an app's IPS changes with frequency
2. **Power sensitivity** (s^pw) — how much an app's power draw changes with frequency
3. **Temperature sensitivity** — how much a given core's power change affects the hottest point on the chip (via the chip's thermal conductivity structure)


https://chatgpt.com/share/6a8af0d0-514c-83e9-b89c-23e0f3ef2596


#### Novelty vs. Prior Work

The clearest way to see the novelty is Figure 4 in the paper, which the authors use explicitly:

- **DTPM** (previous SOTA): decouples the decision — it uses performance/power/temperature sensitivities only to decide _when_ boosting is safe, but selects _which_ app to boost using performance sensitivity alone.
- **SmartBoost**: uses _all three_ sensitivities jointly, for _both_ the "select application" and "select V/f level" decisions.


$T = B⁻¹ · P + T_{amb}$

- `T`: vector of steady-state temperatures, one per thermal node/core.
- `P`: vector of power consumption per core.
- `B⁻¹`: an n×n matrix encoding the thermal conductivity structure of the chip — essentially "how much does raising power at core _i_ raise temperature at core _j_." Cores near each other, or near the heatsink attachment point, have different thermal coupling.
- `T_amb`: ambient temperature (the baseline, e.g. 45°C in this paper).

This matrix `B⁻¹` is why the paper says "the same power on different cores can result in different temperatures" — a core near the edge of the die dissipates heat more easily (lower B⁻¹ entries) than a core buried in the middle surrounded by other hot cores

A **CPI stack** (Cycles Per Instruction stack) is a way of decomposing total execution cycles into "how many cycles were spent because of X" — e.g., cycles lost to L1 misses, L2 misses, DRAM latency, branch mispredictions, etc., plus a "base" component for useful compute. It's essentially an attribution/breakdown of where time went, and it's cheap to derive from standard performance counters. SmartBoost uses CPI stacks as **input features** to its neural network because they encode _why_ a thread behaves the way it does (e.g., a high "memory" component in the CPI stack signals a memory-bound thread, which we'd expect to benefit less from frequency boosts, since a faster clock doesn't make DRAM respond faster).

#### 2.6 Why Threads of the Same Application Should Share a Frequency

If one thread in a parallel application runs at 4 GHz and another at 1 GHz, the fast thread finishes its portion of work early and then **stalls waiting** at synchronization points (barriers, locks) for the slow thread. This wastes the boost — the fast thread's speed doesn't help the _application's_ overall completion time, and might even be wasted energy. So SmartBoost aggregates thread-level sensitivities up to the _application_ level (Eq. 6) rather than optimizing each thread independently.

**Multi-task learning**: instead of training two separate models (one for performance sensitivity, one for power sensitivity), train _one_ model with two output heads that share the internal (hidden layer) representations. This works well when the tasks are correlated — which the authors argue is the case here (a memory-bound thread has both low performance sensitivity _and_ low power sensitivity, since it isn't fully utilizing the pipeline). Sharing representations reduces redundant computation (lower inference overhead) and can improve generalization (regularization effect) especially with limited training data — which matters because the model must generalize to unseen applications.



**Purpose**: Motivate why naive boosting heuristics fail, using a concrete example (Fig. 1, Fig. 2) before presenting the formal contribution.

**Figure 1 (Motivational Example)**: Two SPLASH-2 apps, `lu.cont` (high IPS, high power) and `radix` (low IPS, low power), run together.

- Without boosting: both run at 2.0 GHz; `radix` takes 422ms, `lu.cont` takes 210ms roughly (numbers annotate execution time).
- With boosting: they _throttle_ `lu.cont` slightly and _boost_ `radix` a lot. Result: `radix`'s time drops from 422ms→~280ms(33% faster), while `lu.cont` only gets ~2% slower — and crucially, **peak temperature stays the same**.

This is counterintuitive: intuitively you'd boost the "important-looking" high-IPS app. But the insight is that `lu.cont`'s high power consumption means boosting it costs a lot of thermal budget for a small performance gain, while `radix` is cheap (low power) to boost and — despite low absolute IPS — is very compute-intensive, meaning its performance responds strongly to more frequency.


*high ips doesn't mean compute intensive*
an application could execute an enormous number of simple instructions while doing relatively trivial computation.
Meanwhile, a scientific application performing a relatively small number of extremely expensive calculations could have a much lower IPS but consume substantial computational resources.




- **Design-time** (top): profile benchmarks in HotSniper at multiple frequencies and parallelism settings → generate (input features, sensitivity label) pairs → train the sensitivity NN model.
- **Run-time** (bottom): periodic loop reads per-thread frequency + performance counters → NN predicts sensitivities → three algorithms run in sequence: throttle-to-avoid-violation, boost-to-exploit-margin, throttle-one-to-boost-another.


- **Algorithm 1 (top-level)**: for each thread, run the NN to get sensitivities; then for each application, set frequency bounds for this epoch (limited to ±k steps to keep the linear sensitivity approximation valid, since sensitivities are frequency-dependent and only locally accurate); then run the three sub-algorithms in order.
- **Algorithm 2 (throttle to avoid violation)**: if predicted steady-state temperature exceeds Tcrit, iteratively throttle the application with the _lowest_ boostability-for-throttling `b↓` (i.e., "cheapest to throttle" in terms of performance lost per degree of thermal relief gained) until safe.
- **Algorithm 3 (boost to exploit margin)**: if there's spare thermal margin, iteratively boost the application with the _highest_ boostability-for-boosting `b↑` as far as thermally safe.
- **Algorithm 4 (throttle one to boost another)**: this is the genuinely novel move — even when at a stable/safe point, check whether swapping some frequency from a low-boostability app to a high-boostability app yields a **net performance gain** (Δperf > 0), and if so, do it. This exploits exactly the Fig. 1 motivational scenario (throttle `lu.cont` a little, boost `radix` a lot).

### 4. Methodology Deep Dive

#### Step-by-step Runtime Flow (Every ~1ms Epoch)

1. **Sense**: For every currently active thread `i`, read its current frequency `f_i` and its performance-counter vector `C_i` (CPI stack breakdown).
2. **Predict**: Feed `(C_i, f_i)` into the trained NN → get `(s_perf_i, s_pw_i)`.
3. **Estimate current thermal state**: Using current per-core power (from `P`) and the precomputed thermal matrix `B⁻¹`, compute current steady-state temperature estimate `T = T_amb + B⁻¹P`.
4. **React** (in strict order, because they're mutually exclusive concerns per epoch):
    - a. **Safety first** (Algorithm 2): If projected temperature > Tcrit, throttle the cheapest-to-throttle application repeatedly until safe.
    - b. **Opportunistic boosting** (Algorithm 3): If there's margin, boost the best-boostability application as far as safely possible.
    - c. **Rebalancing** (Algorithm 4): Even with no margin, check if a throttle-swap between the worst and best boostability apps yields net performance gain.
5. **Apply** the resulting frequency vector for the next epoch, and repeat.

*Boostability (Single Thread)*

`b = s_perf / (B⁻¹_ih · s_pw)`
where `h` = index of the current hotspot core (`h = arg max_j T_j`).

Interpretation: **numerator** = how much relative performance you gain per unit frequency; **denominator** = how much the _hotspot temperature specifically_ rises per unit frequency (using Eq. 4 with `j = h`). So `b` = "performance gained per degree Celsius spent at the hotspot" — literally the exchange rate between the scarce resource (thermal headroom at the hottest point on the chip) and the desired outcome (performance). This is why it's the natural metric to rank applications by: whoever gives you the most performance per degree of thermal budget should get boosted first, and whoever gives you the least performance-loss-avoided per degree should be throttled first when you need to cool down.

Note the **hotspot-relative** nature: this is not just "how much does _my own_ core heat up" — it's "how much does the _currently hottest point on the whole chip_ heat up," because that's the binding constraint (`max_j T_j > Tcrit`). This correctly captures your intuition from Eq. 4: a core far from the hotspot (small `B⁻¹_ih`) can be boosted more cheaply in terms of thermal budget, even at equal power sensitivity, than a core physically/thermally close to the hotspot.

Radix has the lowest absolute IPS
but the highest sensitivity of the performance. Canneal has a similar
absolute IPS but a low sensitivity of the performance. Hence, the
sensitivity of the performance is the important metric to be included
into the boosting decision instead of the absolute IPS.


In addition to performance and power, temperature also needs
to be included into the boosting optimization. The reason is that
temperature and power are not interchangeable. The same power con-
sumption on different cores can result in very different temperatures,
e.g., because of the location of the cores on the die, or because of
the power consumption of other cores. That implies the cores on
the chip have different sensitivity of the temperature. In summary,
three metrics need to be included into the boosting optimization: the
application-dependent V/f sensitivities of the performance and power,
and the core-dependent sensitivity of the temperature.

Both are **derivatives with respect to frequency** — i.e., "how much would IPS/power change _if_ I changed the frequency right now." To measure a derivative directly, the textbook way is:

1. Run the thread at frequency `f`, measure `IPS(f)` and `P(f)`.
2. Change to a nearby frequency `f + Δf`, measure `IPS(f+Δf)` and `P(f+Δf)`.
3. Compute the finite-difference: `s_perf ≈ [IPS(f+Δf) − IPS(f)] / (Δf · IPS(f))`.



Q. The application-dependent metrics, i.e., the V/f sensitivities of performance and power, cannot be directly measured and, hence, need to be estimated at run-time.

Good catch — this line is Challenge 1 from Section I-B, and it's worth unpacking carefully because it's the entire justification for why the paper needs machine learning at all instead of just... measuring the darn sensitivities directly.

#### Why can't you just measure `s_perf` and `s_pw` directly?

Look again at the definitions:

```
s_perf = (∂IPS/∂f) · (1/IPS_c)
s_pw   = ∂P/∂f
```

Both are **derivatives with respect to frequency** — i.e., "how much would IPS/power change _if_ I changed the frequency right now." To measure a derivative directly, the textbook way is:

1. Run the thread at frequency `f`, measure `IPS(f)` and `P(f)`.
2. Change to a nearby frequency `f + Δf`, measure `IPS(f+Δf)` and `P(f+Δf)`.
3. Compute the finite-difference: `s_perf ≈ [IPS(f+Δf) − IPS(f)] / (Δf · IPS(f))`.

This is exactly what the paper does **offline**, at design time, to generate training data (that's literally the "Trace at frequency f1 / Trace at frequency f2" comparison step in Fig. 3). So in principle, direct measurement _is_ possible — but only if you're willing to pay a real cost:

- **You have to actually change the frequency and wait**, then change it back. That's disruptive — every measurement of the derivative requires perturbing the very thing you're trying to optimize.
- **The thread's behavior changes over time** (different execution phases — this is why they slice traces into 20 phases per app). By the time you've measured `IPS(f+Δf)`, the thread may have moved into a different phase, so the "before" and "after" measurements aren't really measuring the same underlying behavior — you've introduced noise/confound into your derivative estimate.
- **You'd need to do this continuously**, for every thread, every epoch (every 1ms), for the runtime algorithm to have fresh sensitivity values — that's an enormous amount of live perturbation and lost performance just to _measure_ the thing you want to _optimize_.

So "cannot be directly measured" really means: **you cannot measure them non-invasively and cheaply enough to use every millisecond, for threads whose only currently observed frequency is a single point** (whatever frequency they happen to be running at _right now_). You only ever get to observe the system at _one_ V/f operating point at any given instant — you don't get to also simultaneously observe it at a slightly different frequency to compute the slope.

#### This is a classic "we only observe one point on a curve" problem

It's the same fundamental issue you'd hit in, say, estimating the derivative of a cost function when you can only ever query it at your current parameter setting, never nearby ones simultaneously. You have two options:

1. **Perturb and remeasure** (expensive, disruptive, and confounded by non-stationarity — the thread's own behavior drifts).
2. **Learn a model** that maps _cheaply-observable, single-point features_ (frequency + performance counters, i.e., the CPI stack) to the derivative you actually want — trained _once_, offline, using exactly the perturb-and-remeasure approach from option 1, but paying that cost only during training, not at runtime.

SmartBoost picks option 2. The performance counters (CPI stack breakdown: how much of the current cycle count is attributable to L1 misses, L2 misses, DRAM stalls, etc.) are the "cheap, single-point, non-invasive" signal — you get them for free from hardware counters at the thread's _current_ operating point, no perturbation needed. The NN's job is to have learned, from many prior perturb-and-remeasure experiments across diverse apps, the _mapping_ from "what your CPI stack looks like right now" to "what your sensitivity would be if I nudged your frequency."

#### Why this specifically requires generalization (the "unknown application" problem)

There's a second, subtler reason this is hard, layered on top of the measurement problem: the paper targets an **open system** where applications arrive that were _never seen during training_. Even if you were willing to pay the perturb-and-remeasure cost at design time for a fixed, known set of benchmarks, that doesn't help you at runtime when a brand-new, never-profiled application shows up — you have no historical (frequency, sensitivity) pairs for it specifically.

This is exactly why:

- The model needs to be trained on a **diverse enough set of design-time benchmarks** (22 apps across PARSEC/SPLASH-2, spanning compute-intensive to memory-intensive) so that it learns the _general relationship_ between CPI-stack features and sensitivities, not memorized values for specific apps.
- The features chosen (frequency + memory-intensity ratio) are deliberately **application-agnostic, behavior-based** signals rather than, say, "app ID" — so the model generalizes based on _what the thread is doing right now_ (its resource-usage fingerprint) rather than _which specific program it is_.
- The held-out test in Fig. 7 (canneal, x264, barnes, fft — excluded from training) is the direct empirical validation that this generalization actually works, and reasonably well (0.80%/step RMSE vs DTPM's 2.15%/step).

#### One subtlety worth flagging

Notice the paper's own footnote 2: _"An even simpler model could be employed for the prediction, but with a lower accuracy."_ This tells you the authors are aware there's an accuracy/complexity tradeoff in _how_ you solve the estimation problem — they didn't need an NN specifically, they needed _some_ function approximator that could learn CPI-stack → sensitivity mappings, and chose an NN because it hit a good accuracy/overhead sweet spot (evaluated in Fig. 7 vs. DTPM's simpler linear model, and in the 8.1µs runtime overhead measurement).

Boosting techniques
differ in which metrics (performance, power, temperature) are con-
sidered in the optimization and how they are considered.


So EDP strongly penalizes a design that is simultaneously energy-hungry **and** slow.

![[smartboost.png]]

![[temp_sensitivity.png]]

To maximize the system performance under a thermal constraint,
the thread whose performance increases the most for the available
thermal margin should be boosted. Therefore, we build a single
metric that we simply call boostability. It integrates the sensitivities
of performance, power, and temperature into a single metric to
indicate the change in the performance per peak temperature change
(unit: %/◦C). 
The peak temperature is determined by the core with the highest temperature (hotspot). We denote its index h = arg maxj Tj.
For a thread running on core i, its boostability b is computed by:
![[boostability.png]]

The best boosting benefit can be obtained by boosting the thread
with the highest value of the boostability. When throttling is required
to avoid potential thermal violations, the thread with the lowest
boostability value should be throttled first, because that will lead
to the minimum performance penalty to maintain thermal safety.

*features*
1. We select the frequency as the first feature because it has the highest impact on the sensitivities of performance and power.
2. A crucial characteristic of a thread is its memory-intensiveness. Themore memory-intense a thread is, the lower are its sensitivities of theperformance and power. We select the ratio of all memory-related CPI stack parts over the total CPI as a second feature.

Since the sensitivities of power, performance, and temperature are
properties of the threads and the cores they are running on, we
need to combine them into a single metric per each application.
The sensitivities of power and temperature simply need to be added
because the individual impacts of the threads accumulate on the
thermal hotspot. For the sensitivity of the performance, it plays a
role whether the application should be boosted or throttled. The
thread with the minimum sensitivity of the performance is affected the
least by boosting and determines the application’s sensitivity of the
performance. In contrast, the thread with the maximum sensitivity
of the performance is affected the most by throttling, and, hence,
needs to be considered for throttling. In summary, Eq. (5) needs to be
extended to define the boostability of an application for boosting (b↑)
and throttling (b↓) based on the sensitivities of its set of threads A:
![[boosting_per_thread.png]]

