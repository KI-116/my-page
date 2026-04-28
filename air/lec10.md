### Operational Space Control 逐句翻译

---

1. **Operational Space Control (OSC) directly controls motion and forces in task (operational) coordinates (e.g., end‑effector Cartesian space) by mapping task‑space commands into joint torques using dynamically consistent transforms; it decouples task dynamics via the task‑space inertia \(\Lambda\) and supports hierarchical tasks with null‑space projection.**  
   **操作空间控制（OSC）直接在任务坐标（操作空间）（例如末端执行器的笛卡尔空间）中控制运动和力，通过动态一致的变换将任务空间的指令映射为关节力矩；它通过任务空间惯量 \(\Lambda\) 实现任务动力学的解耦，并通过零空间投影支持分层任务。**

2. **What OSC is (short)**  
   **OSC 的简要定义**

3. **Operational Space Control lets you specify what the robot should do in task space (position, force, impedance) and computes joint torques that realize that behavior while respecting full robot dynamics.**  
   **操作空间控制允许你在任务空间中指定机器人应执行的行为（位置、力、阻抗），并计算出在考虑完整机器人动力学下能够实现该行为的关节力矩。**

4. **Core equations and components**  
   **核心方程与组成要素**

5. **Canonical control law (resolved-acceleration form):**  
   **典型控制律（解加速度形式）：**

6. **\(\tau = J^{T}\Lambda\big(\ddot x_d + K_p e + K_d \dot e - \dot J \dot q\big) + b + g,\)**  
   **\(\tau = J^{T}\Lambda\big(\ddot x_d + K_p e + K_d \dot e - \dot J \dot q\big) + b + g,\)**

7. **where \(J\) is the task Jacobian, \(\Lambda\) the task‑space inertia, \(b\) Coriolis/centrifugal, and \(g\) gravity. This produces torques that make the end‑effector follow \(\ddot x_d\) with PD feedback in task space.**  
   **其中 \(J\) 是任务雅可比，\(\Lambda\) 是任务空间惯量，\(b\) 表示科氏/离心项，\(g\) 表示重力项。该律产生的力矩使末端执行器在任务空间内以 PD 反馈跟踪期望加速度 \(\ddot x_d\)。**

8. **Task‑space inertia: \(\Lambda = (J M^{-1} J^{T})^{-1}\) (dynamically consistent mapping from joint inertia \(M\) to task space). This is what decouples task dynamics.**  
   **任务空间惯量：\(\Lambda = (J M^{-1} J^{T})^{-1}\)（将关节惯量 \(M\) 动力学一致地映射到任务空间）。这正是实现任务动力学解耦的关键。**

9. **Null‑space and hierarchy**  
   **零空间与层级**

10. **OSC naturally supports hierarchical control: primary task torques are computed in task space; secondary objectives (posture, joint limits, internal forces) are injected via null‑space torques that do not affect the primary task. Whole‑Body OSC (WBOSC) formalizes this for multi‑contact, multi‑task systems.**  
    **OSC 自然支持分层控制：主任务的力矩在任务空间中计算；次级目标（姿态、关节极限、内部力等）通过不会影响主任务的零空间力矩注入。全身操作空间控制（WBOSC）将此形式化以处理多接触、多任务系统。**

11. **Why use OSC (pros)**  
    **为什么使用 OSC（优点）**

12. **Task‑level decoupling and intuitive gains: tune gains in Cartesian space rather than joint space.**  
    **任务级解耦与直观的增益调节：可以在笛卡尔空间中调整增益，而不是在关节空间中。**

13. **Compliant force/motion control: can regulate interaction forces while accounting for dynamics.**  
    **顺应性的力/运动控制：在考虑动力学的同时可以调节交互力。**

14. **Limitations and practical risks**  
    **局限与实际风险**

15. **Model dependence: OSC requires accurate \(M,C,g,J\). Model errors degrade performance and can destabilize null‑space behavior.**  
    **依赖模型：OSC 需要准确的 \(M,C,g,J\)。模型误差会降低性能并可能使零空间行为不稳定。**

16. **Numerical issues: computing \(\Lambda\) and inverses near singularities needs damping or regularization.**  
    **数值问题：在接近奇异时计算 \(\Lambda\) 和求逆需要阻尼或正则化处理。**

17. **Practical guide: when and how to apply OSC**  
    **实用指南：何时以及如何应用 OSC**

18. **Key considerations before using OSC:**  
    **使用 OSC 前的关键考虑点：**

19. **Do you have a reasonably accurate dynamic model or online adaptation? If not, prefer low‑gain task controllers or learning‑augmented OSC.**  
    **你是否有相当准确的动力学模型或在线自适应？如果没有，优先使用低增益的任务控制器或结合学习的 OSC。**

20. **Is the task contact‑rich or multi‑task? Use WBOSC and contact‑consistent Jacobians.**  
    **任务是否包含大量接触或多任务？应使用 WBOSC 并采用与接触一致的雅可比。**

21. **Implementation tips:**  
    **实现建议：**

22. **Start with gravity compensation + task‑space PD and add \(-\dot J\dot q\) feedforward.**  
    **从重力补偿 + 任务空间 PD 开始，并加入 \(-\dot J\dot q\) 前馈项。**

23. **Use damped inverses or SVD for numerical stability; add null‑space projection for secondary objectives.**  
    **为数值稳定性使用阻尼逆或 SVD；为次级目标加入零空间投影。**

24. **If model error is significant, combine OSC with online adaptation or data‑driven corrections (e.g., OSCAR‑style methods).**  
    **若模型误差显著，将 OSC 与在线自适应或数据驱动的修正方法结合（例如 OSCAR 风格的方法）。**

25. **Quick decision checklist**  
    **快速决策清单**

26. **Accurate model + multi‑DOF task → use OSC (with null‑space hierarchy).**  
    **模型准确且为多自由度任务 → 使用 OSC（并采用零空间层级）。**

27. **Poor model or high uncertainty → use conservative feedback, add learning/adaptation before full OSC.**  
    **模型不佳或不确定性高 → 使用保守的反馈控制，在全面采用 OSC 前加入学习/自适应。**

28. **If you want, I can put the above control law derivation into step‑by‑step math, or apply it to your Lecture 10 example and give a numeric demonstration.**  
    **如果你愿意，我可以把上述控制律的推导写成逐步数学推导，或将其应用到你 Lecture 10 的例子并给出数值示例。**