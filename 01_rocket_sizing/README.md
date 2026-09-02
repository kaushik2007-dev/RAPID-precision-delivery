# Rocket Sizing

This directory contains the baseline launch vehicle design and OpenRocket simulation data used to establish the initial insertion conditions for the RAPID Payload Glide Vehicle (PGV).

The rocket sizing work is intended as a **demonstration-level baseline configuration**, designed to show that a launch vehicle can insert the PGV to approximately **2 km altitude** before separation and initiation of the glide phase.

## Objective

The objective of this stage is to establish a feasible rocket configuration capable of providing the PGV with a sufficiently high release state for the subsequent guidance and delivery simulation.

The primary output of this stage is the **PGV insertion/separation state**, particularly:

- Release altitude
- Position and trajectory state
- Velocity information
- Flight conditions near apogee

These outputs form the initial conditions for the PGV Guidance, Navigation and Control (GNC) simulations.

---

## Baseline Rocket Configuration

| Parameter | Value |
|---|---|
| Total rocket length | **1250 mm** |
| Outer diameter | **100 mm** |
| Nose cone | Ogive, 250 mm, Polystyrene |
| Body tube | 1000 mm, 100 mm OD, Cardboard |
| Fins | 3 × trapezoidal, Balsa, 3 mm thickness |
| Fin root chord | 180 mm |
| Fin tip chord | 80 mm |
| Fin span | 120 mm |
| Fin sweep | 120 mm |
| Payload module | 0.5 kg, Ø80 mm × 300 mm |
| Motor mount | 54 mm ID, 300 mm long |
| Selected motor | **AeroTech J99N** |
| Motor class | J-class, 54 mm |
| Delay charge | None |

### Motor Selection

Multiple 54 mm I–J class motors were evaluated in OpenRocket during the sizing process:

- AeroTech I117FJ
- AeroTech I599N
- AeroTech J99N
- Cesaroni 502I120-15A
- Cesaroni 518I165-17A
- Cesaroni 491I218-14A
- Cesaroni 836J210-16A

The baseline configuration currently uses the **AeroTech J99N** motor with no delay charge.

> **Note:** Stability margin, dry mass, and launch mass should be read from the final OpenRocket configuration and added once verified.

---

## Simulated Trajectory

![RAPID Baseline Launch Vehicle](rocket_configuration.png)

The baseline configuration was simulated using the **"PGV Insertion States"** run in OpenRocket.

| Parameter | Result |
|---|---:|
| Apogee altitude | **1939.571 m** |
| Time to apogee | **18.492 s** |
| Maximum velocity | **181.739 m/s** |
| Maximum acceleration | **74.405 m/s²** |
| Maximum Mach number | **0.539** |
| Launch rod exit velocity | **12.093 m/s** |
| Total ballistic flight time | **49.757 s** |
| Ground impact velocity | **72.346 m/s** |

The simulation demonstrates that the baseline configuration can achieve an insertion altitude of approximately **1.94 km**, which is used as the nominal PGV release altitude in the current guidance model.

---

## Files

| File | Description |
|---|---|
| `rapid_pgv_lv.ork` | OpenRocket baseline launch vehicle design and simulation configuration |
| `release_states_data.csv` | Raw flight data exported from OpenRocket |
| `release_states_data_clean.csv` | Processed flight data prepared for downstream analysis and PGV guidance work |

---

## Role in RAPID

The rocket sizing stage provides the transition between the launch vehicle and the PGV guidance subsystem:

```text
Rocket Launch
     ↓
Powered Ascent
     ↓
Apogee / PGV Release
     ↓
PGV Initial Conditions
     ↓
PGV Guidance Simulation