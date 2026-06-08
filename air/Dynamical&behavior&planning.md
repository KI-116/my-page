# Dynamical systems Lec4

## 1. **Dynamical systems（动力系统）**
The configuration of a robot (joint angles, body pose) is not static; it changes over time due to dynamics and control inputs. We can model this evolution using **dynamical systems**. It evolves according to time-varying equations of motion, which can be expressed as a system of ordinary differential equations (ODEs):
\[\dot x = f(x, u)\]

\[x\] is the state (e.g., joint angles and velocities), and \[u\] is the control input (e.g., torques). 

**核心思想：**  
把机器人、机械臂、摆、车辆等都看成一个随时间演化的系统，其状态 \(x(t)\) 随时间变化。
systems**.

**关键点：**
- 状态随时间变化：\(\dot x = f(x, u)\)
- 输入 \(u\)（力、力矩、电压）影响系统演化
- 可以分析稳定性、平衡点、吸引域等

**为什么重要：**  
所有控制方法（PD、LQR、iLQR、OSC、MPC）都建立在动力系统模型之上。


## 2. **Equations of motion（运动方程）**
**机器人动力学的数学描述。**

典型形式（机械系统）：
\[
M(q)\ddot q + C(q,\dot q)\dot q + g(q) = \tau
\]

**含义：**
- \(M(q)\)：质量矩阵  
- \(C(q,\dot q)\dot q\)：科氏力、离心力  
- \(g(q)\)：重力  
- \(\tau\)：关节力矩（控制输入）

**为什么重要：**  
这是所有基于模型的控制（CTC、OSC、能量 shaping、反馈线性化）的基础。

---

## 3. **Systems of first-order ODEs（一阶常微分方程系统）**
动力学通常是二阶（\(\ddot q\)），但仿真和控制更喜欢**一阶形式**：

\[
\dot x = f(x, u), \quad x = \begin{bmatrix} q \\ \dot q \end{bmatrix}
\]

**原因：**
- 数值求解器（Euler、RK45）都要求一阶形式
- 状态空间控制（LQR、MPC）必须是一阶系统


## 4. **State space（状态空间）**
把系统写成：

\[
\dot x = f(x, u)
\]

**优点：**
- 可以统一描述线性/非线性系统
- 可以做线性化：\(\dot x = Ax + Bu\)
- 是现代控制（LQR、MPC、iLQR）的语言

**在机器人中：**  
状态通常包含位置、速度，有时还包括力、接触状态等。


## 5. **Simulation techniques（仿真方法）**
### **Euler（显式欧拉）**
最简单但不稳定：

\[
x_{k+1} = x_k + h f(x_k, u_k)
\]

优点：快  
缺点：误差大、容易爆炸


### **RK45（Runge–Kutta 4/5）**

机器人动力学通常是连续时间系统, 而控制器和优化器通常需要离散时间模型。RK4经常用来构造离散模型。

`x_next = RK4(f, x, u, dt)`


### **DAEs（微分代数方程）**
当系统有**约束**（如机械臂的闭链、接触约束）时：

\[
\begin{cases}
\dot x = f(x, u) \\
0 = g(x)
\end{cases}
\]

机器人接触、闭链机构、车轮无侧滑约束 → 都是 DAE。


## 6. **Underactuated systems（欠驱动系统）**
**定义：**  
自由度 > 控制输入数。

例子：
- 倒立摆（2 DOF, 1 input）
- 四足机器人（身体姿态不可直接控制）
- 无人机（位置 3 DOF，但推力方向受限）

**难点：**
- 不能直接控制所有自由度
- 需要利用动力学耦合、能量方法、轨迹优化


## 7. **Partial feedback linearisation（部分反馈线性化）**
对欠驱动系统，只能对**可控子空间**做反馈线性化。

例如倒立摆：
- 角度可控
- 小车位置不可直接线性化

**思想：**
- 找到可控输出 \(y = h(x)\)
- 通过控制律让其变成线性系统：
  \[
  y^{(r)} = v
  \]
