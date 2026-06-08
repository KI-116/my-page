下面给你 **Lecture 14: Reinforcement Learning** 的 **中英双语高质量总结**，结构清晰、覆盖重点、适合复习与考试**（已基于你当前浏览的 PDF 内容）** 

---

# 🎯 **Lecture 14 强化学习总览（中英双语）**

## ⭐ 一句话总结  
**RL = 在未知环境中，通过试错学习最优策略，以最大化长期累积奖励。**  
**RL = Learning optimal decision-making through trial-and-error interaction with an unknown environment.**

---

# 1️⃣ Motivation 动机  
- 机器人任务越来越复杂，传统控制难以手工设计策略  
- RL 允许机器人 **从经验中学习**、**适应新环境**  
- 典型应用：操作 Manipulation、运动控制 Locomotion、可微物理 Differentiable Physics  


---

# 2️⃣ RL 基础概念 Basics

## ✔ Agent & Environment 智能体与环境  
- Agent 通过 **传感器观察**环境，通过 **执行器行动**  
- 环境返回 **下一状态 + 奖励**  


---

## ✔ Reward 奖励  
- 标量，表示当前表现好坏  
- 目标：**最大化累积奖励**  
\[
G_t = \sum_{k=0}^{\infty} \gamma^k R_{t+k+1}
\]  


---

## ✔ State 状态  
- **Environment state**：真实环境内部状态（不可见）  
- **Agent state**：智能体基于历史构建的内部表示  
- **Markov state**：包含预测未来所需的全部信息  


---

## ✔ MDP / POMDP  
- Fully observable → MDP  
- Partially observable → POMDP（机器人常见）  


---

# 3️⃣ RL 三大核心组件  
## **Policy（策略）**  
- 决定在状态下采取什么动作  
- 可为 deterministic 或 stochastic  


## **Value Function（价值函数）**  
- 评估状态或状态-动作的好坏  
- \(V(s)\), \(Q(s,a)\)  


## **Model（环境模型）**  
- 预测状态转移：  
\[
P(s,a,s') = p(S_{t+1}=s'|S_t=s,A_t=a)
\]  


---

# 4️⃣ RL 算法分类  
## 按学习对象  
- **Value-based**：如 Q-learning  
- **Policy-based**：如 REINFORCE  
- **Actor-Critic**：如 A2C、SAC  


## 按是否使用模型  
- **Model-free**：不学习环境模型（主流）  
- **Model-based**：学习模型用于规划（如 MuZero）  


---

# 5️⃣ 动态规划（已知模型）  
如果环境模型已知，可用 DP 求解：  
- **Policy Iteration**  
- **Value Iteration**  
但只适用于 **小规模离散 MDP**（维度灾难）  


---

# 6️⃣ Model-free RL（未知模型）

## ✔ Monte Carlo (MC)  
- 用完整回合的 return 更新  
- 无偏但方差大  


## ✔ Temporal Difference (TD)  
- 用 bootstrap 更新  
- 有偏但收敛快  


---

