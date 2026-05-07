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