- 其余自由度留在零空间中

这是现代机器人控制（尤其是腿式机器人）的核心技术之一。

## 8. **Energy shaping（能量整形）**
通过控制输入改变系统的能量函数，使其具有期望的稳定点。

例子：倒立摆摆起（swing-up）  
目标：让系统能量从低能量（垂下）提升到高能量（直立）

控制律通常基于能量误差：
\[
u = k (E_{\text{desired}} - E(x)) \dot q
\]

**优点：**
- 不需要线性化
- 对欠驱动系统特别有效
- 常用于摆起、跳跃、跑步等“运动技能”



# hybrid systems Lec5

Robots with contacts (walking, manipulation, impacts) cannot be described by a single smooth ODE. They switch between continuous dynamics (flight phase, stance phase) and discrete events (impact, contact gain/loss).

## 2. **Poincaré Maps（庞加莱映射）**

**EN:** A tool to analyze stability of periodic motions (limit cycles). Instead of analyzing full continuous trajectories, we examine how the system state returns to a surface of section.  
**CN:** 用于分析周期运动（极限环）稳定性的工具。通过观察系统在“截面”上的离散返回点，而不是整个连续轨迹。

**用途：**
- 步行机器人步态稳定性分析  
- 被动行走器（rimless wheel、compass gait）  
- juggling、跳跃等周期运动  


## 3. **Contacts（接触）**
**EN:** When two rigid bodies touch, they must satisfy a unilateral constraint  
\[
\phi(q) \ge 0
\]  
meaning no penetration.  
**CN:** 两刚体接触时必须满足单边约束 \(\phi(q) \ge 0\)，即不能穿透。

**关键点：**
- 接触力只能“推”不能“拉”（repulsive only）  
- 接触不满足速度条件时会产生冲量（impact）  

**Lecture 内容：** rigid body contact, signed distance, unilateral constraint  


## 4. **Inequality Constraints（不等式约束）**
**EN:** Contact constraints are inequality constraints on velocity and acceleration.  
**CN:** 接触约束在速度和加速度层面表现为不等式约束。

三种状态（Lecture 图 11.4）：
1. **State 1:** no contact（ϕ>0）  
2. **State 2a:** impact（ϕ=0, ϕ̇<0）  
3. **State 2b:** separating（ϕ=0, ϕ̇>0）  
4. **State 3:** persistent contact（ϕ=0, ϕ̇=0, ϕ̈≥0）

**Lecture 内容：** complementarity conditions  
\[
\dot\zeta \ge 0,\quad \lambda \ge 0,\quad \dot\zeta \lambda = 0
\]  


## 5. **Multi-Point Contact（多点接触）**
**EN:** Real robots have multiple simultaneous contacts (feet, hands, fingertips).  
**CN:** 实际机器人常常有多个接触点（双脚、双手、指尖）。

矩阵形式：
- \(N\)：所有接触法向量  
- \(\lambda\)：所有接触力  
- \(\zeta\)：所有接触分离速度  

接触动力学写成：
\[
\dot\zeta = M\lambda + d
\]

**Lecture 内容：** multi-contact complementarity system  


## 6. **Impulse Dynamics（冲量动力学）**
**EN:** Impulse is the integral of force over a very short time, causing instantaneous velocity change.  
**CN:** 冲量是力在极短时间内的积分，会导致速度瞬时跳变。

\[
\iota = \int f(t)\,dt = h(t_1) - h(t_0)
\]

用于：
- 足部着地  
- 物体碰撞  
- juggling  

**Lecture 内容：** momentum change, impact law  


## 7. **Contact Dynamics as LCP/QP（接触动力学作为互补问题）**
**EN:** Solving contact forces requires solving a **Linear Complementarity Problem (LCP)** or equivalent **Quadratic Program (QP)**.  
**CN:** 求解接触力需要求解 **线性互补问题 LCP** 或等价的 **二次规划 QP**。

