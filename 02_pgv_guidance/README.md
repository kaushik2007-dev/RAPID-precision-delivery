# Payload Glide Vehicle Guidance

This directory contains the **Guidance, Navigation and Control (GNC) proof-of-concept** for the Payload Glide Vehicle (PGV) developed as part of **RAPID — Rocket Assisted Payload Insertion and Descent**.

The launch vehicle provides the PGV with an initial separation state near apogee. Following separation, the PGV autonomously navigates toward a stationary ground target.

The current development is organized into two stages:

```text
Rocket-Provided Separation State
              │
              ▼
     V0 — Mission Feasibility
              │
              ▼
   V1 — Baseline Guidance
              │
              ▼
Future — Robustness & Higher-Fidelity GNC
```

---

# Mission Scenario

The nominal mission considers a stationary target located 5 km from the launch origin.

| Parameter | Value |
|---|---:|
| Target X position | 4000.00 m |
| Target Y position | 3000.00 m |
| Target range from launch origin | 5000.00 m |
| Separation altitude | 1939.57 m |
| Separation X position | 223.00 m |
| Separation Y position | 0.10 m |
| Target type | Stationary |
| Launch profile | Vertical |
| PGV configuration | Steerable parafoil |
| PGV + payload mass | 0.50 kg |

The simulations use an East-North-Up Cartesian coordinate system:

```text
X → East
Y → North
Z → Up
```

The nominal target location is:

```text
Target = (4000.00, 3000.00, 0.00) m
```

---

# PGV Baseline Model

The PGV is represented using a simplified point-mass kinematic model.

| Parameter | Symbol | Value |
|---|---|---:|
| Horizontal speed | Vh | 6.20 m/s |
| Sink rate | Vd | 1.92 m/s |
| Glide ratio | Vh / Vd | 3.23 : 1 |
| Maximum heading rate | ψ̇max | 6.00 °/s |
| Maximum heading rate | ψ̇max | 0.10 rad/s |
| Minimum turn radius | Vh / ψ̇max | 59.22 m |
| PGV + payload mass | m | 0.50 kg |

> **Validation status:** The PGV parameters used in the current simulations are provisional modelling assumptions intended for proof-of-concept development. They are not yet validated against a specific 500 g-class PGV hardware configuration.

---

# Model Assumptions

The current implementation intentionally uses a simplified model to isolate the guidance problem.

The simulations assume:

- Point-mass PGV dynamics
- Constant horizontal speed
- Constant sink rate
- Stationary target
- Flat terrain
- No wind
- No atmospheric disturbances
- No sensor noise
- No navigation uncertainty
- No actuator lag
- No detailed aerodynamic model
- Perfect state knowledge

The current implementation should therefore be interpreted as a **GNC proof-of-concept**, rather than a validated flight-control system.

---

# V0 — Mission Feasibility

**Notebook:** `mission_feasibility.ipynb`

## Objective

V0 addresses the initial mission-level question:

> **Does the available separation altitude and assumed PGV glide capability provide sufficient horizontal range to reach the nominal 5 km target?**

The V0 model assumes ideal initial alignment toward the target and therefore evaluates **mission reachability**, rather than guidance performance.

---

## Feasibility Analysis

The maximum available flight time is estimated using:

```text
t_max = z₀ / Vd
```

The corresponding theoretical horizontal range is:

```text
D_max = Vh × t_max
```

Using the baseline PGV parameters:

| Quantity | Value |
|---|---:|
| Separation altitude | 1939.57 m |
| Horizontal speed | 6.20 m/s |
| Sink rate | 1.92 m/s |
| Maximum flight time | 1010.19 s |
| Maximum theoretical horizontal range | 6263.18 m |
| Required mission range | 5000.00 m |
| Theoretical range margin | 1263.18 m |

## Result

**The nominal 5 km mission is reachable under the simplified idealized assumptions.**

V0 establishes that the assumed PGV performance provides sufficient horizontal range. It does not account for imperfect separation conditions or trajectory correction.

---

## Mission Geometry — 3D

![Mission Geometry 3D](v0_mission_geometry_3d.png)

---

## Horizontal Mission Geometry

