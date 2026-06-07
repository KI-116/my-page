演讲讲稿 / Presentation Script (中英对照)



---

Slide 1 — DeLaN Model

中文:
这一页介绍 DeLaN(深度拉格朗日网络)的基本架构。图中左右两个神经网络分别用于学习系统的动能 T 和势能 V。动能网络的输出通过一个正定矩阵 H 构造为 T = ½ q̇ᵀ H q̇,势能 V 则由另一个网络直接输出。
得到 T 和 V 之后,我们就能定义拉格朗日函数 L = T − V,以及总能量 H = T + V。基于拉格朗日力学的欧拉–拉格朗日方程 d/dt(∂L/∂q̇) − ∂L/∂q − τ = 0,可以反推出电机力矩 τ。同时能量守恒方程 dH/dt − q̇ᵀτ = 0 为训练提供了一个物理一致性约束。
网络参数通过最小化预测力矩与测量力矩之间的残差来学习。这正是 Lutter 等人在 2019 年提出的原始 DeLaN 架构。
基于此架构,我们提出了一个改进版本——引入一个"摩擦网络",用于学习并补偿系统中存在的摩擦力。

The kinetic energy is constructed via a positive-definite matrix H as T = ½ q̇ᵀ H q̇, while V is output directly by the second network.

**English:**
This slide introduces the original architecture of DeLaN — the Deep Lagrangian Network. The two neural networks on the left and right learn the system's kinetic energy T and potential energy V respectively. 
With T and V, we define the Lagrangian L = T − V and the total energy H = T + V. Applying the Euler–Lagrange equation d/dt(∂L/∂q̇) − ∂L/∂q − τ = 0 lets us recover the motor torque τ, while the energy-conservation equation dH/dt − q̇ᵀτ = 0 provides a physically consistent training constraint.

The network parameters are learned by minimizing the residual between predicted and measured torques — this is the original DeLaN architecture proposed by Lutter et al. (2019).

Building on it, we propose a modified version that introduces an additional friction network to learn and compensate the friction forces in the system.

---

Slide 2 — DeLaN Model with Friction Compensation

中文:
这一页展示了我们提出的、带摩擦补偿的 DeLaN 模型架构。整体输出力矩被分解为两部分:τ̂ = τ̂_cons + τ_f(q̇),其中 τ̂_cons 是由原 DeLaN 学到的保守力部分,τ_f(q̇) 是新增的摩擦力部分。
看主图:输入是关节位置 q、速度 q̇ 和加速度 q̈。q 经过一个以 sine 为激活函数的 MLP 主干网络,分支出三个线性头:g 直接输出重力项,l_d 与 l_o 分别构造下三角矩阵 L 的对角与非对角元素,通过 LᵀL = H 得到正定的广义惯量矩阵 H。
随后通过物理变换层(橙色模块)计算惯性项 H·x、科氏与离心项 ∂H/∂t·x、以及 ∂/∂qᵢ(xᵀHx) 等拉格朗日机械量,逐项加总得到保守力部分。
图右下角紫色的"Friction extension"模块,以 q̇ 为输入,按公式 τ_f = b·q̇ + c·arctan(100·q̇) 计算每个关节的摩擦力,最后与保守力相加得到最终 τ。
这种结构既保留了 DeLaN 的物理可解释性,又显著提升了在真实存在摩擦的机器人系统上的预测精度。

: g outputs the gravity term directly; l_d and l_o construct the diagonal and off-diagonal entries of a lower-triangular matrix L, so that

via τ_f = b·q̇ + c·arctan(100·q̇), 


This structure preserves DeLaN's physical interpretability while substantially improving prediction accuracy on real robotic systems where friction is non-negligible.

**English**:
This slide shows our proposed DeLaN model with friction compensation. 

The total predicted torque is decomposed as τ̂ = τ̂_cons + τ_f(q̇), where τ̂_cons is the conservative part from the original DeLaN and τ_f(q̇) is the new friction term.

In the main diagram, the inputs are joint position q, velocity q̇, and acceleration q̈. 

The position q is fed into a sine-activated MLP backbone, which branches into three linear heads. and H = LᵀL here gives a positive-definite generalized inertia matrix.


The orange "physics transformation" blocks then compute the inertial term H·x, the Coriolis term ∂H/∂t·x, and the Lagrangian quantity ∂/∂qᵢ(xᵀHx). 

