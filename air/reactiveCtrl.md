
![control taxonomy](images/image-3.png)

[joint space control](#joint-space-control)



-------------------







#### joint space control

Control is formulated directly on joint coordinates `q`.


- gravity compensation based controllers (pure, ***PID***+GC,***PD***+GC)
    - zero steady‑state error for PID
- computed torque control
    - Useful when the mass–inertia matrix M(q) is not well approximated by a diagonal matrix.
    - uses the full rigid-body model τ = M(q)[q̈_d + K_d ė + K_p e] + C(q,q̇)q̇ + g(q), yielding linear, decoupled error dynamics ë + K_d ė + K_p e = 0 → feedback linearization. Tracks trajectories but requires an accurate dynamic model.
![CTC Scheme](images/image.png)



#### task space control

Control is formulated in end-effector / task coordinates `x`.
> OSC(Operational Space Control)

#### WBC(Whole Body Control) as QP(Quadratic Program)
Modern frame casts instantaneous stabilization as an online Quadratic Program solved at ~1 kHz:

- Decision variables: joint accelerations q̈, torques τ, contact forces λ.
- Cost: weighted sum of task accelerations (CoM, swing-foot, posture …).
$Σᵢ ‖Jᵢ q̈ + J̇ᵢ q̇ − ẍᵢ*‖²_{Wᵢ}$ 
- Constraints (now first-class citizens, unlike closed-form WBC):

![QP Scheme](images/image-1.png)

![WBC](images/image-2.png)

>... how WBC emerged from task space control
... how we can model different robot tasks and constraints in WBC
... how to setup a code example with a quadruped


------------------------

how to keep a system near a desired state or trajectory in the presence of disturbances, model errors, and contact switches.



