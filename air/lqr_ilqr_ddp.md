## **Overall for LQR, iLQR and DDP**

### **LQR (Linear Quadratic Regulator):**
1. Optimal control algorithm for linear systems  
2. Finds a feedback controller to take the system from initial state to goal state while minimizing a quadratic cost function  
3. Requires a linear dynamics model and quadratic cost function  
4. Can compute control inputs online efficiently  

### **iLQR (Iterative LQR):**
1. Iterative version of LQR that handles nonlinear systems  
2. Linearizes the nonlinear dynamics through Gaussian Newton iterations and applies LQR  
3. Requires gradients of dynamics model and cost function  
4. More costly than LQR but can still compute online  

### **DDP (Differential Dynamic Programming):**
1. Online nonlinear optimal control algorithm  
2. Similar to iLQR but uses second order information to better locally approximate the system as quadratic  
3. Reduces linearization errors compared to iLQR  

---
> **LQR 要求系统动力学是线性的，而代价函数是二次型。  


LQR 假设系统满足：

\[
\dot{x} = Ax + Bu
\]


LQR 的代价函数是：

\[
J = \int (x^\top Q x + u^\top R u)\, dt
\]

这里的二次型来自：
- 惩罚状态偏差：\(x^\top Q x\)
- 惩罚控制能量：\(u^\top R u\)

**为什么代价函数要用二次型？**

因为二次型有三个关键好处：

### **(1) 保证凸性 → 唯一最优解**
如果 \(Q \succeq 0\)、\(R \succ 0\)，代价函数是凸的。

### **(2) 与线性动力学组合后 → 形成线性二次调节器（LQR）**
线性动力学 + 二次代价  
→ 代数黎卡提方程（ARE）可解  
→ 得到闭式解 \(K\)

### **(3) 计算效率极高**
LQR 的反馈律：

\[
u = -Kx
\]

可以在线计算。

> online control 和 offline control 的区别在于：
- **Online control**：在系统运行时实时计算控制输入，适用于动态环境和需要快速响应的场景。
- **Offline control**：在系统运行前预先计算控制策略，适用于环境稳定且不需要实时调整的场景。

对一个标量函数  
\[
f(x_1, x_2, \dots, x_n)
\]
它的 Hessian 是：

\[
H = \nabla^2 f =
\begin{bmatrix}
\frac{\partial^2 f}{\partial x_1^2} & \frac{\partial^2 f}{\partial x_1 \partial x_2} & \cdots \\
\frac{\partial^2 f}{\partial x_2 \partial x_1} & \frac{\partial^2 f}{\partial x_2^2} & \cdots \\
\vdots & \vdots & \ddots
\end{bmatrix}
\]

## **1. Hessian 正定 ⇔ 函数是凸的**
如果  
\[
H \succ 0
\]
则函数是严格凸的，有唯一最优解。

这就是为什么 QP 要求 Hessian 正定。

---

## **2. Hessian 是二次型的矩阵**
如果  
\[
f(x) = \frac12 x^\top H x
\]
那么  
\[
\nabla^2 f = H
\]

这就是 LQR、QP、iLQR、DDP 都喜欢用二次型的原因：  
**二次型的 Hessian 就是常数矩阵，计算简单、结构好。**

---

## **3. Hessian 决定牛顿法、DDP 的二阶更新**
- **牛顿法**：  
  \[
  x_{k+1} = x_k - H^{-1} \nabla f
  \]
- **DDP**：使用 Hessian（即二阶信息）构造更准确的局部二次近似  
- **iLQR**：只用一阶线性化（没有 Hessian），所以比 DDP 粗糙

---


## **① 在 QP（Quadratic Programming）中**
目标函数：
\[
\min \frac12 x^\top H x + f^\top x
\]
这里的 \(H\) 就是 Hessian。

- 若 \(H \succ 0\)，QP 有唯一最优解  
- 若 \(H\) 不正定，问题可能无界或不可解  

---

## **② 在 LQR 中**
代价函数：
\[
J = \int (x^\top Q x + u^\top R u)\, dt
\]

这里的 Hessian 就是：
- 对状态：\(Q\)
- 对控制：\(R\)

因为代价是二次型，所以 Hessian 是常数矩阵。

---

## **③ 在 iLQR 中**
iLQR 对非线性系统做一阶线性化，但对代价函数做二阶展开：

\[
\ell(x,u) \approx \ell_0 + \ell_x^\top \delta x + \ell_u^\top \delta u
+ \frac12 \delta x^\top \ell_{xx} \delta x
+ \frac12 \delta u^\top \ell_{uu} \delta u
+ \delta u^\top \ell_{ux} \delta x
\]

这里的  
\[
\ell_{xx},\ \ell_{uu},\ \ell_{ux}
\]
就是 Hessian 的分块。

---

## **④ 在 DDP 中**
DDP 不仅对代价做二阶展开，还对动力学做二阶展开，因此需要：

\[
f_{xx},\ f_{xu},\ f_{uu}
\]

DDP 的二阶信息更完整，所以比 iLQR 更准确。

---

设  
\[
f(x,y) = 3x^2 + 2xy + y^2
\]

梯度：
\[
\nabla f = 
\begin{bmatrix}
6x + 2y \\
2x + 2y
\end{bmatrix}
\]

Hessian：
\[
H = 
\begin{bmatrix}
6 & 2 \\
2 & 2
\end{bmatrix}
\]

这是一个常数矩阵，因为函数是二次型。

