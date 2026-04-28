# M3 right mode

- rigid body
----
- SE(3) — Special Euclidean Group（特殊欧几里得群）

SE(3) 描述三维空间中的**刚体变换**，即**旋转 + 平移**的组合：

$$T = \begin{bmatrix} R & p \\ 0 & 1 \end{bmatrix} \in \mathbb{R}^{4\times4}, \quad R \in SO(3),\ p \in \mathbb{R}^3$$

> SE(3) = SO(3) × ℝ³（旋转部分 + 平移部分）
> SE(3)   ← 六自由度机器人 / 空间运动通用
> SE(2)   ← 二维版本（平面移动机器人常用）
> 不能直接加减，只能通过矩阵乘法复合变换（群运算）。优化时需借助 **se(3) 李代数**（6维向量）进行扰动，再用指数映射 exp 退回 SE(3)
> 实现中常见形式：**4×4 齐次矩阵**
---
- **Euler–Rodrigues Formula**


**给定一个旋转轴 $\hat{s}$ 和旋转角 $\theta$，如何直接计算旋转后的向量？**

即：不通过欧拉角，不构造完整旋转矩阵，直接把向量 $\mathbf{p}$ 绕轴旋转 $\theta$ 得到 $\mathbf{p}'$。

***公式***

$$\mathbf{p}' = \mathbf{p}\cos\theta + (\hat{s} \times \mathbf{p})\sin\theta + \hat{s}(\hat{s} \cdot \mathbf{p})(1 - \cos\theta)$$

三项的几何含义：

| 项 | 含义 |
|----|------|
| $\mathbf{p}\cos\theta$ | 原向量缩短投影到垂直平面 |
| $(\hat{s} \times \mathbf{p})\sin\theta$ | 垂直于轴和v的切向分量 |
| $\hat{s}(\hat{s} \cdot \mathbf{p})(1-\cos\theta)$ | 沿轴方向的分量补偿 |


> **[s]×** 是 $\hat{s}$ 的反对称矩阵（skew-symmetric matrix）(反对称矩阵数学含义：$[s]×\mathbf{v} = \hat{s} \times \mathbf{v}$)




**如何把"轴角"这一紧凑表示，转换为可直接用于坐标变换的旋转矩阵？**

$$\exp : [\hat{s}]\theta \in \mathfrak{so}(3) \mapsto R \in SO(3)$$

**公式拆解**

$$R = \exp([\hat{s}]\theta) = I + [\hat{s}]\sin\theta + [\hat{s}]^2(1-\cos\theta)$$

| 符号 | 含义 |
|------|------|
| $\hat{s} \in \mathbb{R}^3$ | 单位旋转轴 $\|\hat{s}\| = 1$ |
| $\theta \in \mathbb{R}$ | 旋转角度 |
| $[\hat{s}]$ | $\hat{s}$ 的反对称矩阵，即 $[\hat{s}]_\times \in \mathfrak{so}(3)$ |
| $[\hat{s}]^2$ | 反对称矩阵的平方，对称矩阵 |
| $I$ | 恒等映射（零旋转的基） |

***Exponential Map***
so(3) → SO(3)

本质上是两种写法：

