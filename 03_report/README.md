# RAPID — Technical Report

## Rocket-Assisted Payload Insertion and Descent

This directory contains the consolidated technical documentation for **RAPID (Rocket Assisted Payload Insertion and Descent)**.

RAPID is a concept for **rocket-assisted point-to-point payload delivery**, where a launch vehicle inserts a **Payload Glide Vehicle (PGV)** to altitude. Following separation, the PGV autonomously navigates toward a designated stationary ground target.

The current phase focuses on a **simulation-based Guidance, Navigation and Control (GNC) proof-of-concept**, supported by OpenRocket-based rocket sizing and a simplified point-mass PGV model.

> **Project status:** Proof-of-concept simulation. Current results are modelling results and are not hardware validation or flight-ready performance claims.

---

# 1. Report Purpose

The purpose of this report is to consolidate the technical development of the RAPID concept into a single engineering narrative.

The development flow is:

```text
Mission Requirement
        ↓
PGV Capability Assumptions
        ↓
Required Insertion Altitude
        ↓
Rocket Sizing and Simulation
        ↓
Apogee / PGV Separation
        ↓
V0 — Mission Feasibility
        ↓
V1 — Baseline Guidance
        ↓
Target-Vicinity Analysis
        ↓
Ground-Termination Analysis
````

The central question investigated in the current phase is:

> **Given a rocket-provided separation state, can a simplified PGV autonomously correct its trajectory and guide toward a stationary target 5 km from the launch origin?**

---

# 2. Mission Definition

| Parameter                       |              Value |
| ------------------------------- | -----------------: |
| Target type                     |         Stationary |
| Target X position               |          4000.00 m |
| Target Y position               |          3000.00 m |
| Target range from launch origin |          5000.00 m |
| Launch profile                  |           Vertical |
| PGV configuration               | Steerable parafoil |
| PGV + payload mass              |            0.50 kg |
| Separation altitude             |          1939.57 m |
| Separation X position           |           223.01 m |
| Separation Y position           |             0.14 m |

## Coordinate System

The simulation uses an East-North-Up Cartesian coordinate system:

```text
X → East
Y → North
Z → Up
```

The nominal target location is:

```text
(4000.00, 3000.00, 0.00) m
```

---

# 3. System Architecture

## Rocket-Assisted Insertion

The launch vehicle provides the PGV with its initial separation state near apogee.

Its role is to:

* Perform powered ascent
* Provide altitude insertion
* Establish the PGV release position
* Provide the initial flight state used by the PGV guidance model

## Payload Glide Vehicle

Following separation, the PGV performs:

* Navigation toward the target
* Heading correction
* Guided glide
* Target acquisition
* Terminal trajectory propagation

```text
Launch
  ↓
Powered Ascent
  ↓
Apogee / PGV Release
  ↓
PGV Initial Conditions
  ↓
Guidance
  ↓
Guided Glide
  ↓