-----------
1️⃣ **What is iLQR? / 什么是 iLQR？**

### **EN**
iLQR (Iterative Linear Quadratic Regulator) is a **second‑order trajectory optimization method** used for nonlinear systems. It improves a control sequence iteratively by:
- linearizing dynamics  
- quadratizing cost  
- solving an LQR problem  
- updating the trajectory  

It is the foundation of many modern control frameworks (e.g., DDP, MPC, Crocoddyl).

### **中文**
iLQR（迭代线性二次调节器）是一种用于 **非线性系统的二阶轨迹优化方法**。它通过以下步骤不断改进控制序列：
- 对动力学线性化  
- 对代价函数二次化  
- 求解 LQR 问题  
- 更新轨迹  

它是许多现代控制框架（如 DDP、MPC、Crocoddyl）的基础。

---

## 2️⃣ **Optimization Problem / 优化问题形式**

### **EN**
iLQR solves the nonlinear optimal control problem:

\[
\min_{u_{0:T-1}} \sum_{t=0}^{T} \ell(x_t,u_t)
\]

subject to nonlinear dynamics:

\[
x_{t+1} = f(x_t,u_t)
\]

### **中文**
iLQR 求解如下非线性最优控制问题：

\[
\min_{u_{0:T-1}} \sum_{t=0}^{T} \ell(x_t,u_t)
\]

满足非线性动力学：

\[
x_{t+1} = f(x_t,u_t)
\]

---

## 3️⃣ **Key Idea: Local Quadratic Approximation / 核心思想：局部二次近似**

### **EN**
At each iteration, iLQR:
1. **Linearizes** dynamics around the current trajectory  
2. **Quadratizes** the cost function  
3. Solves the resulting **LQR problem**  
4. Updates the control sequence  

This makes iLQR much faster than generic nonlinear solvers.

### **中文**
在每次迭代中，iLQR 会：
1. **对动力学做线性化**  
2. **对代价函数做二次近似**  
3. 求解得到的 **LQR 问题**  
4. 更新控制序列  

这使得 iLQR 比通用非线性优化器快得多。

---

## 4️⃣ **Forward–Backward Pass / 前向–后向迭代**

### 🔙 **Backward Pass / 后向传播**

### **EN**
Compute feedback gains \(K_t\) and feedforward terms \(k_t\) by solving Riccati‑like equations:

\[
Q_x, Q_u, Q_{xx}, Q_{ux}, Q_{uu}
\]

Then compute:

\[
k_t = -Q_{uu}^{-1} Q_u
\]
\[
K_t = -Q_{uu}^{-1} Q_{ux}
\]

### **中文**
通过求解类似黎卡提方程，计算反馈增益 \(K_t\) 和前馈项 \(k_t\)：

\[
k_t = -Q_{uu}^{-1} Q_u
\]
\[
K_t = -Q_{uu}^{-1} Q_{ux}
\]

这些量告诉我们如何调整控制输入。

---

### 🔜 **Forward Pass / 前向传播**

### **EN**
Apply the updated control law:

\[
u_t^{new} = u_t^{old} + \alpha k_t + K_t (x_t^{new} - x_t^{old})
\]

Simulate dynamics to obtain a new trajectory.

### **中文**
使用更新后的控制律：

\[
u_t^{new} = u_t^{old} + \alpha k_t + K_t (x_t^{new} - x_t^{old})
\]

然后前向模拟动力学，得到新的轨迹。

---

## 5️⃣ **Line Search / 线搜索**

### **EN**
The step size \(\alpha\) is chosen via line search to ensure cost decreases.

### **中文**
步长 \(\alpha\) 通过线搜索确定，以确保代价下降。

---

## 6️⃣ **Convergence Properties / 收敛特性**

### **EN**
iLQR converges quickly when:
- dynamics are smooth  
- cost is well‑conditioned  
- initial guess is reasonable  

But it may fail when:
- contact dynamics exist  
- cost landscape is highly non‑convex  

### **中文**
iLQR 在以下情况下收敛很快：
- 动力学平滑  
- 代价函数条件良好  
- 初始猜测合理  

但在以下情况可能失败：
- 存在接触动力学  
- 代价函数高度非凸  

---

## 7️⃣ **Relation to DDP / 与 DDP 的关系**

### **EN**
DDP (Differential Dynamic Programming) is the “full second‑order” version of iLQR:
- iLQR ignores second‑order dynamics terms  
- DDP keeps them  

DDP is more accurate but more expensive.

### **中文**
DDP 是 iLQR 的“完全二阶”版本：
- iLQR 忽略动力学的二阶项  
- DDP 保留这些项  

DDP 更精确但计算更重。

---

## 8️⃣ **Why iLQR is Popular? / 为什么 iLQR 很流行？**

### **EN**
- Fast  
- Scales to long horizons  
- Produces feedback policies  
- Works well for locomotion and manipulation  
- Foundation of modern MPC and Crocoddyl  

### **中文**
- 快速  
- 可处理长时间规划  
- 输出反馈控制律  
- 在行走、操作任务中表现优秀  
- 是现代 MPC 和 Crocoddyl 的基础  

---

# 🌟 **Summary / 总结**

### **EN**
iLQR is an efficient trajectory optimization method that uses local approximations and LQR structure to solve nonlinear optimal control problems.

### **中文**
iLQR 是一种高效的轨迹优化方法，通过局部近似和 LQR 结构求解非线性最优控制问题。

