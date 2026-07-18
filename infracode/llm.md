# 一、PyTorch 高频语法

## 1. Tensor创建

```python
import torch

x = torch.tensor([1,2,3])
x = torch.randn(2,3)
x = torch.zeros(2,3)
x = torch.ones(2,3)

torch.arange(10)
torch.arange(0, 10, 2)

torch.eye(4)
```

---

## 2. Shape操作

### reshape

```python
x = torch.randn(2,3,4)

x.view(6,4)
x.reshape(6,4)
```

### transpose

```python
x.transpose(1,2)
```

Transformer中极其常见：

```python
Q = Q.view(B,S,H,d_k).transpose(1,2)

# (B,S,H,d_k)
# ->
# (B,H,S,d_k)
```

MHA中大量使用。

---

### contiguous

```python
x = x.transpose(1,2)

x = x.contiguous().view(...)
```

因为 transpose 后内存不连续。

---

### unsqueeze

增加维度

```python
x = torch.randn(10)

x.unsqueeze(0)
# (1,10)

x.unsqueeze(1)
# (10,1)
```

位置编码：

```python
freqs.unsqueeze(0).unsqueeze(2)
```



---

### squeeze

删除长度为1的维度

```python
x.squeeze()
```

---

## 3. 拼接

### cat

```python
torch.cat([a,b], dim=0)
```

KV Cache：

```python
K = torch.cat([K_prev, K], dim=2)
V = torch.cat([V_prev, V], dim=2)
```



---

### stack

新增维度

```python
torch.stack([a,b], dim=0)
```

---

# 二、数学计算常用API

## softmax

```python
F.softmax(x, dim=-1)
```



---

## sigmoid

```python
torch.sigmoid(x)
```

SiLU:

```python
x * torch.sigmoid(x)
```



---

## gelu

```python
F.gelu(x)
```



---

## logsigmoid

DPO损失

```python
F.logsigmoid(x)
```



---

## mean

```python
x.mean(dim=-1)
```

LayerNorm:

```python
mu = x.mean(dim=-1, keepdim=True)
```



---

## var

```python
x.var(
    dim=-1,
    keepdim=True,
    unbiased=False
)
```



---

## sqrt

```python
torch.sqrt(x)
```

LayerNorm/RMSNorm



---

## pow

```python
x.pow(2)
```

RMSNorm：

```python
x.pow(2).mean(...)
```



---

# 三、矩阵运算（面试最常考）

## matmul

```python
torch.matmul(A,B)
```

Attention核心：

```python
scores = torch.matmul(
    Q,
    K.transpose(-2,-1)
)
```



---

## @ 运算符

```python
A @ B
```

LoRA：

```python
x @ A.T @ B.T
```



---

## transpose(-2,-1)

Attention必背

```python
K.transpose(-2,-1)
```

shape:

```python
(B,H,S,d)
→
(B,H,d,S)
```

---

# 四、Mask相关

## masked_fill

```python
scores.masked_fill(
    mask == 0,
    float("-inf")
)
```

Attention标准写法。

---

## tril

生成下三角Mask

```python
torch.tril(
    torch.ones(S,S)
)
```



---

# 五、排序与采样

## topk

```python
values, indices = torch.topk(
    logits,
    k
)
```

Top-K Sampling



---

## sort

```python
sorted_logits, sorted_idx = torch.sort(
    logits,
    descending=True
)
```

Top-P Sampling



---

## cumsum

```python
torch.cumsum(
    probs,
    dim=-1
)
```

Top-P累计概率



---

## multinomial

```python
torch.multinomial(
    probs,
    num_samples=1
)
```

从概率分布采样token



---

# 六、神经网络模块（必须会写）

## nn.Linear

```python
nn.Linear(
    in_features,
    out_features
)
```

用于：

```python
W_q
W_k
W_v
W_o
```



---

## nn.Parameter

定义可训练参数

```python
self.gamma = nn.Parameter(
    torch.ones(d_model)
)
```

LayerNorm/RMSNorm/LoRA

---

## nn.Module

所有模块继承

```python
class MyModule(nn.Module):
    def __init__(self):
        super().__init__()

    def forward(self,x):
        return x
```

---

# 七、Transformer核心算法（面试最高频）

## 1. LayerNorm

核心：

```python
mu = x.mean(...)
var = x.var(...)

x_norm = (x - mu) / sqrt(var)
```



---

## 2. RMSNorm

核心：

```python
rms = torch.sqrt(
    x.pow(2).mean(...)
)

x = x / rms
```



---

## 3. Scaled Dot Product Attention

```python
scores = Q @ K.T / sqrt(d_k)

weights = softmax(scores)

output = weights @ V
```



---

## 4. Multi-Head Attention

面试标准流程：

```python
1. Linear(QKV)

2. view拆头

3. transpose

4. Attention

5. transpose

6. view合并头

7. W_o
```



---

## 5. GQA

关键点：

```python
K.repeat_interleave(...)
V.repeat_interleave(...)
```

扩展KV头



---

## 6. RoPE

关键API：

```python
torch.polar()

torch.view_as_complex()

torch.view_as_real()
```

---

## 7. SwiGLU

```python
F.silu(
    gate(x)
) * up(x)
```



---

## 8. Transformer Block

标准结构：

```python
x
↓
RMSNorm
↓
MHA
↓
Residual

↓
RMSNorm
↓
FFN
↓
Residual
```

LLaMA/Qwen结构。

---

## 9. KV Cache

核心：

```python
历史KV缓存

K = cat(K_prev, K)

V = cat(V_prev, V)
```

复杂度：

```python
O(n²d)
↓
O(nd)
```

---

## 10. LoRA

核心公式：

```python
W = W0 + BA
```

实现：

```python
base_out = linear(x)

lora_out =
x @ A.T @ B.T

return base_out + lora_out
```

---

# 八、大模型面试必须背的 Shape

```python
Embedding
(B,S)
→
(B,S,D)

MHA
(B,S,D)
→
(B,H,S,d_k)
→
(B,S,D)

RoPE
(B,S,H,d_k)
→
(B,S,H,d_k)

FFN
(B,S,D)
→
(B,S,d_ff)
→
(B,S,D)

KV Cache
(B,1,D)
→
KV:(B,H,S,d_k)
```



---

建议优先背熟下面这 15 个 PyTorch API：

```python
nn.Linear
nn.Parameter

torch.matmul
torch.cat
torch.stack
torch.topk
torch.sort
torch.cumsum
torch.multinomial
torch.tril

tensor.view
tensor.reshape
tensor.transpose
tensor.unsqueeze

F.softmax
F.cross_entropy
F.silu
F.gelu
F.logsigmoid
```

这套基本覆盖了 Transformer、LoRA、DPO、推理加速等 90% 以上的大模型手撕代码题。
