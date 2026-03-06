# profiling anylsis: Nsight

## nsight GPU
Nsight Systems 是一款系统级性能分析工具，提供了对 GPU 和 CPU 之间交互的深入洞察。在本项目中，我们使用 Nsight Systems 来分析 GPU 的性能表现, CPU-GPU data transfer 时间等。

Nsight Systems is a system-level performance analysis tool that provides insights into the interactions between the GPU and CPU. In this project, we use Nsight Systems to analyze the performance of the GPU, including GPU utilization, memory usage, and CPU-GPU data transfer times.


## Nsight roofline

[CN]
Roofline 模型是一种性能分析工具，用于评估程序对可用硬件资源的利用情况。
传统的Roofline模型依靠两个特征来描述工作负载：
运算强度：计算FLOPs与数据传输量（字节）的比率。
FLOP/s:每秒执行的浮点运算次数。

Nsight Compute 是一款 CUDA 内核分析器，可提供详细的性能测量。在本项目中，我们从命令行开始性能分析。使用标志 --section SpeedofLight_RooflineChart, 得到以下结果。

这是在单张A40上对42M模型进行的测试。

对图片X，

针对主要 computation kernel 的剖析结果显示，该算子的运行状态点位于 Roofline 模型左侧的带宽受限区（Memory-Bound Region）算术强度远低于硬件转折点（Ridge Point,每加载 1 字节数据所支撑的浮点运算不足 0.5 次.

实测观测到的 DRAM 吞吐量约为 15 GB/s。根据 Roofline 理论公式计算，完成当前 84 GFLOPS 计算任务所需的逻辑有效带宽约为175 GB/s（约占 A40 峰值带宽 696 GB/s 的 25%）。远高于实际观测到的 DRAM 吞吐量，说明大量的数据交换发生在 L2 缓存 或指令寄存器中，并未回落至物理显存（DRAM）。尽管如此，仍可以通过增加batch size来提升算术强度，进一步提升性能。



[EN]
The Roofline performance model is a visual performance model, which helps to ubderstand the performance of a program on the available hardware resources and identifying potential bottlenecks. 

The traditional Roofline model relies on two characteristics to describe the workload:

Arithmetic intensity: The ratio between compute work (FLOPs) and data movement (bytes)
FLOP/s: Floating-point operations per second

Nsight Compute is a CUDA kernel profiler that provides detailed performance measurements. In this project, we start the performance analysis from the command line. Using the flag --section SpeedofLight_RooflineChart, we get the following results.

This is the test on a single A40 for the 42M model.

The analysis results for the main computation kernel matmul show that the operating point of the operator is located in the memory-bound region on the left side of the Roofline model, with an arithmetic intensity far below the hardware ridge point (less than 0.5 FLOPs per byte loaded).

The observed DRAM throughput is about 15 GB/s. According to the Roofline theoretical formula, the required effective bandwidth to complete the current 84 GFLOPS compute task is about 175 GB/s (about 25% of A40's peak bandwidth of 696 GB/s). This is much higher than the observed DRAM throughput, indicating that a large amount of data exchange occurs in the L2 cache or register file, and does not fall back to physical memory (DRAM). Nevertheless, there is still room for performance improvement by increasing the batch size to increase arithmetic intensity.




























# GPU-Resident Transformer Inference with CUDA

该CUDA优化也是本项目中toks表现最好的版本，主要优化点在于：
重构forward函数及其调用，将forward的计算完全放在GPU上进行，仅仅是最终的logits结果传回CPU进行后续sample等处理。为了实现这一点，模型的权重全部上传到GPU，避免CPU-GPU之间的频繁数据传输。并且构建和分配属于GPU 的 Runtime Buffer。本实现也集成了KVCache优化，在VRAM上分配了K Cache 和V Cache，避免了每次forward都要重新计算K和V。

此外在kernel设计上，针对Transformer的计算特点，调用cuBLAS库进行矩阵乘法计算，利用其高度优化的性能。同时，针对框架中的其他算子，设计了专门的CUDA kernel来加速计算：RoPE+KVCache 融合成一个rope_and_cache_kernel。 softmax用一个block处理一个head,取消forward中的循环，直接在kernel中处理所有head的softmax计算。同时将attention score循环独立成算子，时间步由线程索引控制。
rmsnorm 和 softmax 都用了 shared memory 做 reduction。

当然有进一步优化的空间也就是使用CUDA stream来并行处理n_layers的计算。 此外还有增加批处理和warp shuffle等优化手段。


[EN]
This CUDA optimization is also the best performing version of toks in this project. The main optimization point is to restructure the forward function and its calls, placing all computations of the forward function on the GPU, with only the final logits results being transferred back to the CPU for subsequent sampling and other processing. To achieve this, all model weights are uploaded to the GPU, avoiding frequent data transfers between the CPU and GPU. Additionally, Runtime States belonging to the GPU are constructed and allocated. This implementation also integrates KVCache optimization, allocating K Cache and V Cache on VRAM, avoiding the need to recompute K and V for each forward pass.

In terms of kernel design, we call the cuBLAS library for matrix multiplication calculations, leveraging its highly optimized performance. For other operators in the framework, we designed dedicated CUDA kernels to accelerate computation: 

RoPE and KVCache are fused into a rope_and_cache_kernel. 

The softmax is processed with one block per head, eliminating loops in the forward function and directly handling all heads' softmax in the kernel.

The attention score loop is also separated into an operator, with time steps controlled by thread indices. 

Both rmsnorm and softmax use shared memory for reduction.

Of course, there is still room for further optimization, such as using CUDA streams to parallelize the computation of n_layers. Additionally, other optimization techniques such as increasing batch processing and warp shuffle can be explored.


# llama benchmark




# kvcache (待定)

# reference
@article{10.1145/1498765.1498785,
author = {Williams, Samuel and Waterman, Andrew and Patterson, David},
title = {Roofline: an insightful visual performance model for multicore architectures},
year = {2009},
issue_date = {April 2009},
publisher = {Association for Computing Machinery},
address = {New York, NY, USA},
volume = {52},
number = {4},
issn = {0001-0782},
url = {https://doi.org/10.1145/1498765.1498785},
doi = {10.1145/1498765.1498785},
abstract = {The Roofline model offers insight on how to improve the performance of software and hardware.},
journal = {Commun. ACM},
month = apr,
pages = {65–76},
numpages = {12}
}