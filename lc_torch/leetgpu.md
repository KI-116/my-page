## 模板代码

``` cpp
// naive：一个线程只负责一个元素
__global__ void add_kernel(const float* A, const float* B, float* C, int N) {
    // 1. 计算全局线程索引
    int idx = blockIdx.x * blockDim.x + threadIdx.x;

    // 2. 边界保护
    if (idx < N) {
        // 3. 逐元素计算
        C[idx] = A[idx] + B[idx];
    }
}
```
- `gridDim.x`：表示网格（grid）中块（block）的数量。
- `blockDim.x`：表示每个块中线程的数量。
- `blockIdx.x`：表示当前线程所在的块（block）的索引。
- `threadIdx.x`：表示当前线程在块内的索引。

在这个模板代码中，索引计算，边界检查、算子实现是我们通用不变的框架。如果要实现其他方案，例如ReLU、Sigmoid、Scale等，只需要替换序号3，其余代码完全不用动。

内存事务 Memory Transaction：GPU 的显存系统并非以单个字节或者单个浮点数为单位与核函数交互，而是以内存事务（Memory Transaction） 为单位。SM发出一次全局内存访问请求时，硬件会尝试将同一 Warp 的所有请求合并成尽可能少的事务。（通常访问的数据位于全局内存也就是 global memory 中的连续地址上）如果访问的数据不连续，SM就会发出多个内存事务，导致性能下降。

``` cpp

__global__ void add_kernel_v2(const float* A, const float* B, float* C, int N) {
    // 起始索引
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    // 总线程数（网格跨度）
    int stride = gridDim.x * blockDim.x;

    // 跨步循环处理多个元素
    for (int i = idx; i < N; i += stride) {
        C[i] = A[i] + B[i];
    }
}
```

启动配置并不需要覆盖整个N，而是可以让gridSize更小，这样每个线程就可以处理更多的数据。

``` cpp
int threadsPerBlock = 256;
int blocksPerGrid = 128 * 2;   // 故意只启动 SM 数量 × 若干倍，而非 N/256
add_kernel_v2<<<blocksPerGrid, threadsPerBlock>>>(d_A, d_B, d_C, N);

``` 

这种方案实际上已经成为了Element Wise核函数的事实标准，这样无论数据量多大，核函数都能够自适应，无需调整启动参数。

>  向量化加载与存储:可以在一次事务中批量进行加载数据。 CUDA 提供了内置的向量类型：float2、float4、double2 等。一个 float4 变量包含 4 个连续的 float，共 16 字节。使用它可以将 4 次 32-bit 访存合并为 1 次 128-bit 访存。

> `const float4* A4 = reinterpret_cast<const float4*>(A);  `

## information&init

- prop.name: GPU的名称
- prop.maxThreadsPerBlock: 每个block中最大线程数
- prop.maxThreadsDim: 每个block中每个维度的最大线程数
- prop.maxGridSize: 每个grid中每个维度的最大block数
- blocksPerGrid: grid中block的数量，计算方式为$$\text{blocksPerGrid} = \lceil \frac{N}{\text{threadsPerBlock}} \rceil$$

``` cpp
extern "C" void solve(const float* A, const float* B, float* C, int N) {
    cudaDeviceProp prop;
    cudaGetDeviceProperties(&prop, 0);

    printf("Device: %s\n", prop.name);
    printf("Max threads per block: %d\n", prop.maxThreadsPerBlock);
    printf("Max block dimensions: (%d, %d, %d)\n",
           prop.maxThreadsDim[0], prop.maxThreadsDim[1], prop.maxThreadsDim[2]);
    printf("Max grid size: (%d, %d, %d)\n",
           prop.maxGridSize[0], prop.maxGridSize[1], prop.maxGridSize[2]);

    int threadsPerBlock = 256;
    int blocksPerGrid = (N + threadsPerBlock - 1) / threadsPerBlock;

    vector_add<<<blocksPerGrid, threadsPerBlock>>>(A, B, C, N);
    cudaDeviceSynchronize();
}
```
## 逐元素算子