Target-Vicinity / Ground-Termination Analysis
```

---

# 4. Rocket Sizing and Insertion

The launch vehicle was designed and simulated using **OpenRocket**.

The rocket sizing stage establishes a demonstration-level baseline configuration intended to provide an insertion altitude of approximately 2 km for the PGV. The rocket sizing outputs form the interface between the launch vehicle and the downstream PGV guidance analysis.

## 4.1 Baseline Rocket Configuration

| Parameter           | Value                         |
| ------------------- | ----------------------------- |
| Total rocket length | **1250 mm**                   |
| Outer diameter      | **100 mm**                    |
| Nose cone           | Ogive, 250 mm, Polystyrene    |
| Body tube           | 1000 mm, 100 mm OD, Cardboard |
| Fins                | 3 × trapezoidal, Balsa, 3 mm  |
| Fin root chord      | **180 mm**                    |
| Fin tip chord       | **80 mm**                     |
| Fin span            | **120 mm**                    |
| Fin sweep           | **120 mm**                    |
| Payload module      | **0.50 kg, Ø80 mm × 300 mm**  |
| Motor mount         | **54 mm ID, 300 mm long**     |
| Selected motor      | **AeroTech J99N**             |
| Motor class         | **J-class, 54 mm**            |
| Delay charge        | **None**                      |

## 4.2 Mass and Stability

The cleaned OpenRocket release-state dataset provides:

| Parameter                          |                    Value |
| ---------------------------------- | -----------------------: |
| Launch mass                        | **1725.03 g (1.725 kg)** |
| Mass at motor burnout              | **1169.50 g (1.170 kg)** |
| Propellant mass burned             |             **555.50 g** |
| Mean early-flight stability margin |        **1.36 calibers** |
| Early-flight stability range       |   **0.95–1.55 calibers** |

The stability margin remains close to the stable regime throughout most of the early-flight interval, with a brief minimum near launch-rod clearance.

## 4.3 OpenRocket Simulation Results

The baseline configuration was evaluated using the **PGV Insertion States** simulation.

| Parameter                   |         Result |
| --------------------------- | -------------: |
| Apogee altitude             |  **1939.57 m** |
| Time to apogee              |    **18.49 s** |
| Maximum velocity            | **181.74 m/s** |
| Maximum acceleration        | **74.41 m/s²** |
| Maximum Mach number         |       **0.54** |
| Launch rod exit velocity    |  **12.09 m/s** |
| Total ballistic flight time |    **49.76 s** |
| Ground impact velocity      |  **72.35 m/s** |

The resulting approximately **1.94 km apogee** is used as the nominal PGV release altitude in the current guidance analysis.

---

# 5. PGV Baseline Model

The PGV is represented using a simplified point-mass kinematic model.

| Parameter            | Symbol             |            Value |
| -------------------- | ------------------ | ---------------: |
| Horizontal speed     | $V_h$              |     **6.20 m/s** |
| Sink rate            | $V_d$              |     **1.92 m/s** |
| Glide ratio          | $V_h/V_d$          |     **3.23 : 1** |
| Maximum heading rate | $\dot{\psi}_{max}$ |     **6.00 °/s** |
| Maximum heading rate | $\dot{\psi}_{max}$ | **0.1047 rad/s** |
| Minimum turn radius  | $R_{min}$          |      **59.22 m** |
| PGV + payload mass   | $m$                |      **0.50 kg** |

The minimum turn-radius estimate is obtained from the horizontal velocity and maximum angular rate:

$$
R_{min}=\frac{V_h}{\dot{\psi}_{max}}
$$

using $\dot{\psi}_{max}$ in radians per second.

> **Validation status:** The PGV parameters are provisional modelling assumptions for proof-of-concept development and are not yet validated against a specific 500 g-class PGV hardware configuration.

---

# 6. Model Assumptions

The current implementation intentionally uses a simplified model to isolate the guidance problem.

The simulations assume:

* Point-mass PGV dynamics
* Constant horizontal speed
* Constant sink rate
* Stationary target
* Flat terrain
* No wind
* No atmospheric disturbances
* No sensor noise
* No navigation uncertainty
* No actuator lag
* No detailed aerodynamic model
* Perfect state knowledge

The current implementation should therefore be interpreted as a **GNC proof-of-concept**, rather than a validated flight-control system.

---

# 7. Separation State

The PGV release state is obtained from the OpenRocket-derived cleaned release-state dataset.

| Parameter             | Symbol   |         Value |
| --------------------- | -------- | ------------: |
| Separation altitude   | $z_0$    | **1939.57 m** |
| Separation X position | $x_0$    |  **223.01 m** |
| Separation Y position | $y_0$    |    **0.14 m** |
| Initial PGV heading   | $\psi_0$ |     **0.07°** |

The initial target-relative geometry is therefore calculated from the release position and heading.

---

# 8. V0 — Mission Feasibility

**Notebook:** `mission_feasibility.ipynb`

V0 addresses the initial mission-level question:

> **Does the available separation altitude and assumed PGV glide capability provide sufficient horizontal range to reach the nominal 5 km target?**

V0 assumes ideal initial alignment toward the target and therefore evaluates **mission reachability**, rather than guidance performance.

## 8.1 Feasibility Analysis

The maximum available flight time is:

$$
t_{max}=\frac{z_0}{V_d}
$$

The corresponding maximum theoretical horizontal glide range is:

$$
D_{max}=V_h t_{max}
$$

The horizontal distance from the actual release point to the target is:

$$
D=
\sqrt{
(x_T-x_0)^2+
(y_T-y_0)^2
}
$$

Using the current release state:

| Quantity                             |         Value |
| ------------------------------------ | ------------: |
| Separation altitude                  | **1939.57 m** |
| Horizontal distance to target        | **4823.36 m** |
| Horizontal speed                     |  **6.20 m/s** |
| Sink rate                            |  **1.92 m/s** |
| Available flight time                | **1010.19 s** |
| Maximum theoretical horizontal range | **6263.20 m** |
| Theoretical range margin             | **1263.20 m** |

## 8.2 Result

**The nominal 5 km mission is reachable under the simplified idealized assumptions.**

The theoretical range margin is:

$$
6263.20-5000.00
=
1263.20\ \mathrm{m}
$$

V0 establishes mission reachability but does not account for imperfect separation heading, atmospheric disturbances, navigation uncertainty or terminal guidance effects.

## 8.3 Figures

![Mission Geometry 3D](../02_pgv_guidance/v0_mission_geometry_3d.png)

![Horizontal Mission Geometry](../02_pgv_guidance/v0_horizontal_mission_geometry.png)

![Altitude vs Horizontal Distance](../02_pgv_guidance/v0_altitude_vs_horizontal_distance.png)

---

# 9. V1 — Baseline Guidance

**Notebook:** `baseline_guidance.ipynb`

V1 extends the V0 feasibility model by introducing the actual rocket-provided separation heading and applying a closed-loop guidance strategy.

The baseline strategy is a:

> **Continuously updated line-of-sight pursuit guidance law with a bounded heading rate.**

The controller does not generate a complete trajectory in advance. Instead, it repeatedly evaluates the instantaneous PGV-target geometry, determines the heading correction required to point toward the target, applies the maximum turn-rate constraint, and propagates the vehicle state forward.

---

# 10. Guidance Law

## 10.1 Guidance Objective

The objective of the baseline guidance law is to continuously reduce the angular difference between:

1. the PGV's current heading, and
2. the instantaneous direction from the PGV to the stationary target.

The guidance loop can therefore be summarized as:

```text
PGV Position
     ↓