Summing them yields the conservative torque.

The purple "Friction extension" block at the bottom right takes q̇ as input and learns per-joint friction, which is added to the conservative part to produce the final τ.



---

Slide 3 — τ_f Selection

中文:
这一页解释我们为什么选择当前形式的摩擦模型。我们对比了三种候选方案:
公式 3.a 是经典的 Stribeck 摩擦模型,综合了库仑摩擦 τ_c、静摩擦 τ_s(随速度指数衰减)以及粘滞摩擦 d·q̇。它物理上最完整,但参数多、非线性强,训练时数值上不够稳定。
公式 3.b 是我们提出的粘滞–库仑混合模型:τ_{f,i}(q̇_i) = b_i·q̇_i + c_i·arctan(100·q̇_i)。用 arctan(100·q̇) 平滑近似 sign(q̇),既保留了库仑摩擦在零速度附近的切换特性,又处处可导,便于反向传播;粘滞项 b·q̇ 则刻画速度相关的摩擦。这一形式参数极少、物理含义清晰,实现也非常简单。
公式 3.c 给出了将摩擦纳入拉格朗日框架的理论依据——通过引入 Rayleigh 耗散函数 R,使 τ_f = −∂R/∂q̇,从而把非保守摩擦力自然地嵌入欧拉–拉格朗日方程。
综合考虑物理可解释性与实现简洁性,我们最终采用公式 3.b。



**English**:
This slide explains why we chose the current form of the friction model. We compared three candidates:

The first is the classical Stribeck friction model.
It combines Coulomb friction, static friction, and viscous friction, making it physically comprehensive. However, it contains many parameters and strong nonlinearities, which make training less stable.

The second — and the one we finally adopt. τ_{f,i}(q̇_i) = b_i·q̇_i + c_i·arctan(100·q̇_i). The term arctan(100·q̇) smoothly approximates sign(q̇), retaining the Coulomb switching behavior near zero velocity while remaining everywhere differentiable — perfect for back-propagation. The viscous term b·q̇ captures velocity-dependent friction. 

Formula 3.c provides the theoretical justification for embedding friction in the Lagrangian framework: by introducing a Rayleigh dissipation function R with τ_f = −∂R/∂q̇

Balancing physical interpretability and implementation simplicity, we finally adopt formula 3.b.

------------


Formula 3.a is the classical Stribeck friction model, combining Coulomb friction τ_c, static friction τ_s (with an exponential velocity-dependent decay), and viscous friction d·q̇. It is the most physically complete, but the many parameters and strong nonlinearity make training numerically less stable.


Formula 3.b is our proposed viscous–Coulomb model: τ_{f,i}(q̇_i) = b_i·q̇_i + c_i·arctan(100·q̇_i). The term arctan(100·q̇) smoothly approximates sign(q̇), retaining the Coulomb switching behavior near zero velocity while remaining everywhere differentiable — perfect for back-propagation. The viscous term b·q̇ captures velocity-dependent friction. It has very few parameters, a clear physical meaning, and is extremely simple to implement.


Formula 3.c provides the theoretical justification for embedding friction in the Lagrangian framework: by introducing a Rayleigh dissipation function R with τ_f = −∂R/∂q̇, the non-conservative friction force fits naturally into the Euler–Lagrange equation.
Balancing physical interpretability and implementation simplicity, we finally adopt formula 3.b.

---

Slide 4 — Model Training

中文:
这一页介绍模型训练设置。表 I 列出了主要超参数:损失函数采用 MSE,优化器使用 AdamW,学习率 5×10⁻⁴,权重衰减 1×10⁻⁵,mini-batch 大小 512,训练 1300 个 epoch,激活函数采用 sine——这与 DeLaN 中要求二阶可导以构造 H 的需求是一致的。
右侧公式 3.d 是我们使用的总损失函数,由两部分组成:
第一项 L_inv = 𝔼[‖τ̂ − τ‖²] 是逆动力学损失,衡量预测力矩与真实力矩之间的均方误差,这是主要的监督信号。
第二项 L_E = 𝔼[(Ė_model − q̇ᵀ(τ − F(q̇)))²] 是能量一致性损失,要求模型预测的总能量变化率,与外力(扣除摩擦耗散 F(q̇) 之后)所做的功率相匹配,这一项把能量守恒约束注入训练。
最终总损失 L = L_inv + λ_E·L_E,通过权重 λ_E 平衡两项。这种带物理约束的训练方式让 DeLaN 不仅拟合数据,还遵守底层物理规律,从而在外推场景下泛化能力更强。