$$\underbrace{\mathbf{v}' = \mathbf{v}\cos\theta + (\hat{s}\times\mathbf{v})\sin\theta + \hat{s}(\hat{s}\cdot\mathbf{v})(1-\cos\theta)}_{\text{Rodrigues（作用在向量上）}}$$

$$\underbrace{R = I + [\hat{s}]\sin\theta + [\hat{s}]^2(1-\cos\theta)}_{\text{Exponential Map（构造旋转矩阵）}}$$

> Rodrigues 公式 = Exponential Map 矩阵作用于向量 $\mathbf{v}$ 的展开形式


来自矩阵指数的 Taylor 展开：

$$\exp([\hat{s}]\theta) = \sum_{n=0}^{\infty} \frac{([\hat{s}]\theta)^n}{n!}$$



$$\log : SO(3) \mapsto \mathfrak{so}(3)$$

$$\theta = \arccos\frac{\text{tr}(R)-1}{2}, \quad [\hat{s}] = \frac{R - R^T}{2\sin\theta}$$

> exp 和 log 构成一对工具，在旋转矩阵和轴角之间来回转换。


---

- Forward Kinematics（正运动学）


**已知关节角/位移 $q$，求末端执行器的位姿 $X$。**

$$X = f(q), \quad X \in SE(3),\quad q \in \mathcal{Q}$$

| 对象 | 问题名称 | 描述 |
|------|---------|------|
| **位置层级** | Forward Geometric Problem | 给定 $q$，求位姿 $X \in SE(3)$ |
| **速度层级** | Forward Kinematic Problem | 给定 $\dot{q}$，求末端速度 $\dot{X}$ |

速度层级通过雅可比展开：

$$\dot{X} = \frac{\partial f}{\partial q}\dot{q} = J(q)\dot{q}$$

> $J(q)$ 是**雅可比矩阵**，依赖当前构型 $q$，将关节速度映射到末端速度。


| 场景 | 说明 |
|------|------|
| **轨迹跟踪** | 给定关节插值序列，实时计算末端轨迹 |
| **碰撞检测** | 需要知道每个连杆的位姿，逐级 FK 计算 |
| **可视化 / 仿真** | 驱动 URDF 模型渲染各连杆位置 |
| **雅可比计算基础** | $J(q)$ 依赖 FK 的中间结果 |


---
- Inverse Kinematics（逆运动学）

**已知末端位姿 $X$，求关节角/位移 $q$。**

$$q = f^{-1}(X), \quad X \in SE(3),\quad q \in \mathcal{Q}$$



| 对象 | 问题名称 | 公式 |
|------|---------|------|
| **位置层级** | Inverse Geometric Problem | $q = f^{-1}(X)$ |
| **速度层级** | Inverse Kinematic Problem | $\dot{q} = J^{-1}(q)\dot{X}$ |

冗余自由度时用伪逆代替逆：

$$\dot{q} = J^\dagger(q)\dot{X}$$


**串联 vs 并联（与 FK 对比）**

```
                FK              IK
串联机器人    唯一解，易       多解，难
并联机器人    多解，难         唯一或多解，易
```

## FK 与 IK 完整对比

| | **FK** | **IK** |
|---|---|---|
| 输入 | $q$ | $X \in SE(3)$ |
| 输出 | $X \in SE(3)$ | $q$ |
| 速度形式 | $\dot{X} = J(q)\dot{q}$ | $\dot{q} = J^{-1}(q)\dot{X}$ |
| 串联难度 | 易，唯一解 | 难，多解 |
| 并联难度 | 难，多解 | 易 |
| 冗余处理 | — | 伪逆 $J^\dagger$ |



| 场景 | 说明 |
|------|------|
| **任务空间控制** | 给定目标位姿，求驱动关节指令 |
| **轨迹规划** | 笛卡尔空间路径点 → 关节空间序列 |
| **冗余臂优化** | 用 $J^\dagger$ 在满足末端约束的同时优化次级目标（避障、关节限位） |
| **实时控制** | 速度级 IK 在每个控制周期内增量求解 |

---








---
## 直接应用场景

- **关节轴旋转**：已知关节轴方向，计算旋转后的连杆位姿
- **SO(3) 的指数映射实现**：$\exp(\theta \hat{k}) \in SO(3)$ 的显式计算
- **避免万向节锁**：以轴角形式操作，绕开欧拉角的奇异性
- **四元数的几何理解基础**：Rodrigues 公式与单位四元数旋转等价

---

> 如需继续：反对称矩阵 [k]× 的用法、与四元数旋转公式的对应关系，或指数映射 exp: so(3)→SO(3) 的直接含义，随时告诉我。

-------

## 直接应用场景

| 场景 | 具体用途 |
|------|---------|
| **正/逆运动学** | 末端执行器位姿用 SE(3) 元素表示，目标即在 SE(3) 中求解 |
| **坐标系变换** | 相机系 → 机器人基座系 → 世界系，每一步都是 SE(3) 变换的链式相乘 |
| **SLAM / 位姿图优化** | 每个关键帧位姿是 SE(3) 元素，约束边是相对变换 |
| **轨迹规划** | 在 SE(3) 上做插值（而非分开插值旋转和平移），保证刚体一致性 |
| **手眼标定** | 求解 $AX = XB$，X ∈ SE(3) |



---

## 实用要点

---

> 如需继续：se(3) 李代数的直接用法、SE(3) 插值方法（Screw 运动），或 SE(2) 与 SE(3) 的对比，可随时告诉我。