LCP 形式：
\[
\dot\zeta = M\lambda + d,\quad 
\dot\zeta \ge 0,\ \lambda \ge 0,\ \dot\zeta^T\lambda = 0
\]

**Lecture 内容：** LCP/QP equivalence  

## 8. **Considerations for Developing a Contact Dynamics Simulator（构建接触动力学仿真器的要点）**
**EN:** A good simulator must handle both continuous integration and discrete events.  
**CN:** 一个好的仿真器必须同时处理连续积分与离散事件。

关键点：
- event detection（接触建立/失效、冲量）  
- step truncation（缩短积分步长）  
- variable-step integration（变步长积分）  
- avoid Euler（不稳定）  
- use RK45 / ode23 / ode23t  

**Lecture 内容：** hybrid simulation architecture  


## 9. **Existing Libraries（现有动力学库）**
Lecture 中列出的库：

### **Pinocchio（LAAS-CNRS）**
- 高效的几何动力学库  
- 支持多接触、多体系统  
- C++/Python  

### **RBDL（Uni Heidelberg）**
- 轻量级刚体动力学库  
- 适合教学与快速原型  

### **KDL（KU Leuven）**
- 机器人运动学/动力学工具箱  
- ROS 生态常用  

### **HyRoDyn（DFKI-RIC）**
- 高级混合动力学库  
- 尚未开源  

**Lecture 内容：** recommended libraries  

**EN:**  
This lecture builds the mathematical and computational foundation for analyzing and controlling robots that interact with the world through impacts and contacts. Hybrid systems, Poincaré maps, complementarity constraints, and impulse dynamics form the core tools for understanding legged locomotion, manipulation, and dynamic behaviors.

**CN：**  
本讲义构建了分析与控制带接触机器人的数学与计算基础。混合系统、庞加莱映射、互补约束、冲量动力学是理解腿式机器人、操作机器人以及动态行为的核心工具。



## QUESTIONS
> hybrid dynamical systems, how to deal with contacts:

**EN:**  
Contacts introduce *discontinuities* — when a robot’s foot hits the ground, velocity changes instantaneously. The system switches between continuous dynamics (flight phase) and discrete events (impact).  
**CN:**  
接触导致系统出现**不连续性**：例如机器人脚落地时速度瞬间跳变。系统在**连续动力学（摆动阶段）**与**离散事件（碰撞、接触建立）**之间切换，因此称为混合系统。

**Modeling contacts（接触建模）**
**EN:**  
Contacts are modeled as **unilateral constraints**:
\[
\phi(q) \ge 0
\]
where \(\phi(q)\) is the signed distance between bodies.  
**CN:**  
接触用**单边约束**表示：
\[
\phi(q) \ge 0
\]
其中 \(\phi(q)\) 是物体间的有符号距离。

- 当 \(\phi(q) > 0\)：未接触  
- 当 \(\phi(q) = 0\)：接触  
- 当 \(\phi(q) < 0\)：穿透（物理上不允许）

**Complementarity conditions（互补条件）**
**EN:**  
To ensure physically correct contact forces:
\[
\dot\phi \ge 0,\quad \lambda \ge 0,\quad \dot\phi \lambda = 0
\]
**CN:**  
为了保证物理正确性，接触力与分离速度满足互补条件：
\[
\dot\phi \ge 0,\quad \lambda \ge 0,\quad \dot\phi \lambda = 0
\]
即：  
- 若分离速度为正（物体分开），接触力为零  
- 若接触力为正（推开），分离速度为零  

这就是 **Linear Complementarity Problem (LCP)** 的核心。

**Impulse dynamics（冲量动力学）**
**EN:**  
During impact, forces act over a very short time → integrate to impulse:
\[
M(q)(\dot q^+ - \dot q^-) = J_c^T \iota
\]
**CN:**  
碰撞时，力在极短时间内作用形成**冲量**：
\[
M(q)(\dot q^+ - \dot q^-) = J_c^T \iota
\]
其中：
- \(\dot q^-\)：碰撞前速度  
- \(\dot q^+\)：碰撞后速度  
- \(\iota\)：冲量（impact impulse）

