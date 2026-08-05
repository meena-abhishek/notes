memory-induced CPU throttling

major bottleneck in implementing inference on CPUs is the saturation of limited off-chip memory bandwidth. Typically, DNNs exhibit large memory footprints due to large amounts of dynamically generated intermediate data in each network layer. A DNN is composed of a series of alternating convolutional and pooling layers, followed by fully connected layers toward the end. Each layer generates hundreds of megabytes of output data which serves as input to the next layer. The enormous memory footprint of these workloads quickly saturates the memory bandwidth of conventional DRAMs, causing processors to stall. The frequent stalling of cores waiting for data from DRAM degrades system performance and increases inference time.

utilizing application knowledge for making thermal management decisions.

Our proposed prefetching mechanism dynamically computes the best prefetch settings for different DNN workload phases.