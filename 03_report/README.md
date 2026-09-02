# RAPID — Technical Report

## Rocket-Assisted Payload Insertion and Descent

This directory contains the consolidated technical documentation for **RAPID (Rocket-Assisted Payload Insertion and Descent)**.

RAPID is a concept for **rocket-assisted point-to-point payload delivery**, where a launch vehicle inserts a **Payload Glide Vehicle (PGV)** to altitude. Following separation, the PGV autonomously guides toward a designated ground target.

The current phase focuses on a **simulation-based Guidance, Navigation and Control (GNC) proof-of-concept**, supported by OpenRocket-based rocket sizing and a simplified point-mass PGV model.

> **Project status:** Proof-of-concept simulation. The current results are not hardware validation or flight-ready performance claims.

---

## 1. Report Purpose

The purpose of this section is to consolidate the technical development of the RAPID concept into a single engineering narrative.

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
Trajectory / Terminal Analysis
```

The central question investigated in the current phase is:

> **Given a plausible rocket-provided separation state, can a simplified PGV autonomously correct its trajectory and approach a stationary target 5 km from the launch location?**

---

## 2. Mission Definition

| Parameter                       |                            Value |
| ------------------------------- | -------------------------------: |
| Target type                     |                       Stationary |
| Target X position               |                        4000.00 m |
| Target Y position               |                        3000.00 m |
| Target range from launch origin |                        5000.00 m |
| PGV + payload mass              |                          0.50 kg |
| Launch profile                  |                         Vertical |
| PGV baseline                    |               Steerable parafoil |
| Launch vehicle role             |               Altitude insertion |
| PGV role                        | Autonomous guidance and delivery |

### Coordinate System

```text
X → East
Y → North
Z → Up
```

Target position:

```text
(4000.00, 3000.00, 0.00) m
```

---

## 3. System Architecture

### Rocket-Assisted Insertion

The launch vehicle performs:

* Vertical launch
* Payload insertion
* Ascent to the required altitude
* PGV release near apogee

The rocket is treated primarily as the **insertion platform**.

### Payload Glide Vehicle

Following separation, the PGV performs:

* Autonomous navigation
* Heading correction
* Guided glide
* Terminal positioning

```text
Launch
  ↓
Rocket Ascent
  ↓
Apogee / PGV Separation
  ↓
Initial Heading Correction
  ↓
Guided Glide
  ↓
Target Vicinity
  ↓