这描述了速度的瞬时跳变。

5. **Multi-point contact（多点接触）**
**EN:**  
Robots often have multiple contacts (feet, hands). Each contact adds constraints and forces.  
**CN:**  
机器人常有多个接触点（脚、手、指尖），每个接触点都增加约束与力。

系统写成矩阵形式：
\[
\dot\zeta = M\lambda + d
\]
其中 \(\lambda\) 是所有接触力的向量。

6. **Numerical treatment（数值处理）**
**EN:**  
Contacts are solved using **LCP** or **QP** formulations.  
**CN:**  
接触问题通常通过 **线性互补问题（LCP）** 或 **二次规划（QP）** 求解。

- LCP: 用于刚体接触求解  
- QP: 用于优化型控制（如 Whole-Body Control）  
- DAE: 用于连续接触阶段的积分  

7. **Simulation considerations（仿真器设计要点）**
**EN:**  
A contact simulator must handle both continuous integration and discrete events.  
**CN:**  
接触仿真器必须同时处理连续积分与离散事件。

关键技术：
- **Event detection**：检测接触建立/失效  
- **Step truncation**：缩短积分步长以捕捉事件  
- **Variable-step integration**：变步长积分提高精度  
- **Impact resolution**：冲量求解（LCP/QP）  


**Libraries for implementation（实现库）**


**EN:**  
Hybrid dynamical systems handle contacts by combining continuous dynamics with discrete transitions. Contacts are modeled as unilateral constraints, solved via complementarity or optimization, and simulated using event-driven integration.  
**CN:**  
混合动力系统通过结合连续动力学与离散事件来处理接触。接触用单边约束建模，通过互补条件或优化求解，并在仿真中采用事件驱动积分。


# dynamic programming Lec7

## Introduction to Optimal Control  

### Physics is Optimization

> *Action is defined as the integral of the Lagrangian… The path taken by the system is the one for which the action is stationary.*  

 **最小作用量原理（Principle of Least Action）**


> *What if we want to walk different path & be somewhere else?*  

### OCP

> *Optimal control is a control design process using mathematical optimization.*  

- 给定系统动力学  
- 给定任务目标（例如到达某个状态）  
- 给定代价（running cost + terminal cost）  
- **寻找使总代价最小的控制输入 u(t)**

页面明确列出了 OCP 的结构：

\[
J = l_f(x_f) + \int_{t_0}^{t_f} l(x,u)\,dt
\]

- **系统动力学**：\(\dot x = f(x,u)\)  
- **控制策略**：\(u = \pi(x,t)\)  
- **代价项**：状态正则化、控制能量等  

### 最优控制问题（OCP）的正式定义  

\[
\min_u J(x,u,t_0,t_f)
\quad\text{s.t.}\quad
\dot x = f(x,u),\quad x_f = x(t_f)
\]


这就是最优控制的核心问题：  
**在满足动力学约束的前提下，找到使总代价最小的控制输入序列。**

**Finding the optimal control input sequence that minimizes the cost while satisfying the system dynamics.**

### 离散 OCP

离散版本：

- 状态集合 \(S = \{s_1, s_2, …\}\)  
- 动作集合 \(A = \{a_1, a_2, …\}\)  
- 离散动力学：  
  \[
  s[n+1] = g(s[n], a[n])
  \]

> *OCP can be seen as a graph-search problem.*  


## Dynamic Programming（动态规划）

1️⃣ 动态规划的核心思想  
**中文：**  
动态规划（DP）通过“将复杂决策问题分解为一系列子问题”来寻找最优策略。它基于 Bellman 最优性原理：当前最优决策应同时考虑即时代价与未来最优代价。

**English:**  
Dynamic Programming (DP) solves multi-step decision problems by breaking them into smaller subproblems. It relies on the Bellman Optimality Principle: the optimal decision at each step minimizes the immediate cost plus the optimal future cost.

