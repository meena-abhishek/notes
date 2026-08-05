inference - forward propagation
training - forward + backward propagation

| Aspect                | How DNNs use HBM                                                                                                                                                                               |
| --------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Bandwidth demand**  | DNNs need 1–3+ TB/s memory bandwidth; HBM provides this via 8–16 independent channels with 1024–2048-bit wide bus [wikipedia](https://en.wikipedia.org/wiki/High_Bandwidth_Memory)             |
| **Concurrent access** | Multiple channels operate independently; DNN accelerators parallelize memory requests across channels [wikipedia](https://en.wikipedia.org/wiki/High_Bandwidth_Memory)                         |
| **Forward phase**     | Loads weights + input activations; computes layer outputs; stores activations for later backward pass eems.mit+1                                                                               |
| **Backward phase**    | Re-loads stored forward activations + weights; computes gradients; updates weights eems.mit+1                                                                                                  |
| **Data movement**     | Activations must remain in memory between forward/backward passes, causing high memory usage [surakuma.github](https://surakuma.github.io/CR12/Project-articles/DeepNeuralNetworkTraining.pdf) |

|Traditional micro-bump stacking| Hybrid bonding                                                                                                                                                                 |
|---|---|
|Solder bumps create thermal resistance| Direct Cu-to-Cu + dielectric bonding eliminates bumps baslerweb+1                                                                                                              |
|Limited to ~12 layers before thermal issues| Enables 16–24+ layers with better heat dissipation x+1                                                                                                                         |
|Higher junction-to-junction thermal resistance| **Cuts thermal resistance by 22.8–47%** [semiengineering](https://semiengineering.com/examination-of-thermal-issues-related-to-hybrid-bonding-of-3d-stacked-hbm/)              |
|Lower vertical thermal conductivity| **Increases vertical thermal conductivity up to 3×** [semiengineering](https://semiengineering.com/examination-of-thermal-issues-related-to-hybrid-bonding-of-3d-stacked-hbm/) |
|~1000 I/Os, larger pitch (10+ µm)| **15× higher connection density, sub-10 µm pitch** [x](https://x.com/Jukanlosreve/status/1986760641051566397)                                                                  |
|Thermal efficiency baseline| **Improves thermal efficiency ~100×** [x](https://x.com/Jukanlosreve/status/1986760641051566397)<br>                                                                           |

Because all data flowing into or out of the entire 4, 8, or 12-layer DRAM stack _must_ pass through this single logic die, it consumes significantly more power per unit area than any individual DRAM layer

Yes, the output of a layer—typically referred to in deep learning hardware architecture as the **intermediate feature map** or **activations**—must be stored in memory so the next layer can read it as its input.

However, _where_ and _how long_ it is stored depends entirely on whether the hardware is executing a **Forward Pass (Inference)** or a **Backward Pass (Training)**.

## 1. During Inference (Forward Pass Only)

When a neural network is just making a prediction, memory management is highly transient.

- **The Flow:** Layer $N$ computes its outputs, writes them to memory, and then Layer $N+1$ reads those outputs as its inputs.
    
- **Overwriting Memory:** Once Layer $N+1$ finishes its computation, the outputs of Layer $N$ are usually **no longer needed**. To save precious hardware memory, the system will overwrite that memory buffer with the outputs of subsequent layers.
    
- **The Exception (Skip Connections):** In architectures like ResNets or Transformers (Residual Stream), the output of an early layer is added to the output of a later layer. In these cases, the intermediate activations must be kept alive in memory longer until that addition operation is completed.
    

## 2. During Training (Forward + Backward Pass)

During training, the memory footprint explodes dramatically because of the **Backward Pass (Backpropagation)**.

- **Why they must be kept:** To calculate the gradients of the weights for Layer $N$ during backpropagation, the system mathematically requires the exact activation values that were calculated during the forward pass.
    
- **The Lifetime:** The activations for _every single layer_ must be cached and held in memory. They cannot be overwritten because the backward pass travels in reverse. The activations for Layer 1, computed at the very beginning of the forward pass, must sit in memory until the very end of the backward pass.
    

> This accumulation of intermediate activations across all layers is the primary reason training large DNNs requires massive amounts of high-bandwidth memory (HBM) and often hits Out-Of-Memory (OOM) errors.

## Where exactly in memory is it stored?

Because moving data consumes orders of magnitude more energy than processing it, hardware compilers try to keep activations as close to the compute units as possible:

1. **On-Chip SRAM (SRAM/Scratchpad/Cache):** If a layer's output is small enough, the compiler will keep it entirely inside the processor's ultra-fast internal memory (like the SRAM tiles on a TPU or Tensor Core). Layer $N+1$ reads it immediately from there, avoiding external memory trips.
    
2. **Off-Chip DRAM/HBM (Main Memory):** If the feature maps are too large to fit on-chip (which is common with high-resolution images or massive batch sizes), they are written out to HBM/DRAM.
    

### Optimizations Used in Modern Accelerators

Because writing these intermediate outputs to off-chip HBM creates massive thermal and bandwidth bottlenecks, compilers use advanced optimization tricks:

- **Operator Fusion:** Modern compilers (like XLA or PyTorch Inductor) will "fuse" adjacent layers together (e.g., combining a Convolution, a BatchNorm, and a ReLU into a single execution step). This allows the intermediate outputs to stay strictly inside the processor's registers/SRAM, completely bypassing the need to write them to main memory.
    
- **Activation Checkpointing (Rematerialization):** During training, instead of storing all activations, the system throws away some intermediate layer outputs. If they are needed during the backward pass, the system quickly re-computes them on the fly. This trades a little extra computation time to save massive amounts of memory capacity.