1. TP & PP

- TP (Tensor Parallelism)： 是算子级的并行。它将 Transformer 内部的矩阵运算（如 MLP 层和 Self-Attention 的权重矩阵）拆分到不同 GPU 上。每一张 GPU 只负责矩阵计算的一部分，计算完成后通过 All-Reduce 等通信原语汇总结果。
    - 拆分方式：
- PP (Pipeline Parallelism)： 是层级的并行。它将整个模型的计算层按照顺序划分为多个 Stage（阶段），每个 Stage 包含一定数量的 Transformer 层。模型像流水线一样运行，GPU 之间通过 Point-to-Point（点对点）方式传递中间层的激活值。

2. TTFT(Time To First Token)

即用户发送请求后，模型生成出第一个字（Token）所花费的时间。它是衡量大模型响应速度最核心的指标，直接决定了用户的感知体验。

- 预填充阶段 (Prefill/Prompt Processing)： 模型需要先“读懂”用户的输入（Prompt）。这是计算密集型阶段，GPU 需要并行处理输入的所有 Token，即计算输入Token的kv矩阵，并将结果存储在 GPU 内存中。
- 影响 TTFT 的关键因素
    - 计算能力： 增加 GPU 的算力，或者通过 TP（张量并行） 将计算任务分摊，可以加速预填充阶段的处理。
    - Continuous Batching (连续批处理)： 传统的静态 Batch 会导致新请求必须等前一批处理完才能进入。连续批处理允许新请求在旧请求生成过程中“插队”或并行，从而大幅降低等待时间。

3. 投机采样 (Speculative Decoding)

LLM 的推理本质上是一个串行自回归（Autoregressive）过程：每生成一个 Token，都需要把整个模型跑一遍。

投机采样的核心思想是：用一个更小、更快的“草稿模型”（Draft Model）去“猜测”接下来要生成的几个 Token，然后用大模型（Target Model）进行一次性验证。

>流程：
>a. 草稿阶段 (Drafting)： 使用一个轻量级模型（或简单的启发式算法）快速生成 $k$ 个 Token。
>b. 验证阶段 (Verification)： 将这 $k$ 个 Token 一次性喂给大模型（Target Model）。
>c. 大模型并行计算这 $k$ 个位置的概率分布。
>d. 如果大模型的预测结果与草稿模型匹配，则一次性接受多个 Token；如果有 Token 不匹配，则丢弃错误部分，保留正确的前缀，并由大模型重新生成接下来的内容。

4. 