2️⃣ 离散动态规划：直观的最优控制起点  
**中文：**  
在离散时间系统中，状态和动作都是离散集合。Bellman 方程给出了最优 cost-to-go 的递归形式：

\[
J^\*(s)=\min_{a}[\ell(s,a)+J^\*(g(s,a))]
\]

它构成了价值迭代（value iteration）和策略迭代（policy iteration）的基础。

**English:**  
In discrete-time systems, states and actions are discrete sets. The Bellman equation provides a recursive definition of the optimal cost-to-go:

\[
J^\*(s)=\min_{a}[\ell(s,a)+J^\*(g(s,a))]
\]

This forms the basis of value iteration and policy iteration.

3️⃣ 连续动态规划：通向 HJB 的桥梁  
**中文：**  
当时间步长趋近于零，离散 Bellman 方程自然过渡为连续形式的 Hamilton–Jacobi–Bellman (HJB) 方程：

\[
-\frac{\partial J^\*}{\partial t}
= \min_u \left[ l(x,u) + \frac{\partial J^\*}{\partial x} f(x,u) \right]
\]

HJB 是连续时间最优控制的核心 PDE。

**English:**  
As the time step approaches zero, the discrete Bellman equation transitions into the continuous Hamilton–Jacobi–Bellman (HJB) equation:

\[
-\frac{\partial J^\*}{\partial t}
= \min_u \left[ l(x,u) + \frac{\partial J^\*}{\partial x} f(x,u) \right]
\]

The HJB equation is the fundamental PDE of continuous-time optimal control.
4️⃣ 离散 vs 连续：两种形式如何构成最优控制的入口  
**中文：**  
- 离散 DP 提供直观、易理解的最优决策框架  
- 连续 DP（HJB）提供精确描述物理系统的数学工具  
- 两者之间的过渡展示了最优控制的统一结构  
- 现代控制方法（LQR、MPC、RL）都可视为对 HJB 的近似或特例  

**English:**  
- Discrete DP offers an intuitive and accessible framework for optimal decision-making  
- Continuous DP (HJB) provides a mathematically rigorous description of physical systems  
- The transition between them reveals a unified structure of optimal control  
- Modern methods such as LQR, MPC, and Reinforcement Learning can be viewed as approximations or special cases of HJB  

5️⃣ 例子：双积分器（Double Integrator）  
**中文：**  
Lecture07 用双积分器展示如何从 HJB 推导最优控制律。  
通过给定 cost-to-go，可以直接得到最优反馈控制：

\[
u^\* = -q - \sqrt{3}\dot q
\]

**English:**  
Lecture07 uses the double integrator to demonstrate how optimal feedback control can be derived from the HJB equation.  
Given the optimal cost-to-go, the optimal control law becomes:

\[
u^\* = -q - \sqrt{3}\dot q
\]

6️⃣ 总结：为什么 DP 是最优控制的最佳入口？  
**中文：**  
离散与连续动态规划提供了理解最优控制的统一视角：  
- 离散 DP → 直观  
- 连续 HJB → 严谨  
- LQR/MPC/RL → 实用近似  

它们共同构成了现代机器人控制的理论基础。

**English:**  
Discrete and continuous DP offer a unified perspective on optimal control:  
- Discrete DP → intuitive  
- Continuous HJB → rigorous  
- LQR/MPC/RL → practical approximations  

Together, they form the theoretical foundation of modern robotic control.

# Lecture 08: Trajectory Optimization
**EN**
Trajectory optimization allows robots to:
- generate complex motions (walking, jumping, manipulation)
- handle constraints (torque limits, contact forces)
- incorporate dynamics directly
- produce smooth, feasible, optimal trajectories

**中文**
轨迹优化让机器人能够：
- 生成复杂动作（行走、跳跃、操作）
- 处理各种约束（力矩限制、接触力）
- 直接考虑动力学模型
- 得到平滑、可行、最优的轨迹

---

 **Direct Transcription / 直接转录法**