# 7️⃣ Q-learning（离散动作最经典）  
更新：  
\[
Q(s,a) \leftarrow Q(s,a) + \alpha[r + \gamma \max_{a'} Q(s',a') - Q(s,a)]
\]  


---

# 8️⃣ 连续状态/动作：函数逼近  
- 用 NN 逼近 \(V_\theta(s)\)、\(Q_\theta(s,a)\)  
- 用 SGD 更新参数  


---

# 9️⃣ Policy Gradient（策略梯度）  
目标：最大化  
\[
J(\theta) = E_{\pi_\theta}[G]
\]  
策略梯度定理：  
\[
\nabla_\theta J(\theta) \propto E_{\pi_\theta}[Q(s,a)\nabla_\theta \log \pi_\theta(a|s)]
\]  


---

# 🔟 REINFORCE（MC PG）  
- 使用 return 作为 Q 的估计  
- 方差大  


---

# 1️⃣1️⃣ Actor-Critic  
- Actor：策略  
- Critic：价值函数  
- 低方差、稳定性更好  


---

# 1️⃣2️⃣ Off-policy PG  
- 允许使用其他策略的数据  
- Importance Sampling  


---

# 1️⃣3️⃣ Deterministic Policy Gradient (DPG)  
- 连续动作更高效  
- 基础：  
\[
\nabla_\theta J = E[\nabla_a Q(s,a)\nabla_\theta \mu_\theta(s)]
\]  


---

# 1️⃣4️⃣ DDPG / TD3 / SAC（现代主流 RL）

## ✔ DDPG  
- Actor-Critic + Replay Buffer + Target Networks  


## ✔ TD3  
- 双 Q 网络  
- 延迟更新  
- 更稳定  


## ✔ SAC（最强之一）  
- 最大化奖励 + 最大化策略熵  
- 更好的探索与稳定性  


---

# 1️⃣5️⃣ Sim2Real（仿真到现实）  
核心问题：**Reality Gap**  
来源：动力学误差、摩擦、视觉差异、延迟等  


## 三大类方法  
### ① 改善仿真  
- System ID  
- Real2Sim2Real  
### ② 提高策略鲁棒性  
- Domain Randomization（最常用）  
### ③ 现实数据适应  
- Residual learning  
- Offline fine-tuning  


---

# 1️⃣6️⃣ RL 在机器人中的优势  
- 可从原始感知学习  
- 适应噪声  
- 适用于连续动作  
- 可学习复杂策略（如运动、操作）  
- Sim2Real 让训练更快更安全  


---

# 📌 最终总结（中英）

## **强化学习的本质**  
- 通过与环境交互学习最优策略  
- 关键是平衡：**探索 vs 利用**  
- 现代 RL 依赖：深度网络、经验回放、目标网络、熵正则化  

## **Essence of RL**  
- Learn optimal behavior through interaction  
- Balance exploration vs exploitation  
- Modern RL powered by deep learning + replay + target networks + entropy regularization  

--------------
下面给你 **Lecture 15: System Identification（系统辨识）** 的 **中英双语总结**，严格依据你当前 PDF 的 **Table of Content + Summary 内容**（引用自当前页面内容 ）。  
我会按照课程结构（Intro → Model Parameters → Experiment Design → Data Handling → Least Squares → Case Study → Conclusion）进行高质量整理，适合复习、考试、作业使用。

---

# 🎯 **Lecture 15 系统辨识：中英双语总结**

## ⭐ 一句话总结  
**系统辨识 = 通过实验数据反推机器人动力学参数，使模型更准确、更可控。**  
**System Identification = estimating unknown dynamic parameters from measured data to build accurate robot models.**

---

# 1️⃣ Introduction 简介  
（来源：Slide 3–4 ）

机器人在执行敏捷动作（如 Atlas 抓取、跳跃）时，需要准确知道：

- **物体的动力学参数**（mass, inertia）
- **机器人的动力学参数**

系统辨识（SysID）就是：  
**通过输入 u[n] 与输出 y[n] 的测量，拟合状态空间模型：**

\[
x[n+1] = f_\Phi(x[n],u[n]),\quad y[n] = g_\Phi(x[n],u[n])
\]

目标：  
\[
\min_{\Phi,x[0]} \sum_{n=0}^{N-1} \|y[n] - \hat y_n\|^2
\]

---

# 2️⃣ Model Parameters 模型参数  
（Slide 8–14 ）

系统辨识主要估计 **动态参数**：

### ✔ Kinematic parameters（运动学参数）
- link lengths  
- frame transforms  
- joint offsets（需标定）

### ✔ Dynamic parameters（动力学参数）
- mass  
- COM position  
- inertia tensor  
- friction  
- motor inertia  

### ✔ Rigid body minimal parameter sets  
- SE(1): \([m]\)  
- SE(2): \([m, c_x, c_y, I_{zz}]\)  
- SE(3): 10 parameters（mass + COM + inertia tensor）

这些参数是 **固有属性**，不会随时间改变。

---

# 3️⃣ Experiment Design 实验设计  
（Slide 27–30 ）

好的辨识实验必须：

### ✔ 覆盖大范围工作空间  
→ 估计 COM 相关参数

### ✔ 包含足够加速度与速度变化  
→ 估计惯性、摩擦

### ✔ 多样化动作  
→ 避免过拟合

### ✔ 安全  
避免激发共振、结构极限

### 常见轨迹设计方法  
- **Fourier series excitation**（频域丰富）  
- **Task-space complex shapes**（椭球轨迹等）

---

# 4️⃣ Data Handling 数据处理  
（Slide 31–33 ）

### 必要测量：
- q（位置）  
- q̇（速度）  
- q̈（加速度）  
- τ（力矩）

### 加速度估计：
- 对速度微分（需滤波）

### 常用滤波器：
- Moving Average（简单但有延迟）  
- Butterworth（频域滤波，可双向消除相位延迟）

---

# 5️⃣ Least Squares & Validation 最小二乘与验证  
（Slide 34–45 ）

## ✔ 关键思想：动力学方程对参数是 **仿射的（affine）**

EOM：  
\[
M(q)\ddot q + C(q,\dot q)\dot q + G(q) = \tau
\]

可写成线性形式：  
\[
Y(q,\dot q,\ddot q)\Phi = \tau
\]

其中 Y 是 **regressor matrix（回归矩阵）**。

---

## ✔ 经典最小二乘  
\[
\min_\Phi \|Y\Phi - \tau\|^2
\]

解析解：  
\[
\Phi = Y^\dagger \tau
\]

---

## ✔ Weighted Least Squares  
考虑不同关节噪声不同：  
\[
\Phi = (Y^T\Sigma_\tau^{-1}Y)^{-1}Y^T\Sigma_\tau^{-1}\tau
\]

---

## ✔ Regularization（正则化）  
利用 CAD 参数作为先验：  
\[
\|Y\Phi - \tau\|^2 + \gamma\|\Phi - \Phi_{CAD}\|^2
\]

---

## ✔ Physical Consistency（物理一致性）  
确保：
- mass > 0  
- inertia positive-definite  
- inertia ellipsoid inside body  

通过约束矩阵 \(P(\Phi)\succeq 0\) 实现。

---

## ✔ Identifiability（可辨识性）  
（Slide 27）

三类：

1. **Not identifiable**（列线性相关）  
2. **Identifiable only in combinations**（如 ml）  
3. **Fully identifiable**

---

## ✔ Model Validation  
- 用辨识参数重新计算 τ  
- 与真实 τ 对比  
- RMSE  
- 可视化惯性椭球

---

# 6️⃣ Case Study 案例：四足机器人在线质量与 COM 辨识  
（Slide 46–49 ）

### 模型：单刚体动力学  
\[
m(\dot v + g) = \sum F_i
\]
\[
R(I_c - m[c]_\times^2)R^T\dot\omega + Rc\times mg = \sum r_i \times F_i
\]

### 参数：  
\[
\Phi = [m, mc_x, mc_y]
\]

### 使用 Kalman Filter 在线更新  
Prediction:  
\[
\hat\Phi^-_k = \hat\Phi^+_{k-1}
\]

Update:  
\[
\hat\Phi^+_k = \hat\Phi^-_k + K_k(z_k - Y_k\hat\Phi^-_k)
\]

---

# 7️⃣ Conclusion 结论  
（Slide 50–51 ）

你学到了：

- 什么是系统辨识、为什么重要  
- 运动学 vs 动力学参数  
- 回归矩阵 Y 的构造  
- 参数可辨识性  
- 如何设计辨识轨迹  
- 如何处理实验数据  
- 最小二乘及其扩展（加权、正则、物理一致性）  
- 在线辨识案例（四足机器人）

---

# 📌 最终中英总结（适合考试复习）

### **系统辨识的核心：**
- 将动力学写成线性形式  
- 构造 regressor matrix  
- 用最小二乘求参数  
- 验证模型是否合理  

### **Essence of SysID:**
- Express dynamics affine in parameters  
- Build regressor matrix  
- Solve via least squares  
- Validate with real data  