Relative Target Position
     ↓
Target Line-of-Sight
     ↓
Heading Error
     ↓
Turn-Rate Command
     ↓
Heading Update
     ↓
Position Update
     ↓
Recalculate Geometry
     ↓
Repeat
```

This makes the controller **closed-loop**, because the guidance command is recalculated from the newly updated state at every timestep.

---

## 10.2 Relative Target Geometry

Let the current PGV position be:

$$
(x,y)
$$

and the target position be:

$$
(x_T,y_T)
$$

The target-relative horizontal displacement is:

$$
\Delta x=x_T-x
$$

$$
\Delta y=y_T-y
$$

or:

$$
\mathbf{r}
=
\begin{bmatrix}
x_T-x\\
y_T-y
\end{bmatrix}
$$

The vector $\mathbf{r}$ points from the current PGV position toward the target.

---

## 10.3 Target Line-of-Sight Angle

The instantaneous target line-of-sight angle is obtained from the relative position vector:

$$
\boxed{
\lambda
=
\operatorname{atan2}
\left(
y_T-y,\;
x_T-x
\right)
}
$$

where $\lambda$ is measured in the horizontal East-North plane.

The use of `atan2` preserves the correct quadrant of the target direction.

Because the PGV position changes with time,

$$
x=x(t),\qquad y=y(t)
$$

the LOS angle also changes:

$$
\lambda=\lambda(t)
$$

Therefore, the desired direction is **not fixed at separation**. It is continuously recomputed as the PGV travels.

---

## 10.4 PGV Heading

The PGV heading is represented by:

$$
\psi
$$

The heading convention is:

$$
\psi=0^\circ
$$

for motion toward positive East, with positive angles measured counter-clockwise toward North.

Thus:

|     Heading | Direction |
| ----------: | --------- |
|   $0^\circ$ | East      |
|  $90^\circ$ | North     |
| $180^\circ$ | West      |
| $270^\circ$ | South     |

---

## 10.5 Heading Error

The difference between the instantaneous target LOS and the current PGV heading is:

$$
e_\psi=\lambda-\psi
$$

However, angles are periodic, so the raw difference is wrapped to the shortest equivalent angular displacement:

$$
\boxed{
e_\psi
=
\operatorname{wrap}
\left(
\lambda-\psi
\right)
}
$$

The wrapping operation constrains the heading error to:

$$
-\pi\leq e_\psi<\pi
$$

or equivalently:

$$
-180^\circ\leq e_\psi<180^\circ
$$

This ensures that the controller always selects the shortest direction of rotation toward the target LOS.

---

## 10.6 Commanded Heading Rate

The implemented heading-rate command is:

$$
\boxed{
\dot{\psi}_{cmd}
=
\operatorname{clip}
\left(
\frac{e_\psi}{\Delta t},
-\dot{\psi}_{max},
+\dot{\psi}_{max}
\right)
}
$$

where:

* $e_\psi$ is the instantaneous heading error
* $\Delta t$ is the simulation timestep
* $\dot{\psi}_{max}$ is the maximum allowable turn rate

For the baseline simulation:

$$
\Delta t=0.10\ \mathrm{s}
$$

and:

$$
\dot{\psi}_{max}=6.00^\circ/\mathrm{s}
$$

The term

$$
\frac{e_\psi}{\Delta t}
$$

represents the angular rate required to remove the current heading error within one timestep, before the physical/modelled turn-rate constraint is applied.

---

## 10.7 Equivalent Proportional Gain

The unsaturated part of the command can be written as:

$$
\frac{e_\psi}{\Delta t}
=
K_p e_\psi
$$

where:

$$
K_p=\frac{1}{\Delta t}
$$

For:

$$
\Delta t=0.10\ \mathrm{s}
$$

the equivalent gain is:

$$
\boxed{
K_p=10.00\ \mathrm{s}^{-1}
}
$$

This is an equivalent representation of the implemented discrete-time command. It is not an independently tuned gain.

The exact implemented guidance law remains:

$$
\dot{\psi}_{cmd}
=
\operatorname{clip}
\left(
\frac{e_\psi}{\Delta t},
-\dot{\psi}_{max},
+\dot{\psi}_{max}
\right)
$$

---

## 10.8 Turn-Rate Saturation

The commanded heading rate is constrained by:

$$
-\dot{\psi}_{max}
\leq
\dot{\psi}_{cmd}
\leq
+\dot{\psi}_{max}
$$

This produces two operating regions.

### Saturated Region

If:

$$
\left|
\frac{e_\psi}{\Delta t}
\right|
>
\dot{\psi}_{max}
$$

then:

$$
\dot{\psi}_{cmd}
=
\pm\dot{\psi}_{max}
$$

The PGV therefore turns at its maximum assumed rate.

### Unsaturated Region

If:

$$
\left|
\frac{e_\psi}{\Delta t}
\right|
\leq
\dot{\psi}_{max}
$$

then:

$$
\dot{\psi}_{cmd}
=
\frac{e_\psi}{\Delta t}
$$

The transition occurs when:

$$
|e_\psi|
=
\dot{\psi}_{max}\Delta t
$$

For the baseline parameters:

$$
|e_\psi|
=
6.00\times0.10
=
0.60^\circ
$$

Therefore:

$$
\boxed{
|e_\psi|>0.60^\circ
\Rightarrow
|\dot{\psi}_{cmd}|=6.00^\circ/\mathrm{s}
}
$$

while:

$$
\boxed{
|e_\psi|\leq0.60^\circ
\Rightarrow
\dot{\psi}_{cmd}
=
\frac{e_\psi}{\Delta t}
}
$$

This explains the rate-limited initial turning behaviour observed in the simulation.

---

## 10.9 Heading Propagation

Once the heading-rate command has been calculated, the PGV heading is advanced by one timestep:

$$
\boxed{
\psi_{k+1}
=
\operatorname{wrap}
\left(
\psi_k+
\dot{\psi}_{cmd,k}\Delta t
\right)
}
$$

The wrapping step maintains the heading representation within the chosen angular interval.

---

## 10.10 Horizontal Motion Model

The PGV horizontal speed is assumed constant:

$$
V_h=6.20\ \mathrm{m/s}
$$

The East and North components of velocity are:

$$
V_x=V_h\cos\psi
$$

$$
V_y=V_h\sin\psi
$$

Therefore:

$$
\boxed{
\frac{dx}{dt}
=
V_h\cos\psi
}
$$

$$
\boxed{
\frac{dy}{dt}
=
V_h\sin\psi
}
$$

The corresponding discrete-time update is:

$$
\boxed{
x_{k+1}
=
x_k+
V_h\cos\left(\psi_{k+1}\right)\Delta t
}
$$

$$
\boxed{
y_{k+1}
=
y_k+
V_h\sin\left(\psi_{k+1}\right)\Delta t
}
$$

Thus, the guidance controller directly modifies the heading $\psi$, while the heading determines the resulting horizontal ground track.

---

## 10.11 Vertical Motion Model

The vertical motion is modelled independently of horizontal guidance.

The sink-rate magnitude is:

$$
V_d=1.92\ \mathrm{m/s}
$$

with vertical velocity:

$$
V_z=-V_d=-1.92\ \mathrm{m/s}
$$

The vertical equation is:

$$
\boxed{
\frac{dz}{dt}=V_z
}
$$

and the discrete update is:

$$
\boxed{
z_{k+1}
=
z_k+
V_z\Delta t
}
$$

The present model therefore assumes a constant descent rate throughout the flight.

No dynamic relationship between bank angle, aerodynamic force and sink rate is included.

---

## 10.12 Complete Guidance and Propagation Loop

The complete V1 algorithm can be expressed as:

### Step 1 — Determine target-relative position

$$
\Delta x_k=x_T-x_k
$$

$$
\Delta y_k=y_T-y_k
$$

### Step 2 — Determine instantaneous LOS

$$
\lambda_k
=
\operatorname{atan2}
\left(
\Delta y_k,\Delta x_k
\right)
$$

### Step 3 — Determine heading error

$$
e_{\psi,k}
=
\operatorname{wrap}
\left(
\lambda_k-\psi_k
\right)
$$

### Step 4 — Calculate the heading-rate command

$$
\dot{\psi}_{cmd,k}
=
\operatorname{clip}
\left(
\frac{e_{\psi,k}}{\Delta t},
-\dot{\psi}_{max},
+\dot{\psi}_{max}
\right)
$$

### Step 5 — Update heading

$$
\psi_{k+1}
=
\operatorname{wrap}
\left(
\psi_k+
\dot{\psi}_{cmd,k}\Delta t
\right)
$$

### Step 6 — Update position

$$
x_{k+1}
=
x_k+
V_h\cos\left(\psi_{k+1}\right)\Delta t
$$

$$
y_{k+1}
=
y_k+
V_h\sin\left(\psi_{k+1}\right)\Delta t
$$

$$
z_{k+1}
=
z_k+
V_z\Delta t
$$

### Step 7 — Repeat

The newly calculated state becomes the input to the next guidance cycle.

This repeated feedback process allows the PGV to continuously adapt its ground track to the changing target LOS.

---

# 11. Initial Separation Geometry

The actual release state produces:

| Quantity               |         Value |
| ---------------------- | ------------: |
| Separation X position  |  **223.01 m** |
| Separation Y position  |    **0.14 m** |
| Separation altitude    | **1939.57 m** |
| Initial PGV heading    |     **0.07°** |
| Initial target bearing |    **38.46°** |
| Initial heading error  |    **38.39°** |

The initial heading mismatch is therefore:

$$
e_{\psi,0}=38.39^\circ
$$

The corresponding unconstrained turn-rate demand is:

$$
\frac{38.39^\circ}{0.10\ \mathrm{s}}
=
383.90^\circ/\mathrm{s}
$$

which is much greater than:

$$
\dot{\psi}_{max}=6.00^\circ/\mathrm{s}
$$

Therefore, the initial command is rate saturated:

$$
\boxed{
\dot{\psi}_{cmd}=+6.00^\circ/\mathrm{s}
}
$$

---

# 12. Initial Heading Correction

The initial correction produces the following state:

| Quantity                        |                 Value |
| ------------------------------- | --------------------: |
| Correction completion time      |            **6.40 s** |
| Position at heading alignment   | **(259.77, 12.98) m** |
| Altitude at heading alignment   |         **1927.29 m** |
| Altitude lost during correction |           **12.29 m** |
| Final heading error             |             **0.00°** |

The ideal rate-limited estimate is:

$$
t_{turn}
\approx
\frac{|e_{\psi,0}|}
{\dot{\psi}_{max}}
$$

giving:

$$
t_{turn}
\approx
\frac{38.39}
{6.00}
\approx
6.40\ \mathrm{s}
$$

The PGV therefore corrects the initial heading mismatch shortly after separation while retaining most of the available insertion altitude.

---

# 13. V1 Figures

![V1 Mission Geometry 3D](../02_pgv_guidance/v1_mission_geometry_3d.png)

![V1 Horizontal Mission Geometry](../02_pgv_guidance/v1_horizontal_mission_geometry.png)

![V1 Heading vs Time](../02_pgv_guidance/v1_heading_vs_time.png)

![V1 Heading Correction Magnified](../02_pgv_guidance/v1_horizontal_mission_geometry_magnified.png)

![V1 Heading Correction Geometry](../02_pgv_guidance/v1_heading_correction.png)

![V1 Altitude vs Horizontal Distance](../02_pgv_guidance/v1_altitude_vs_horizontal_distance.png)

![V1 Range vs Time](../02_pgv_guidance/v1_range_vs_time.png)

---

# 14. Target-Vicinity Result

The primary guidance loop uses a horizontal target-vicinity tolerance of:

$$
r_T=10.00\ \mathrm{m}
$$

The first entry into this region occurs at:

| Quantity                    |         Value |
| --------------------------- | ------------: |
| Time at target vicinity     |  **776.90 s** |
| Horizontal range error      |    **9.44 m** |
| Altitude at target vicinity |  **447.92 m** |
| Vertical velocity           | **−1.92 m/s** |

The **9.44 m result represents horizontal target-vicinity error**, not landing accuracy.

The PGV is still **447.92 m above ground** when the target-vicinity condition is reached.

Therefore, this result demonstrates close horizontal target acquisition under the baseline guidance model, but does not represent touchdown accuracy.

---

# 15. Ground-Termination Analysis

The target-vicinity condition intentionally stops the primary guidance loop when the PGV enters the 10 m horizontal tolerance.

A separate propagation is therefore performed without this early target-vicinity termination and continues until:

$$
z=0
$$

For comparison, an uncorrected case holds the initial separation heading fixed throughout the flight.

## 15.1 Terminal Results

| Quantity             |        Guided |   Uncorrected |
| -------------------- | ------------: | ------------: |
| Terminal X           | **3977.91 m** | **6486.20 m** |
| Terminal Y           | **2955.80 m** |    **7.24 m** |
| Ground miss distance |   **49.42 m** | **3890.74 m** |
| Flight time          | **1010.20 s** | **1010.19 s** |

The terminal miss-distance reduction is:

$$
\text{Reduction}
=
\frac{
3890.74-49.42
}{
3890.74
}
\times100
$$

Therefore:

$$
\boxed{
\text{Reduction}=98.73\%
}
$$

The result represents a comparison between two trajectories generated under the same simplified vehicle assumptions.

> **Metric distinction:** The **9.44 m** result is horizontal target-vicinity error at **447.92 m altitude**, whereas the **49.42 m** result is horizontal miss distance after a separate propagation to ground altitude. These are different metrics.

![V1 Ground-Termination Analysis](../02_pgv_guidance/v1_ground_termination.png)

---

# 16. Key Results

| Result                                |         Value |
| ------------------------------------- | ------------: |
| Rocket apogee                         | **1939.57 m** |
| Mission range                         | **5000.00 m** |
| Release-to-target horizontal distance | **4823.36 m** |
| Maximum theoretical PGV range         | **6263.20 m** |
| Theoretical range margin              | **1263.20 m** |
| Initial PGV heading                   |     **0.07°** |
| Initial target bearing                |    **38.46°** |
| Initial heading error                 |    **38.39°** |
| Heading correction time               |    **6.40 s** |
| Altitude at heading alignment         | **1927.29 m** |
| Horizontal target-vicinity error      |    **9.44 m** |
| Altitude at target vicinity           |  **447.92 m** |
| Guided ground miss distance           |   **49.42 m** |
| Uncorrected ground miss distance      | **3890.74 m** |
| Terminal miss-distance reduction      |    **98.73%** |

---

# 17. Data Source and Precision

All current PGV guidance calculations use:

```text
01_rocket_sizing/release_states_data_clean.csv
```

The cleaned dataset is treated as the **canonical downstream data source** for the present analysis.

Calculations are performed using the full floating-point values from the dataset, with numerical values rounded only for final display.

Earlier development versions mixed raw and cleaned release-state data and, in some cases, reused rounded intermediate values. This produced small inconsistencies in separation position, initial heading and downstream results.

The current values reported in this document are based on the corrected full-precision workflow.

---

# 18. Current Limitations

The current implementation is deliberately simplified and does not yet model:

* Wind and atmospheric disturbances
* Sensor noise
* Navigation uncertainty
* Closed-loop state estimation
* Actuator dynamics and lag
* Detailed parafoil aerodynamics
* Bank-angle dynamics
* Bank-induced sink-rate coupling
* Variable sink rate
* Pendulum payload coupling
* Terrain variation
* Hardware-specific PGV performance validation
* Dedicated terminal energy management

The PGV parameters should therefore be interpreted as **baseline simulation assumptions**.

---

# 19. Future Development

## Sensitivity Analysis

Evaluate the effect of variations in:

* Separation altitude
* Separation position
* Initial heading
* Horizontal speed
* Sink rate
* Maximum heading rate

## Monte Carlo / CEP Analysis

Introduce uncertainty into the initial state and model parameters and evaluate:

* Miss-distance distributions
* Guidance robustness
* Statistical terminal performance
* CEP-style measures

## Wind and Disturbance Modelling

Evaluate the guidance response under:

* Crosswind
* Headwind
* Tailwind
* Variable wind profiles

## Higher-Fidelity PGV Dynamics

Introduce:

* Bank dynamics
* Variable sink rate
* Aerodynamic coupling
* More realistic parafoil response
* Payload pendulum dynamics

## Moving-Target Extension

For a non-stationary target, extend the baseline formulation toward full closing-velocity-weighted Proportional Navigation using explicit relative velocity and LOS-rate information.

## Navigation and Control

Future GNC development may include:

* Sensor modelling
* State estimation
* Extended Kalman filtering
* Actuator dynamics
* Navigation uncertainty

## MATLAB Cross-Verification

The Python implementation can later be independently reproduced and verified using MATLAB to compare:

* Guidance logic
* Heading response
* Trajectory propagation
* Terminal performance

---

# 20. Conclusion

The current RAPID development phase establishes a baseline **rocket-assisted payload delivery concept** using an OpenRocket-derived insertion state and a simplified point-mass PGV guidance model.

The launch vehicle reaches an apogee of **1939.57 m**, which is used as the nominal PGV release altitude.

For the current release state, the horizontal distance from the release point to the target is **4823.36 m**, while the theoretical maximum horizontal glide range is **6263.20 m**. This provides a theoretical range margin of **1263.20 m**.

The V1 guidance model introduces an actual separation heading of **0.07°** against an initial target bearing of **38.46°**, resulting in an initial heading error of **38.39°**.

The guidance law continuously calculates the instantaneous target line-of-sight, determines the wrapped heading error, and commands a bounded heading rate according to:

$$
\dot{\psi}_{cmd}
=
\operatorname{clip}
\left(
\frac{e_\psi}{\Delta t},
-\dot{\psi}_{max},
+\dot{\psi}_{max}
\right)
$$

with a simulation timestep of **0.10 s** and a maximum heading rate of **6.00 °/s**.

The initial heading correction is completed in approximately **6.40 s**, with the PGV reaching heading alignment at an altitude of **1927.29 m**.

The PGV subsequently enters the defined **10 m horizontal target-vicinity region** with a horizontal error of **9.44 m** at **447.92 m altitude**. This is a target-acquisition metric rather than a touchdown-accuracy measurement.

A separate ground-termination propagation produces a guided ground miss distance of **49.42 m**, compared with **3890.74 m** for the uncorrected case. This corresponds to a **98.73% reduction in terminal miss distance relative to the uncorrected baseline** under the same simplified model.

The present work remains a **simulation-based GNC proof-of-concept**. The V0–V1 framework establishes a baseline for future sensitivity analysis, Monte Carlo robustness studies, wind modelling, higher-fidelity PGV dynamics, navigation estimation and terminal guidance development.

---

# Related Repository Sections

### Rocket Sizing

[`../01_rocket_sizing/`](../01_rocket_sizing/)

Contains:

* OpenRocket vehicle configuration
* Open Rocket Data set

### PGV Guidance

[`../02_pgv_guidance/`](../02_pgv_guidance/)

Contains:

* Python simulation notebooks
* Mission geometry plots

# References

1. Murray, J. E., Sim, A. G., Neufeld, D. C., Rennich, P. K., Norris, S. R., and Hughes, W. S., *Further Development and Flight Test of an Autonomous Precision Landing System Using a Parafoil*, NASA Technical Memorandum NASA-TM-4599, AIAA Paper 94-2141, 1994.  
   [NASA Technical Reports Server](https://ntrs.nasa.gov/citations/19940029489)

2. Sim, A. G., Murray, J. E., Neufeld, D. C., and Reed, R. D., *Development and Flight Test of a Deployable Precision Landing System*, NASA Technical Reports Server, 1994.  
   [NASA Technical Reports Server](https://ntrs.nasa.gov/citations/19950037644)

3. Stein, J. M., et al., *An Overview of the Guided Parafoil System Derived from X-38 Experience*, NASA Technical Reports Server, 2005.  
   The work describes a large guided parafoil system and discusses parafoil aerodynamic modelling, including glide ratio and turn-performance characteristics used in GN&C development.  
   [NASA Technical Reports Server](https://ntrs.nasa.gov/citations/20070026249)

4. Defence Research and Development Organisation (DRDO), *Controlled Aerial Delivery System (CADS)*, DRDO Products for Export, 2025.  
   The CADS documentation describes an autonomous ram-air parafoil delivery system and reports a maximum no-wind glide ratio of approximately 3:1 for its 500 kg and 1000 kg systems.  
   [DRDO CADS](https://drdo.gov.in/drdo/en/offerings/products/controlled-aerial-delivery-system-cads)  
   [DRDO Products for Export 2025](https://drdo.gov.in/drdo/sites/default/files/inline-files/CompendiumProductForExport2025.pdf)

5. Ermolli, L., Rimani, J., Schutte, A., and Quadrelli, M. B., *Aero Maneuvering Dynamics and Control for Precision Landing on Titan*, NASA Technical Reports Server, 2019.  
   The study develops parafoil dynamic models ranging from point-mass to multibody representations and investigates autonomous parafoil turning and environmental-parameter estimation.  
   [NASA Technical Reports Server](https://ntrs.nasa.gov/citations/20210005963)

---

## Parameter Basis

The PGV parameters used in the present study are **provisional baseline modelling assumptions**, rather than experimentally measured characteristics of a specific 0.50 kg PGV.

Published guided-parafoil systems demonstrate the feasibility of autonomous navigation and precision delivery, while reported glide-performance values provide a reference for the order of magnitude of the present assumptions. For example, DRDO's Controlled Aerial Delivery System (CADS) reports a maximum no-wind glide ratio of approximately **3:1** for a substantially larger payload class, while NASA guided-parafoil programs document the use of parafoil aerodynamic models incorporating glide ratio and turn performance for autonomous guidance and control.

Based on this published context, the present study adopts:

- Horizontal speed: **6.20 m/s**
- Sink rate: **1.92 m/s**
- Effective glide ratio: **3.23:1**
- Maximum heading rate: **6.00 °/s**

The glide ratio is calculated directly from the two assumed velocity components:

$$
\frac{V_h}{V_d}
=
\frac{6.20}{1.92}
=
3.23:1
$$

The minimum turn radius is subsequently calculated from the assumed horizontal speed and maximum heading rate:

$$
R_{min}
=
\frac{V_h}{\dot{\psi}_{max}}
=
59.22\ \mathrm{m}
$$

These values are therefore **selected simulation parameters informed by published guided-parafoil performance**, not values claimed to have been measured on the proposed PGV. Validation against an actual PGV configuration would be required before treating them as vehicle-performance characteristics.