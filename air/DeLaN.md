# 数据准备
## 数据收集
## 数据预处理
## 数据加载

# 定义模型


``` python
class LagrangianLayer(nn.Module):

    def __init__(self, input_size, n_dof, activation="ReLu"):
        super(LagrangianLayer, self).__init__()

        # Create layer weights and biases:
        self.n_dof = n_dof
        self.weight = nn.Parameter(torch.Tensor(n_dof, input_size))
        self.bias = nn.Parameter(torch.Tensor(n_dof))

        # Initialize activation function and its derivative:
        if activation == "ReLu":
            self.g = nn.ReLU()
            self.g_prime = ReLUDer()

        elif activation == "SoftPlus":
            self.softplus_beta = 1.0
            self.g = nn.Softplus(beta=self.softplus_beta)
            self.g_prime = SoftplusDer(beta=self.softplus_beta)

        elif activation == "Cos":
            self.g = Cos()
            self.g_prime = CosDer()

        elif activation == "Linear":
            self.g = Linear()
            self.g_prime = LinearDer()

        else:
            raise ValueError("Activation Type must be in ['Linear', 'ReLu', 'SoftPlus', 'Cos'] but is {0}".format(self.activation))

    def forward(self, q, der_prev):
        # Apply Affine Transformation:
        a = F.linear(q, self.weight, self.bias)
        out = self.g(a)
        der = torch.matmul(self.g_prime(a).view(-1, self.n_dof, 1) * self.weight, der_prev)
        return out, der
```
## 工具类

`LowTri`(下三角矩阵构造器)
- `__init__(m)`:预计算 `m×m` 下三角矩阵的索引。
- `__call__(l)`:把一维向量 `l` 填充成批量下三角矩阵 `[batch, m, m]`。



二、激活函数及其导数(用于解析求导)

| 类 | 功能 |
|---|---|
| `SoftplusDer` | Softplus 的导数(即 sigmoid),含数值裁剪防溢出 |
| `ReLUDer` | ReLU 的导数(0/1 阶跃) |
| `Linear` | 线性激活 `f(x) = x` |
| `LinearDer` | 线性激活的导数(恒为 1) |
| `Cos` | 余弦激活 `cos(x)` |
| `CosDer` | 余弦的导数 `-sin(x)` |

每个激活都成对出现,目的是在前向传播时同步解析地计算 ∂y/∂q,避免使用 autograd。

---

三、`LagrangianLayer`(核心自定义层)

- `__init__(input_size, n_dof, activation)`:创建权重/偏置,选择激活函数(ReLu / SoftPlus / Cos / Linear)。
- `forward(q, der_prev)`:
  - 计算 `a = Wq + b`,输出 `out = g(a)`;
  - 同时通过链式法则更新雅可比 `der = g'(a) · W · der_prev`,把对输入 `q` 的导数一路传下去。

---

四、`DeepLagrangianNetwork`(主网络)

初始化 `__init__(n_dof, **kwargs)`
- 读取超参数:网络宽度、深度、初始化方式、激活函数、对角正则 ε 等。
- 定义三种权重初始化器:`xavier_normal` / `orthogonal` / `sparse`。
- 计算下三角矩阵 L 的对角元素索引和非对角索引,用于把两个网络头的输出拼成正确顺序。
- 构建网络:
  - 共享主干:多层 `LagrangianLayer`;
  - `net_g`:输出势能 V(标量,线性激活);
  - `net_lo`:输出 L 的下三角(非对角)元素;
  - `net_ld`:输出 L 的对角元素(ReLU 保证非负,确保 H 正定)。

前向与动力学方法

| 方法 | 功能 |
|---|---|
| `forward(q, qd, qdd)` | 返回预测力矩 `tau_pred` 和总能量变化率 `dE/dt` |
| `_dyn_model(q, qd, qdd)` | 核心:计算 `tau, H, c, g, T, V, dT/dt, dV/dt` |
| `inv_dyn(q, qd, qdd)` | 逆动力学:已知状态求力矩 τ |
| `for_dyn(q, qd, tau)` | 正动力学:`qdd = H⁻¹(τ - c - g)` |
| `energy(q, qd)` | 总机械能 `E = T + V` |
| `energy_dot(q, qd, qdd)` | 能量变化率 `dE/dt` |

`_dyn_model` 内部计算的物理量
- H(q):惯性矩阵,通过 `L·Lᵀ + εI` 的 Cholesky 形式保证正定;
- dH/dt 和 dH/dq:基于雅可比 `der_l` 解析求得;
- c(q,q̇):科氏力 + 离心力,`c = Ḣq̇ - ½ ∂(q̇ᵀHq̇)/∂q`;
- g(q):重力项,等于 `∂V/∂q`;
- τ = H·q̈ + c + g:拉格朗日逆动力学方程;
- T:动能 `½ q̇ᵀHq̇`;
- dT/dt, dV/dt:能量变化率。

设备迁移
- `cuda(device)` / `cpu()`:重写以同时移动网络参数和缓存的单位矩阵 `_eye`。

---

总体设计思想