**EN**
Direct transcription discretizes the trajectory into time steps and treats all states and controls as optimization variables. Dynamics become equality constraints. This converts the continuous optimal control problem into a large but structured nonlinear program (NLP).

**中文**
直接转录法将轨迹离散化，把所有状态与控制量都作为优化变量。动力学方程变成等式约束，从而把连续时间的最优控制问题转化为一个大型但结构化的非线性规划（NLP）。

**2. Direct Collocation / 直接配点法**

 **Formulation / 数学形式**

 **EN**
Direct collocation improves transcription by enforcing dynamics not only at nodes but also at *collocation points* between nodes. This yields smoother trajectories and better numerical stability.

 **中文**
直接配点法在节点之间的“配点”处也强制满足动力学，使轨迹更平滑、数值稳定性更好。

 **Cost & Constraints / 代价与约束**
**EN**
The cost function typically includes:
- tracking error
- control effort
- smoothness penalties

Constraints include:
- system dynamics
- control limits
- state limits
- contact constraints (for legged robots)
 **中文**
代价函数通常包括：
- 轨迹跟踪误差
- 控制能量
- 平滑性惩罚项

约束包括：
- 系统动力学
- 控制输入限制
- 状态限制
- 接触约束（用于足式机器人）

 **3. Direct Shooting: iLQR / DDP / 直接射击法**

 **Problem Formulation / 问题形式**

**EN**
Direct shooting optimizes only the control sequence. The trajectory is generated by forward integrating the dynamics. iLQR (iterative LQR) and DDP (Differential Dynamic Programming) are efficient second‑order methods for solving this.

**中文**
直接射击法只优化控制序列，通过前向积分动力学生成轨迹。iLQR 和 DDP 是求解该类问题的高效二阶方法。

**Need for Human Feedback / 需要人工反馈**

 **EN**
Because shooting methods are sensitive to initialization, human intuition is often needed:
- to provide a reasonable initial guess
- to tune cost weights
- to adjust constraints
- to interpret solver failures

**中文**
由于射击法对初始猜测非常敏感，因此常常需要人工经验：
- 提供合理的初始轨迹
- 调整代价函数权重
- 设置合理的约束
- 分析优化失败的原因

**总结 Summary**

**EN**
These three methods—Direct Transcription, Direct Collocation, and Direct Shooting (iLQR/DDP)—form the foundation of trajectory optimization. Each has different numerical properties and is suited for different robotic applications.

**中文**
直接转录、直接配点、直接射击（iLQR/DDP）构成了轨迹优化的三大基础方法。它们各自具有不同的数值特性，适用于不同类型的机器人任务。


# Lecture 09: Planning over Contacts


This lecture introduces planning and trajectory optimization for **hybrid systems**, especially systems involving **contacts** (walking, bouncing, impacts). It covers both cases where the **mode sequence is known** and where it must be **discovered**. It also introduces **mixed‑integer optimization** and **contact implicit methods**.


本讲主要介绍 **混合系统（Hybrid Systems）** 的规划与轨迹优化，特别是涉及 **接触（contacts）** 的机器人系统（如行走、弹跳、碰撞）。内容包括 **已知模式序列** 与 **未知模式序列** 两类问题，并介绍 **混合整数优化（MIP）** 与 **隐式接触方法（Contact Implicit Methods）**。

 1️⃣ **Recap: Hybrid Systems / 回顾：混合系统**

A hybrid system has:
- **continuous dynamics** (e.g., robot motion)
- **discrete modes** (e.g., foot in air / foot on ground)
- **guards** that trigger transitions (e.g., height = 0)
- **reset maps** that update state after impact

混合系统包含：
- **连续动力学**（如机器人关节运动）
- **离散模式**（如脚在空中 / 脚接触地面）
- **触发条件（guards）**（如高度为 0）
- **重置映射（reset map）**（碰撞后的状态更新）

2️⃣ **Recap: Trajectory Optimization / 回顾：轨迹优化**