![Horizontal Mission Geometry](v0_horizontal_mission_geometry.png)

---

## Altitude vs Horizontal Distance

![Altitude vs Horizontal Distance](v0_altitude_vs_horizontal_distance.png)

---

# V1 — Baseline Guidance

**Notebook:** `baseline_guidance.ipynb`

## Objective

V1 extends the idealized feasibility model by introducing an imperfect rocket-provided separation state and evaluating whether the PGV can correct its trajectory toward the stationary target.

The baseline implementation uses a **continuously updated, turn-rate-limited pursuit guidance law**.

---

# Initial Separation Geometry

The rocket-provided separation state introduces an initial heading mismatch between the PGV trajectory and the target direction.

| Quantity | Value |
|---|---:|
| Separation X position | 223.00 m |
| Separation Y position | 0.10 m |
| Separation altitude | 1939.57 m |
| Initial PGV heading | 0.10° |
| Initial bearing to target | 38.46° |
| Initial heading error | 38.36° |

The PGV must therefore perform an initial heading correction toward the target direction.

---

# Guidance Model

The PGV state is represented using horizontal position, altitude and heading.

The kinematic motion model is:

```text
dx/dt = Vh cos(ψ)

dy/dt = Vh sin(ψ)

dz/dt = −Vd
```

The instantaneous target bearing is calculated as:

```text
bearing = atan2(yT − y, xT − x)
```

The heading error is:

```text
heading_error = wrap(bearing − ψ)
```

The commanded heading rate is:

```text
ψ̇cmd = clamp(Kp × heading_error, −ψ̇max, +ψ̇max)
```

with:

```text
Kp = 1.00

ψ̇max = 6.00 °/s
```

The heading command is continuously updated as the PGV moves.

---

# Initial Heading Correction

The initial heading correction produces the following results.

| Quantity | Value |
|---|---:|
| Correction completion time | 6.40 s |
| X position at heading alignment | 259.77 m |
| Y position at heading alignment | 12.98 m |
| Altitude at heading alignment | 1927.29 m |
| Horizontal distance travelled during correction | 38.94 m |
| Altitude lost during correction | 12.29 m |
| Final heading | 38.46° |
| Desired heading | 38.46° |
| Final heading error | 0.00° |

The initial heading correction is completed shortly after separation while consuming only a small fraction of the available altitude.

---

# V1 Mission Geometry — 3D

![V1 Mission Geometry 3D](v1_mission_geometry_3d.png)

---

# V1 Horizontal Mission Geometry - Heading Correction Phase

![V1 Horizontal Mission Geometry](v1_horizontal_mission_geometry.png)

---

# Altitude vs Horizontal Distance

![V1 Altitude vs Horizontal Distance](v1_altitude_vs_horizontal_distance.png)

---

# Heading vs Time

![V1 Heading vs Time](v1_heading_vs_time.png)

The heading response demonstrates the initial turn-rate-limited correction from the rocket-provided separation heading toward the target direction.

---

# Heading Correction Geometry

Because the complete mission trajectory spans several kilometres, the initial heading correction is difficult to observe at full mission scale.

The following plots isolate the initial turning region.

---

## Horizontal Mission Geometry — Magnified

![V1 Heading Correction Magnified](v1_heading_correction_magnified.png)

These plots provide a closer view of the initial turning manoeuvre and the point at which the PGV heading aligns with the required direction.

---

## Horizontal Mission Geometry — Heading Correction

![V1 Heading Correction](v1_heading_correction.png)

---

# Target-Vicinity Result

The baseline guidance simulation evaluates horizontal target acquisition.

| Quantity | Value |
|---|---:|
| Time at target vicinity | 776.90 s |
| Horizontal range error | 9.44 m |
| Altitude at target vicinity | 447.92 m |
| Vertical velocity | −1.92 m/s |

## Interpretation

The **9.44 m value represents horizontal target-vicinity error**.

It is **not a landing miss distance**, because the PGV remains at an altitude of **447.92 m** when this condition is reached.

This result demonstrates that the current baseline guidance model can achieve close horizontal target acquisition, while also highlighting the need for future terminal altitude and energy management.

---