| 类别      | 算子名称                         | 数学表达式 / 描述                                                           | 典型应用场景                   |
| ------- | ---------------------------- | -------------------------------------------------------------------- | ------------------------ |
| 基础算术    | 1. 加法（Add）                   | $C[i]=A[i]+B[i]$                                                     | 特征融合、残差连接                |
| 基础算术    | 2. 减法（Sub）                   | $C[i]=A[i]-B[i]$                                                     | 误差计算、差分运算                |
| 基础算术    | 3. 乘法（Mul）                   | $C[i]=A[i]\times B[i]$                                               | 注意力掩码、逐通道缩放              |
| 基础算术    | 4. 除法（Div）                   | $C[i]=A[i]/B[i]$                                                     | 归一化中间步骤、比值计算             |
| 基础算术    | 5. 绝对值（Abs）                  | $y[i]=|x[i]|$                                                        | L1 Loss、距离计算             |
| 基础算术    | 6. 取反（Neg）                   | $y[i]=-x[i]$                                                         | 梯度反转层（GRL）、相位翻转          |
| 基础算术    | 7. 平方（Square）                | $y[i]=x[i]^2$                                                        | 均方误差（MSE）计算              |
| 基础算术    | 8. 平方根（Sqrt）                 | $y[i]=\sqrt{x[i]}$                                                   | 标准差计算、RMSNorm            |
| 基础算术    | 9. 幂运算（Pow）                  | $y[i]=x[i]^p$                                                        | Gamma 校正、指数变换            |
| 激活函数    | 10. ReLU                     | $y=\max(0,x)$                                                        | 深度神经网络标准激活函数             |
| 激活函数    | 11. Leaky ReLU               | $y=\max(\alpha x,x)$                                                 | 缓解神经元死亡问题、GAN            |
| 激活函数    | 12. Sigmoid                  | $y=\frac{1}{1+e^{-x}}$                                               | 二分类输出、门控机制               |
| 激活函数    | 13. Tanh                     | $y=\frac{e^x-e^{-x}}{e^x+e^{-x}}$                                    | RNN、LSTM 内部状态激活          |
| 激活函数    | 14. Swish / SiLU             | $y=x\cdot\sigma(x)$                                                  | EfficientNet、LLaMA       |
| 激活函数    | 15. GELU                     | $y=x\cdot\Phi(x)$（常用近似：$0.5x(1+\tanh(\sqrt{2/\pi}(x+0.044715x^3)))$） | Transformer（BERT、GPT）    |
| 激活函数    | 16. Hard Swish               | $y=x\cdot\frac{\text{ReLU6}(x+3)}{6}$                                | MobileNetV3              |
| 激活函数    | 17. ELU                      | $y=\begin{cases}x,&x>0\\alpha(e^x-1),&x\le0\end{cases}$              | 加速收敛、均值接近零               |
| 激活函数    | 18. Softplus                 | $y=\ln(1+e^x)$                                                       | ReLU 平滑近似                |
| 裁剪与归一化  | 19. Clip / Clamp             | $y=\min(\max(x,\text{min}),\text{max})$                              | 梯度裁剪、像素范围约束              |
| 裁剪与归一化  | 20. 最大值（Max）                 | $C[i]=\max(A[i],B[i])$                                               | 最大池化、ReLU 变体             |
| 裁剪与归一化  | 21. 最小值（Min）                 | $C[i]=\min(A[i],B[i])$                                               | 距离场计算、对偶操作               |
| 裁剪与归一化  | 22. 舍入（Round / Ceil / Floor） | $y=\mathrm{round}(x)$、$\lfloor x\rfloor$、$\lceil x\rceil$            | QAT 伪量化、离散化              |
| 特殊函数与图像 | 23. 指数运算（Exp）                | $y[i]=e^{x[i]}$                                                      | Softmax、中间计算             |
| 特殊函数与图像 | 24. 对数运算（Log）                | $y[i]=\ln(x[i])$                                                     | 交叉熵、对数似然                 |
| 特殊函数与图像 | 25. 倒数（Reciprocal）           | $y[i]=1/x[i]$                                                        | 除法优化、调和平均                |
| 特殊函数与图像 | 26. 符号函数（Sign）               | $y[i]=\operatorname{sgn}(x[i])$                                      | 二值神经网络（BNN）              |
| 特殊函数与图像 | 27. 数值比较（Eq / Ne / Gt / Lt）  | $y[i]=(A[i],\text{op},B[i])$                                         | 布尔掩码生成、准确率统计             |
| 特殊函数与图像 | 28. RGB 转灰度                  | $Y=0.299R+0.587G+0.114B$                                             | 图像预处理、单通道特征提取            |
| 特殊函数与图像 | 29. 颜色反转                     | $y=255-x$                                                            | 图像负片、数据增强                |
| 特殊函数与图像 | 30. Dropout                  | $y=\dfrac{x\cdot\text{mask}}{1-p}$                                   | 神经网络正则化                  |
| 特殊函数与图像 | 31. 缩放与偏置（Scale & Bias）      | $y=\gamma x+\beta$                                                   | BatchNorm、LayerNorm 仿射变换 |
