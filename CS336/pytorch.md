# 

## cal
Question: How long would it take to train a 70B parameter model on 15T tokens on 1024 H100s?
    total_flops = 6 * 70e9 * 15e12  # @inspect total_flops
    assert h100_flop_per_sec == 1979e12 / 2
    mfu = 0.5
    flops_per_day = h100_flop_per_sec * mfu * 1024 * 60 * 60 * 24  # @inspect flops_per_day
    days = total_flops / flops_per_day  # @inspect days

Question: What's the largest model that can you can train on 8 H100s using AdamW (naively)?
    h100_bytes = 80e9  # @inspect h100_bytes
    bytes_per_parameter = 4 + 4 + (4 + 4)  # parameters, gradients, optimizer state  @inspect bytes_per_parameter
    num_parameters = (h100_bytes * 8) / bytes_per_parameter  # @inspect num_parameters

    Caveat 1: we are naively using float32 for parameters and gradients. We could also use bf16 for parameters and gradients (2 + 2) and keep an extra float32 copy of the parameters (4). This doesn't save memory, but is faster.  [Rajbhandari+ 2019]
    Caveat 2: activations are not accounted for (depends on batch size and sequence length).
    This is a rough back-of-the-envelope calculation.

## tensors:

**FP16：** 
不适合用来表示太大或太小的数值，容易出现溢出或下溢问题。
如果用它来计算大型模型，可能会导致数值不稳定。
signe bit: 1 bit
exponent bits: 5 bits
mantissa bits: 10 bits

**bfloat16:**
signe bit: 1 bit
exponent bits: 8 bits
mantissa bits: 7 bits

可以看出使用了更多的指数位，可以表示更大的数值范围，适合用于训练大型模型。

resolution表现会更差，因为只有7位的尾数位，导致精度较低。但大型模型通常没有太多精度要求。

**FP32:**
对于storing optimizer 和 parameters, 仍然需要FP32

storing optimizer: 需要存储每个参数的动量和二阶矩估计，这些值可能会有较大的范围，因此需要使用FP32来存储以避免数值不稳定。

parameters:参数更新


FP8 被Nvidia提出，使用4位指数和3位尾数，可以进一步减少内存占用和计算成本，但精度更低，适用于H100

Implications on training:
- Training with float32 works, but requires lots of memory.
- Training with fp8, float16 and even bfloat16 is risky, and you can get instability.
- Solution (later): use mixed precision training, see  mixed_precision_training

例如，对attention使用FP32，但对涉及矩阵乘法的简单前向传播，使用BF16。

tensors: 指向内存的指针，包含数据类型和形状信息。

contiguous tensors: 在内存中连续存储的张量，可以更高效地进行计算。
如果要对tensor进行切片或转置等操作，可能会导致非连续存储，这时需要调用.contiguous()方法来确保张量在内存中是连续的。
``` python
y = x.transpose(1,0).contiguous().view(2,3)# 将x转置后变为非连续存储的张量，再调用.contiguous()方法使其变为连续存储的张量，最后使用.view()方法改变其形状。
```