**English**
:
This slide covers the training setup. Table I lists the main hyperparameters: loss function MSE, optimizer AdamW, learning rate 5×10⁻⁴, weight decay 1×10⁻⁵, mini-batch size 512, 1300 epochs, and sine activation — consistent with DeLaN's need for twice-differentiable activations to construct H.

The total training loss consists of two terms.

The first is the inverse-dynamics loss:

which measures the error between predicted and measured torques and serves as the main supervision signal.

The second is the energy-consistency loss:This term enforces consistency between the predicted energy change and the actual input power of the system

The final loss is:where λ_E balances the two objectives.

 ----------


On the right, formula 3.d is the total loss used for training, with two components:
The first, L_inv = 𝔼[‖τ̂ − τ‖²], is the inverse-dynamics loss — the mean-squared error between predicted and measured torques, which is the primary supervisory signal.
The second, L_E = 𝔼[(Ė_model − q̇ᵀ(τ − F(q̇)))²], is the energy-consistency loss. It enforces that the model-predicted rate of change of total energy matches the power delivered by external torques after subtracting friction dissipation F(q̇), injecting an energy-conservation constraint into training.
The total loss is L = L_inv + λ_E·L_E, with λ_E balancing the two terms. This physics-constrained training pushes DeLaN beyond simple curve fitting — it respects the underlying physics, which yields stronger generalization in extrapolation scenarios.

---

Here is the updated Slide 3 bilingual script with the three friction methods comparison:

---

Slide 3 — τ_f Selection (τ_f 选型)

中文:

这一页我们讨论摩擦项 τ_f 的三种候选计算方式，以及为什么最终选择了第二种。

方法一：恒定摩擦系数。  
将摩擦力建模为 τ_f = c·sign(q̇)，其中 c 为固定标量。这种方法最简单，计算开销最低，但无法适应不同关节速度下的非线性摩擦特性，在高精度力矩预测中误差偏大。

方法二：速度依赖的摩擦网络。  
τ_f = f(q̇) 由一个独立的小网络输出，输入仅为关节速度。该网络在训练数据上学到了速度相关的摩擦曲线，不需要额外梯度计算，推理速度快。实验对比表明，这种方法在实时力矩预测的 MSE 和 MAE 指标上综合表现最优，兼顾了精度与推理延迟。

方法三：基于能量耗散的摩擦计算。  
τ_f 由能量对速度的梯度导出，即 τ_f = ∂F/∂q̇，其中 F 为耗散函数。该方法严格保证能量守恒，非常适合用于能量模拟和长期动力学推演。但由于需要在线计算梯度，推理开销显著增加，在 τ 的实时预测场景下延迟超出控制周期，表现不佳。

结论： 综合实时精度与推理速度，我们选择方法二作为 τ_f 的计算方式，将方法三保留用于离线能量分析场景。

---

English:

This slide discusses three candidate formulations for the friction term τ_f, and why the second one was ultimately selected.

Method 1: Constant Friction Coefficient.  
The friction is modeled as τ_f = c·sign(q̇), where c is a fixed scalar. This is the simplest approach with minimal computational overhead, but it cannot capture nonlinear friction behavior across different joint velocities, resulting in larger errors for high-precision torque prediction.

Method 2: Velocity-Dependent Friction Network.  
τ_f = f(q̇) is output by a small dedicated network whose input is joint velocity only. This network learns velocity-dependent friction curves from training data and requires no additional gradient computation at inference, enabling fast evaluation. Experimental comparisons show that this method achieves the best combined performance on real-time torque prediction MSE and MAE metrics, balancing accuracy and inference latency.

Method 3: Energy-Dissipation-Based Friction.  
τ_f is derived from the gradient of energy with respect to velocity, i.e., τ_f = ∂F/∂q̇, where F is a dissipation function. This formulation rigorously guarantees energy conservation and is well-suited for energy simulation and long-term dynamics propagation. However, because it requires online gradient computation, the inference overhead increases significantly, causing latency to exceed the control cycle in real-time τ prediction scenarios, where its performance degrades.

Conclusion: Considering real-time accuracy and inference speed together, we selected Method 2 as the τ_f computation. Method 3 is reserved for offline energy analysis scenarios.

---