1. 物理结构嵌入网络:不直接回归 τ,而是显式参数化 H(q) 和 V(q);
2. 解析雅可比传播:`LagrangianLayer` 在前向时手动维护 ∂·/∂q,避免对 q 二阶 autograd;
3. 正定性保证:用 Cholesky 分解 H = LLᵀ + εI;
4. 能量一致性:可输出 dE/dt 作为额外训练损失(能量守恒约束)。

---

将摩擦融入能量守恒（耗散）方案

理想双摆满足能量守恒 $\frac{dE}{dt}=0$,其中 $E=T+V$。一旦引入 Stribeck 摩擦,系统就变成耗散系统,需要把"守恒律"改写为"能量平衡律":

$$
\frac{dE}{dt} \;=\; P_{\text{ctrl}} \;+\; P_{f}
$$

其中 $P_{\text{ctrl}}=\dot q^{\top}\tau_{\text{ctrl}}$ 是控制输入功率,$P_f=\dot q^{\top}\tau_f \le 0$ 是摩擦耗散功率。下面给出几种把摩擦"自洽地"嵌入能量框架的方案。

---

1. 增广能量方程(Energy Balance Form)

把原始 Euler–Lagrange 方程

$$
M(q)\ddot q + C(q,\dot q)\dot q + G(q) = \tau_{\text{ctrl}} + \tau_f
$$

两边左乘 $\dot q^{\top}$,利用 $\dot M - 2C$ 反对称的性质,可得:

$$
\boxed{\;\frac{d}{dt}\underbrace{\Big[\tfrac12\dot q^{\top}M(q)\dot q + V(q)\Big]}{E(q,\dot q)} \;=\; \dot q^{\top}\tau{\text{ctrl}} \;-\; \underbrace{\dot q^{\top}\big[D(\dot q)\dot q + \tau_c\!\odot\!\text{sgn}(\dot q) + \tau_s\!\odot\!\text{sgn}(\dot q)\,e^{-\dot q^2/\nu}\big]}_{\mathcal D(\dot q)\;\ge 0}\;}
$$

$\mathcal D(\dot q)\ge 0$ 即瑞利耗散函数(广义形式)。这是把摩擦写进能量法的标准形式:能量不再守恒,但 $E(t)-\!\int_0^t P_{\text{ctrl}}\,d\tau = -\!\int_0^t \mathcal D\,d\tau$ 是单调下降的"伪守恒量"。

---

2. Rayleigh Dissipation

Viscous friction

$$
\mathcal R_v(\dot q) = \tfrac12 \dot q^{\top}\text{diag}(d)\dot q,\qquad \tau_{f,v} = -\frac{\partial \mathcal R_v}{\partial \dot q}
$$

stribeck

$$
\mathcal R_c(\dot q) = \sum_i \tau_{c,i}|\dot q_i| + \tau_{s,i}\!\int_0^{\dot q_i}\!\text{sgn}(s)\,e^{-s^2/\nu_i}\,ds
$$

Lagrange genéralized

$$
\frac{d}{dt}\frac{\partial L}{\partial \dot q} - \frac{\partial L}{\partial q} + \frac{\partial \mathcal R}{\partial \dot q} = \tau_{\text{ctrl}},\quad \mathcal R = \mathcal R_v+\mathcal R_c
$$



瑞利耗散介导的τ摩擦计算

friction calculation with Rayleigh dissipation













---

3. DeLaN


| 物理量 | 网络输出 | 正定性保证 |
|---|---|---|
| $H(q)$ | `L(q)` → $H=LL^\top+\epsilon I$ | Cholesky |
| $V(q)$ | 标量网络 | — |
| $\mathcal R(\dot q)$ | 标量网络,输入 $\dot q$,输出 $\ge0$ | softplus 输出 |
| $\tau_f$ | $-\partial \mathcal R/\partial \dot q$(autograd) | 自动耗散 |

损失里再加一项能量一致性损失:

$$
\mathcal L_{\text{energy}} = \Big\|\frac{dE}{dt} - \dot q^{\top}\tau_{\text{ctrl}} + \dot q^{\top}\frac{\partial \mathcal R}{\partial \dot q}\Big\|^2
$$

这样网络在学逆动力学的同时,强制满足"机械能减少量 = 摩擦耗散积分",既保留了 Lagrangian 的物理结构,又把非保守摩擦纳入统一框架。

---

4. 数值积分时的注意点

- $\text{sgn}(\dot q)$ 在 $\dot q=0$ 处不连续 → 仿真用 $\tanh(\dot q/\varepsilon)$ 平滑,避免刚性 ODE 求解器在零速时震荡(极限环假象)。
- 平滑化后,$\mathcal R$ 变 $C^1$,可直接 autograd,与 DeLaN 的解析雅可比传播兼容。
- 死区效应可在仿真中用 LCP / 互补条件 处理(更物理,但不可微);若要可微训练,优先用平滑 Stribeck。

---

一句话总结

> 摩擦不破坏能量法,只是把"守恒"升级为"平衡":引入瑞利耗散势 $\mathcal R(\dot q)$,让 $\dot E = P_{\text{ctrl}} - \dot q^\top\partial_{\dot q}\mathcal R$,网络里再把 $\mathcal R$ 当作一个与 $H,V$ 平级的正定标量场学习,即可在 DeLaN 中自洽地嵌入 Stribeck 摩擦。
