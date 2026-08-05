In semiconductor electronics—particularly for CMOS circuits—**static or leakage current** ($I_{\text{leak}}$) is the current that flows through a transistor or circuit when it is in a steady-state (inactive/off) condition.

Unlike dynamic current, which flows during switching due to charging and discharging capacitors, static current flows continuously as long as power is applied.

Here is how you calculate static current at both the single-transistor level and the full-chip/macro level.

## 1. The Components of Leakage Current

To calculate total static current, you must account for its three primary physical sources:

$$I_{\text{leak}} = I_{\text{sub}} + I_{\text{gate}} + I_{\text{junction}}$$

### A. Subthreshold Leakage ($I_{\text{sub}}$)

This is the dominant leakage component in modern nanometer nodes. It occurs when the gate voltage ($V_{\text{gs}}$) is below the threshold voltage ($V_{\text{th}}$), but a weak inversion layer still allows current to drift between the drain and source.

It is calculated using the BSIM model formula:

$$I_{\text{sub}} = I_0 \cdot e^{\frac{V_{\text{gs}} - V_{\text{th}}}{\eta V_{\text{T}}}} \cdot \left(1 - e^{-\frac{V_{\text{ds}}}{V_{\text{T}}}}\right)$$

Where:

- $I_0 = \mu_0 \cdot C_{\text{ox}} \cdot \frac{W}{L} \cdot (V_{\text{T}})^2 \cdot (\eta - 1)$
    
- $V_{\text{T}}$ is the thermal voltage ($\approx 26\text{ mV}$ at room temperature).
    
- $\eta$ is the subthreshold swing coefficient (typically $1.1$ to $1.4$).
    
- $W$ and $L$ are the transistor width and length.
    

### B. Gate Oxide Tunneling Leakage ($I_{\text{gate}}$)

As oxide thickness ($T_{\text{ox}}$) shrinks to a few nanometers, electrons quantum-mechanically tunnel straight through the gate dielectric.

It is modeled by the Fowler-Nordheim (FN) tunneling equation or direct tunneling approximations:

$$I_{\text{gate}} = A \cdot E_{\text{ox}}^2 \cdot e^{-\frac{B}{E_{\text{ox}}}}$$

Where:

- $E_{\text{ox}}$ is the electric field across the oxide ($V_{\text{gs}}/T_{\text{ox}}$).
    
- $A$ and $B$ are physical constants dependent on the electron effective mass and barrier height.
    

### C. Reverse-Biased Junction Leakage ($I_{\text{junction}}$)

This is the parasitic diode current flowing from the source/drain diffusions to the substrate/well. It consists of band-to-band tunneling (BTBT) and minority carrier generation:

$$I_{\text{junction}} = I_{\text{S}} \cdot \left(e^{\frac{V_{\text{bias}}}{V_{\text{T}}}} - 1\right)$$

## 2. Chip-Level / Macro-Level Calculation

In practical IC design and power analysis, calculating leakage using raw physics equations for billions of transistors is impossible. Instead, engineers use **Standard Cell Library Characterization** (using tools like Synopsys PrimeTime or Cadence Innovus).

### Method A: Summation of State-Dependent Leakage

The leakage of a digital gate depends heavily on its **input state**. For example, a 2-input NAND gate leaks differently when its inputs are `00` vs. `11` because different transistors are turned off or stacked.

The total static current of a block is calculated as:

$$I_{\text{static\_total}} = \sum_{i \in \text{cells}} \sum_{s \in \text{states}} P_i(s) \cdot I_{\text{leak}, i}(s)$$

Where:

- $I_{\text{leak}, i}(s)$ is the leakage current of cell $i$ in input state $s$ (looked up from the `.lib` timing/power file).
    
- $P_i(s)$ is the **state probability** (the percentage of time the cell spends in state $s$, derived from logic simulation files like VCD).
    

### Method B: Experimental/Hardware Measurement

If you have a physical silicon chip on a testbench, calculating static current is straightforward:

1. **Stop the clocks:** Disable all clock signals to the chip ($f = 0\text{ Hz}$). This eliminates all dynamic power ($P_{\text{dyn}} = C V^2 f$).
    
2. **Ensure steady state:** Wait a few microseconds for any residual switching transients to settle.
    
3. **Measure Current:** Use a Source Measure Unit (SMU) or an ammeter to measure the current drawn directly from the $V_{\text{DD}}$ supply rail. This total measured current is your static/leakage current.
    

## Key Environmental Factors to Watch

When calculating or simulating static current, you must strictly define three variables because leakage is highly non-linear:

- **Temperature ($T$):** Subthreshold leakage increases exponentially with temperature. A chip at 85°C will leak drastically more than at 25°C.
    
- **Supply Voltage ($V_{\text{DD}}$):** Higher voltage increases DIBL (Drain-Induced Barrier Lowering), dropping $V_{\text{th}}$ and compounding subthreshold and gate leakage.
    
- **Process Corner:** Fast process corners (where transistors have shorter channel lengths or lower $V_{\text{th}}$) will exhibit significantly higher static current than Typical or Slow corners.
    

Are you looking to calculate this for an academic device-physics problem, or are you trying to estimate power bounds for an ASIC/VLSI design?