# Ground-Termination Analysis

A separate propagation of the same baseline model can be evaluated until ground altitude:

```text
z = 0
```

Using the documented initial conditions and guidance logic, the resulting terminal positions are:

| Quantity | Guided | Uncorrected |
|---|---:|---:|
| Terminal X | **3977.89 m** | **6486.19 m** |
| Terminal Y | **2955.78 m** | **11.03 m** |
| Ground miss distance | **49.44 m** | **3887.81 m** |
| Flight time | **1010.20 s** | **1010.19 s** |
| Miss-distance reduction | **98.73%** | — |

The guided case substantially reduces terminal error relative to the uncorrected trajectory.

> **Metric distinction:** The **9.44 m** result represents horizontal target-vicinity error at **447.92 m altitude**, whereas **49.44 m** represents horizontal miss distance when the separate propagation reaches ground altitude.

---

# Guidance Development Summary

```text
V0 — Mission Feasibility
│
├── Determine whether available altitude provides sufficient range
│
└── Result: 5 km mission is theoretically reachable
        │
        ▼
V1 — Baseline Guidance
│
├── Introduce imperfect separation heading
│
├── Apply turn-rate-limited heading correction
│
└── Evaluate target acquisition
        │
        ▼
Target-Vicinity Result
│
├── Horizontal error: 9.44 m
│
└── Altitude remaining: 447.92 m
        │
        ▼
Ground-Termination Analysis
│
├── Guided miss: 49.44 m
│
└── Uncorrected miss: 3887.81 m
```

---

# Current Limitations

The current implementation is deliberately simplified and does not yet model:

- Wind and atmospheric disturbances
- Sensor noise
- Navigation uncertainty
- Actuator dynamics and lag
- Detailed parafoil aerodynamics
- Bank-induced sink-rate coupling
- Pendulum payload dynamics
- Terrain variation
- Closed-loop state estimation
- Hardware-specific PGV performance validation

The current PGV parameters should therefore be interpreted as **baseline simulation assumptions**.

---

# Future Development

Future work will focus on increasing robustness and physical fidelity.

## Sensitivity Analysis

Evaluate the effect of variations in:

- Separation altitude
- Separation position
- Initial heading
- PGV performance parameters

## Monte Carlo and CEP Analysis

Introduce uncertainty and evaluate miss-distance distributions across multiple simulated conditions.

## Wind and Disturbance Modelling

Evaluate guidance performance under atmospheric disturbances.

## Higher-Fidelity PGV Dynamics

Introduce:

- Bank dynamics
- Variable sink rate
- Aerodynamic coupling
- More realistic parafoil behaviour

## Navigation and Control

Future development may include:

- Sensor modelling
- State estimation
- Extended Kalman filtering
- Actuator dynamics

## MATLAB Cross-Verification

The Python implementation can later be independently reproduced and verified using MATLAB.

---

# Directory Structure

```text
02_pgv_guidance/
│
├── mission_feasibility.ipynb
├── baseline_guidance.ipynb
│
├── v0_mission_geometry_3d.png
├── v0_horizontal_mission_geometry.png
├── v0_altitude_vs_horizontal_distance.png
│
├── v1_mission_geometry_3d.png
├── v1_horizontal_mission_geometry.png
├── v1_altitude_vs_horizontal_distance.png
├── v1_heading_vs_time.png
├── v1_heading_correction.png
├── v1_heading_correction_magnified.png
│
└── README.md
```

---

# Status

**Current Stage: GNC Proof-of-Concept**

The current work establishes:

- A theoretically feasible 5 km mission under the assumed PGV glide model
- A rocket-provided separation altitude of **1939.57 m**
- Recovery from a significant initial heading mismatch
- Heading alignment within **6.40 s**
- Horizontal target-vicinity error of **9.44 m** at **447.92 m altitude**
- A separately evaluated guided ground miss distance of **49.44 m**
- An uncorrected ground miss distance of **3887.81 m**
- A **98.73% reduction** in terminal miss distance under the baseline model

The implementation is intentionally framed as a **simplified engineering proof-of-concept**, providing a baseline for future robustness analysis and higher-fidelity PGV guidance development.