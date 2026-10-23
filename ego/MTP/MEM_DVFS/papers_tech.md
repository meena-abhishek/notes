[NVHBM](https://blogs.nvidia.com/blog/nvlink-fusion-nvhbm-custom-high-bandwidth-memory/?preview_id=97957) - memory controller on hbm base die.
[thermal resistance in hbm](https://www.patsnap.com/resources/blog/articles/thermal-resistance-in-stacked-dram-hbm4-solutions/)
- HBM interface die exhibits the highest power density
- thermal hotspot in interface dies can lead to bit errors
- serial thermal resistance is dominated by microbumps (polymer-filled) and silicon bulk of each die - not by the interconnects.
- *polymer underfill*. It surrounds the microbumps and provides mechanical reinforcement, reducing stress on the solder/Cu connections during thermal cycling.
- conventional heatsink only addresses top of the stack and is structurally mismatched to where the heat is generated.
- lateral coupling from processors.
- use thermal TSVs that have low thermal resistance and can help get heat flow from bottom layers to heatsink.
- The TSV terminates into the die's BEOL metal layers. To connect that metal to another die, you need some form of die-to-die bonding.
- The TSV and the inter-die connection are optimized for different manufacturing problems. TSVs are made _inside_ the silicon wafer/die. Microbumps are made on the *surface* and allow two separately fabricated dies to be aligned and joined.
- TSV - cu_to_cu direct bond - TSV
- *A planar conductive layer* at the base of the stack converts point-source hotspots into a more uniform heat flux before routing to the cooling path.
- routing complexity increases when placing base layer on top.
- The per-group DDVFS approach surgically reduces performance only in the affected region.
- mapping data workloads to physical partitions based on each partition’s thermal characteristics, then optimising the DRAM refresh frequency per partition based on its operating temperature.
- This workload-aware approach is increasingly relevant as AI training and inference workloads create highly non-uniform memory access patterns.