Goal: find state & control trajectories that minimize cost  
subject to:
- dynamics  
- state constraints  
- input constraints  

目标：寻找最优的状态与控制轨迹，使代价最小  
并满足：
- 动力学约束  
- 状态约束  
- 控制输入约束  

3️⃣ **Hybrid Trajectory Optimization / 混合轨迹优化**


The lecture focuses on planning for systems with **partially discrete dynamics**, i.e., hybrid systems.

Two major cases:
1. **Mode sequence known**  
2. **Mode sequence unknown**


本讲重点是具有部分离散动力学的系统（混合系统）的规划。

两大类问题：
1. **模式序列已知**  
2. **模式序列未知**

4️⃣ **Case 1: Known Mode Sequence / 情况 1：已知模式序列**

If the sequence of modes is known (e.g., stance → swing → stance), the problem becomes a structured trajectory optimization with:
- collocation constraints  
- guard constraints  
- reset maps  

Examples:
- **Rimless wheel**: periodic walking cycle  
- **Basketball bounce pass**: known bounce count  

若模式序列已知（如支撑相 → 摆动相 → 支撑相），轨迹优化只需求解连续变量，包含：
- 配点约束  
- 触发条件  
- 重置映射  

示例：
- **无轮缘轮（rimless wheel）**：周期步态  
- **篮球弹地传球**：已知弹跳次数  

5️⃣ **Case 2: Unknown Mode Sequence / 情况 2：未知模式序列**

When the mode sequence is not known, the optimizer must **discover**:
- when contact happens  
- which region to step on  
- whether thrusters fire  

This requires **discrete decision variables**.

当模式序列未知时，优化器必须 **自行决定**：
- 何时接触  
- 脚落在哪个区域  
- 推进器是否点火  

这需要 **离散决策变量**。

6️⃣ **Mixed Integer Optimization (MIP) / 混合整数优化**

Discrete decisions can be modeled using integer variables:
- z ∈ {0,1}  
- Big‑M method to activate/deactivate constraints  

MIPs:
- very expressive  
- extremely hard (NP-hard)  

离散决策可用整数变量建模：
- z ∈ {0,1}  
- Big‑M 方法激活/关闭约束  

MIP：
- 表达能力强  
- 求解极难（NP-hard）  

示例：  
**Chinese Garden（中国园林踩石头）**  
- 区域约束  
- 步点分配矩阵 H  
- Big‑M 处理区域激活  

 7️⃣ **Contact Implicit Methods / 隐式接触方法**

Contacts can also be modeled using **complementarity constraints**:

\[
z \ge 0,\quad f \ge 0,\quad f \perp z
\]

Meaning:
- if foot is above ground → no force  
- if force exists → foot must be on ground  

These methods:
- avoid integer variables  
- still very hard to solve  
- active research area  

接触也可用 **互补约束** 表示：

\[
z \ge 0,\quad f \ge 0,\quad f \perp z
\]

含义：
- 脚离地 → 无接触力  
- 有接触力 → 必须接触地面  

特点：
- 不需要整数变量  
- 仍然难求解  
- 是当前研究热点  

 8️⃣ **Examples / 示例**

**Rimless Wheel / 无轮缘轮**  
周期性步态优化（已知模式序列）

**Basketball Trick Shot / 篮球弹跳传球**  
不同弹跳次数对应不同解

**Chinese Garden / 中国园林踩石头**  
混合整数优化 + Big‑M

**Contact Implicit Walking / 隐式接触行走**  
互补约束 + 动力学 + 摩擦锥


9️⃣ **Conclusion / 总结**

### **EN**
This lecture covered:
- Hybrid systems  
- Trajectory optimization with known mode sequences  
- Trajectory optimization with unknown mode sequences  
- Mixed integer programming  
- Contact implicit methods  

### **中文**
本讲内容包括：
- 混合系统  
- 已知模式序列的轨迹优化  
- 未知模式序列的轨迹优化  
- 混合整数优化  
- 隐式接触方法  

