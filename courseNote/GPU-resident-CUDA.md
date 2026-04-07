# slide1

This slide covers GPU-resident CUDA optimization for LLM inference.  

The key idea is moving computation onto the GPU, minimizing costly CPU-GPU data transfers.  //TODO:

For a compute-intensive operation, all functions it depends on are co-migrated to the GPU as well as model weights. This naturally leads to a fully GPU-resident Forward pipeline. 

The diagram shows the forward pass flow. Initiates by calling Forward for the next token in generate() or chat() on the CPU, then hands off to the GPU. After copying embedding to d_x using weights stored on GPU, 
computation flows bottom-up through N repeated transformer layers: `RMSNorm → QKV Matmul → RoPE & KV Cache → Attention Scores → Softmax → Weighted Sum → Output Projection → RMSNorm → FFN → FFN Output Projection`. After all layers and final normalization, the raw logits are sent back to the CPU for sampling.

Four main changes help implement this: First, model weights are copied to GPU memory upfront. Second, runtime states and the Transformer struct are updated to live on the GPU. Third, a KV Cache is integrated directly on the GPU, avoiding repeated recomputation. Fourth, matrix multiplications are replaced with cuBLAS for hardware-optimized performance.


 matching the natural parallelism structure of the operation.

# slide2

## 完整讲稿（上页 + 结果页）｜~2分钟

---

**English**

This section covers our GPU-resident CUDA optimization and its results.

The core idea is simple: keep everything on the GPU. Weights are loaded to GPU memory upfront, the KV Cache lives on the GPU, and the entire forward pass executes without touching the CPU — only the final logits come back for sampling.

To make this work, we restructured the forward function into three kernel types matched to three levels of parallelism: Multi-Head kernels parallelize across attention heads for QKV matmul; Reduction kernels parallelize within a vector for Softmax and RMSNorm; and Elementwise kernels parallelize across individual elements for residual additions. Each kernel targets the natural parallelism of its operation.

Now for the results, these two pictures are taken from our report.
Tested on a single H100 with step 256. Stories15M gets over 2000 tokens/second, and Stories42M around 1300. For the production-scale LLaMA2-7B, we achieve roughly 89–94 tokens/second.

However, the Roofline analysis tells an important story. Our arithmetic intensity is only 0.48 FLOP/byte for the smallest model, far below both the double and single precision rooflines. 

We're achieving 83 GFLOP/s against a theoretical ceiling of 27,000. This means the implementation is **memory-bandwidth bound**, not compute bound — the bottleneck is data movement, not computation. This points to the next optimization direction: techniques like quantization or operator fusion to increase arithmetic intensity.

---

**中文**

这部分介绍我们的 GPU 常驻 CUDA 优化方案及实验结果。

核心思路是：让一切计算留在 GPU 上。权重预先加载到显存，KV Cache 也常驻 GPU，整个前向传播无需回到 CPU——仅最终 logits 回传采样。

为此，我们将前向函数重构为三类 kernel，对应三个并行层级：多头 kernel 在注意力头维度并行，处理 QKV 矩阵乘；Reduction kernel 在向量内部并行，处理 Softmax 与 RMSNorm；Elementwise kernel 在元素维度并行，处理残差加法。每类 kernel 精准匹配其操作的并行结构。

来看结果，测试环境为单张 H100，batch size 256。小模型速度很快——Stories15M 超过 2000 tokens/秒，Stories42M 约 1300。生产级别的 LLaMA2-7B 达到约 89–94 tokens/秒，单卡吞吐表现良好。

但 Roofline 分析揭示了一个关键问题。我们的算术强度仅为 0.48 FLOP/byte，远低于双精度和单精度的理论上限。实际达到 83 GFLOP/s，而理论峰值高达 27,000。这说明当前实现是**内存带宽瓶颈**，而非算力瓶颈——数据搬运才是真正的瓶颈所在。这也指明了下一步优化方向：通过量化或算子融合来提升算术强度。