Ground-Termination Analysis
```

---

## 4. Rocket Sizing and Insertion

The launch vehicle was designed and simulated using **OpenRocket**.

The rocket-sizing stage establishes a plausible insertion state for the PGV guidance simulation.

### 4.1 Baseline Rocket Configuration

| Parameter               | Value         |
| ----------------------- | ------------- |
| Nose cone               | Ogive         |
| Nose cone length        | 250 mm        |
| Body tube length        | 1000 mm       |
| Total rocket length     | 1250 mm       |
| Outer diameter          | 100 mm        |
| Number of fins          | 3             |
| Fin geometry            | Trapezoidal   |
| Fin material            | Balsa         |
| Fin thickness           | 3 mm          |
| Root chord              | 180 mm        |
| Tip chord               | 80 mm         |
| Fin span                | 120 mm        |
| Sweep                   | 120 mm        |
| Motor mount diameter    | 54 mm         |
| Motor mount length      | 300 mm        |
| Payload module mass     | 0.50 kg       |
| Payload module diameter | 80 mm         |
| Payload module length   | 300 mm        |
| Baseline motor          | AeroTech J99N |

### 4.2 OpenRocket Results

| Parameter                   |          Value |
| --------------------------- | -------------: |
| Apogee altitude             |  **1939.57 m** |
| Maximum velocity            | **181.74 m/s** |
| Maximum acceleration        | **74.41 m/s²** |
| Maximum Mach number         |       **0.54** |
| Time to apogee              |    **18.49 s** |
| Launch rod exit velocity    |  **12.09 m/s** |
| Total ballistic flight time |    **49.76 s** |
| Ground impact velocity      |  **72.35 m/s** |

The rocket simulation is used primarily to characterize the **ascent and PGV separation state**.

---

## 5. PGV Baseline Capability Model

The PGV is represented using a simplified point-mass kinematic model.

| Parameter                       | Symbol                   |    Value |
| ------------------------------- | ------------------------ | -------: |
| Horizontal velocity             | \(V_h\)                  | 6.20 m/s |
| Sink rate                       | \(V_d\)                  | 1.92 m/s |
| Glide ratio                     | \(V_h/V_d\)              | 3.23 : 1 |
| Maximum heading rate            | \(\dot{\psi}_{max}\)     | 6.00 °/s |
| Combined PGV + payload mass     | \(m\)                    |  0.50 kg |
| Approximate minimum turn radius | \(V_h/\dot{\psi}_{max}\) |  59.22 m |

> **Validation status:** The PGV parameters are provisional modelling assumptions used for the current proof-of-concept. They are not experimentally validated values for a specific 500 g-class PGV.

---

## 6. Separation State

The OpenRocket trajectory provides the initial state for the PGV guidance model.

| Parameter             | Symbol     |         Value |
| --------------------- | ---------- | ------------: |
| Separation altitude   | \(z_0\)    | **1939.57 m** |
| Separation X position | \(x_0\)    |  **223.00 m** |
| Separation Y position | \(y_0\)    |    **0.10 m** |
| Initial PGV heading   | \(\psi_0\) |     **0.10°** |

---

## 7. V0 — Mission Feasibility

**Notebook:** `mission_feasibility.ipynb`

V0 evaluates whether the available altitude and assumed PGV glide performance are theoretically sufficient for the nominal 5 km mission.

The maximum available flight time is:

$$
t_{max} = \frac{z_0}{V_d}
$$

The corresponding maximum theoretical horizontal range is:

$$
D_{max} = V_h t_{max}
$$

### V0 Results

| Quantity                  |         Value |
| ------------------------- | ------------: |
| Separation altitude       | **1939.57 m** |
| Maximum flight time       | **1010.19 s** |
| Maximum theoretical range | **6263.18 m** |
| Required mission range    | **5000.00 m** |
| Theoretical range margin  | **1263.18 m** |

### Result

**The nominal 5 km mission is theoretically reachable under the baseline assumptions.**

V0 assumes ideal initial alignment toward the target and therefore establishes **mission feasibility**, not guidance performance.

---

## 8. V1 — Baseline Guidance

**Notebook:** `baseline_guidance.ipynb`

V1 introduces the imperfect rocket-provided separation state and evaluates whether the PGV can autonomously correct its trajectory toward the stationary target.

The baseline method uses a **continuously updated, turn-rate-limited pursuit guidance law**.

### 8.1 Motion Model

$$
\frac{dx}{dt}=V_h\cos\psi
$$

$$
\frac{dy}{dt}=V_h\sin\psi
$$

$$
\frac{dz}{dt}=-V_d
$$

### 8.2 Guidance Law

$$
\text{bearing}=\operatorname{atan2}(y_T-y,\ x_T-x)
$$

$$
\text{heading error}=\operatorname{wrap}(\text{bearing}-\psi)
$$

$$
\dot{\psi}_{cmd}
=
\operatorname{clamp}
\left(
K_p\,\text{heading error},
-\dot{\psi}_{max},
+\dot{\psi}_{max}
\right)
$$

with:

$$
K_p=1.00
$$

$$
\dot{\psi}_{max}=6.00^\circ/s
$$

The target bearing is continuously updated as the PGV moves.

---

## 9. Initial Heading Correction

The initial separation state produces a significant heading mismatch.

| Quantity                  |      Value |
| ------------------------- | ---------: |
| Initial PGV heading       |  **0.10°** |
| Initial bearing to target | **38.46°** |
| Initial heading error     | **38.36°** |

### Heading Correction Results

| Parameter                                 |         Value |
| ----------------------------------------- | ------------: |
| Correction completion time                |    **6.40 s** |
| X position at heading alignment           |  **259.77 m** |
| Y position at heading alignment           |   **12.98 m** |
| Altitude at heading alignment             | **1927.29 m** |
| Horizontal displacement during correction |   **38.94 m** |
| Altitude lost during correction           |   **12.29 m** |
| Final heading                             |    **38.46°** |
| Desired heading                           |    **38.46°** |
| Final heading error                       |     **0.00°** |

---

## 10. Target-Vicinity Performance

The guidance simulation reaches close horizontal proximity to the target.

| Quantity                         |         Value |
| -------------------------------- | ------------: |
| Time at target vicinity          |  **776.90 s** |
| Horizontal target-vicinity error |    **9.44 m** |
| Altitude at target vicinity      |  **447.92 m** |
| Vertical velocity                | **−1.92 m/s** |

### Interpretation

The **9.44 m value is a horizontal target-vicinity error**, not a landing miss distance.

At this point, the PGV is still approximately **447.92 m above ground**.

Therefore, this result demonstrates close horizontal target acquisition but does **not** represent touchdown accuracy.

---

## 11. Ground-Termination Analysis

A separate propagation of the baseline model is evaluated until:

$$
z=0
$$

The resulting terminal positions are:

| Quantity             |        Guided |   Uncorrected |
| -------------------- | ------------: | ------------: |
| Terminal X           | **3977.89 m** | **6486.19 m** |
| Terminal Y           | **2955.78 m** |   **11.03 m** |
| Ground miss distance |   **49.44 m** | **3887.81 m** |
| Flight time          | **1010.20 s** | **1010.19 s** |

The terminal miss-distance reduction is:

$$
\text{Reduction}
=
\frac{3887.81-49.44}{3887.81}\times100
=
98.73\%
$$

> The **98.73% reduction** is a comparison against the uncorrected trajectory under the same simplified baseline model. It is not a real-world accuracy claim.

---

## 12. Key Results

| Result                           |         Value |
| -------------------------------- | ------------: |
| Rocket apogee                    | **1939.57 m** |
| Mission range                    | **5000.00 m** |
| Maximum theoretical PGV range    | **6263.18 m** |
| Theoretical range margin         | **1263.18 m** |
| Initial heading error            |    **38.36°** |
| Heading correction time          |    **6.40 s** |
| Altitude at heading alignment    | **1927.29 m** |
| Horizontal target-vicinity error |    **9.44 m** |
| Altitude at target vicinity      |  **447.92 m** |
| Guided ground miss distance      |   **49.44 m** |
| Uncorrected ground miss distance | **3887.81 m** |
| Terminal miss-distance reduction |    **98.73%** |

---

## 13. Modelling Assumptions

The baseline model assumes:

* Point-mass PGV dynamics
* Constant horizontal velocity
* Constant sink rate
* Stationary target
* Flat terrain
* No wind
* No atmospheric disturbances
* No sensor noise
* Perfect state knowledge
* No actuator lag
* No detailed parafoil aerodynamic model
* No bank-induced sink-rate variation

These assumptions intentionally isolate the guidance problem before introducing higher-fidelity dynamics and environmental uncertainty.

---

## 14. Current Limitations

The current implementation does not yet include:

* Wind modelling
* Atmospheric disturbances
* Navigation uncertainty
* Sensor noise
* State estimation
* Actuator dynamics
* Detailed parafoil aerodynamics
* Bank dynamics
* Variable sink rate
* Pendulum payload dynamics
* Terrain variation
* Hardware-specific PGV validation
* Dedicated terminal energy management

The current terminal behaviour therefore remains a result of the simplified model rather than a validated flight-performance prediction.

---

## 15. Future Development

### Sensitivity Analysis

Evaluate the effect of variations in:

* Separation altitude
* Separation position
* Initial heading
* Horizontal speed
* Sink rate
* Maximum heading rate

### Monte Carlo / CEP Analysis

Introduce uncertainty and evaluate:

* Miss-distance distributions
* Guidance robustness
* Statistical terminal accuracy
* CEP-style performance

### Wind and Disturbance Modelling

Introduce:

* Crosswind
* Headwind
* Tailwind
* Variable wind profiles

### Higher-Fidelity PGV Dynamics

Introduce:

* Bank dynamics
* Variable sink rate
* Aerodynamic coupling
* Parafoil response
* Payload pendulum dynamics

### Navigation and Control

Future GNC development may include:

* Sensor models
* State estimation
* Extended Kalman filtering
* Actuator dynamics

### MATLAB Cross-Verification

The Python implementation can be independently reproduced using MATLAB to verify:

* Guidance logic
* Heading response
* Trajectory propagation
* Terminal performance

---

## 16. Conclusion

The current RAPID development phase establishes a baseline **rocket-assisted payload guidance concept** using a physically grounded rocket insertion state and a simplified PGV model.

The OpenRocket simulation produces an apogee of **1939.57 m**, providing sufficient theoretical altitude for a **5000.00 m** mission under the baseline PGV glide assumptions.

The V0 model establishes a maximum theoretical horizontal range of **6263.18 m**, giving a theoretical range margin of **1263.18 m**.

The V1 baseline guidance model starts with an initial heading error of **38.36°** and completes the initial heading correction within **6.40 s**.

The PGV reaches a horizontal target-vicinity error of **9.44 m** while still at **447.92 m altitude**. This is a target-vicinity metric and not a landing-error measurement.

When separately propagated to ground altitude, the guided case produces a **49.44 m ground miss distance**, compared with **3887.81 m** for the uncorrected trajectory.

This corresponds to a **98.73% reduction in terminal miss distance relative to the uncorrected baseline**.

The work remains a **simulation-based engineering proof-of-concept**. The results provide a baseline for future sensitivity analysis, Monte Carlo robustness studies, wind modelling, higher-fidelity PGV dynamics, navigation estimation, and terminal guidance development.

---

## Related Repository Sections

### Rocket Sizing

`../01_rocket_sizing/`

Contains:

* OpenRocket vehicle configuration
* Rocket geometry
* Motor selection
* Rocket trajectory data
* Cleaned simulation data

### PGV Guidance

`../02_pgv_guidance/`

Contains:

* V0 — Mission Feasibility
* V1 — Baseline Guidance
* Guidance notebooks
* Mission geometry plots
* Heading correction plots
* Terminal analysis
