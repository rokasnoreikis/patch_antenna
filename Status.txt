
# Project: 4.95 GHz Microstrip Patch Antenna

## 1. Project objective

I am developing a **4.95 GHz inset-fed microstrip patch antenna**, initially as a single element, with plans to develop **4×4 and 8×8 phased arrays**.

Long-term applications:
- Electronic beam steering in azimuth and elevation.
- Drone detection radar.
- Beamformed RF communications.
- Potential drone-mounted antenna arrays.

The immediate objective is to **manufacture and test five single-element antenna PCBs**.

The antenna has already been designed and optimized using openEMS. We are now transitioning to PCB design in KiCad 10.

**Do not restart antenna optimization unless a manufacturing issue requires it.**

---

## 2. Development environment

Operating system: Ubuntu

Simulation software:
- openEMS v0.37
- GNU Octave 11.3
- AppCSXCAD

PCB design software:
- KiCad 10

Measurement equipment:
- LibreVNA for S11 measurements.
- USRP B210 clone for future RF experiments.

Antenna source file:

`~/DARBINIS/openEMS-Project/Patch_4p9_5GHz/patch_4p95GHz.m`

Preferred editor: VS Code (`code`), not nano.

---

## 3. Final optimized antenna geometry — V1

| Parameter | Value |
|---|---:|
| Center frequency | 4.950 GHz |
| Operating frequency | 4.900–5.000 GHz |
| Patch width | 19.85 mm |
| Patch length | **15.29 mm** |
| Microstrip feed width | 3.3 mm |
| Inset depth | 6.0 mm |
| Inset gap | 0.5 mm |
| Feed stub length | 6.0 mm |
| Input impedance | 50 Ω |

Corresponding openEMS parameters:

```matlab
patch.W  = 19.85;
patch.L  = 15.29;
patch.Ws = 3.3;
patch.Gs = 0.5;
patch.l  = 6;
patch.y0 = 6.0;
```

**Important:** The antenna uses an inset-fed microstrip line, not a coaxial probe feed.

---

## 4. Substrate and PCB

Selected material: **Rogers RO4350B**

| Parameter | Value |
|---|---|
| Material | RO4350B |
| Dielectric constant used in simulation | 3.66 |
| Loss tangent | 0.0037 |
| Dielectric thickness | 1.524 mm |
| PCB layers | 2 |
| Top layer | Patch antenna and microstrip |
| Bottom layer | Continuous ground plane |
| Intended copper thickness | Approximately 35 µm |
| Manufacturer | JLCPCB (candidate) |

The simulation uses:

```matlab
substrate.name = 'RO4350B';
substrate.epsR = 3.66;
substrate.kappa = 0.0037 * 2*pi*f0 * EPS0*substrate.epsR;
substrate.thickness = 1.524;
substrate.cells = 16;
```

Rogers specifies a process dielectric constant of approximately 3.48 and a design dielectric constant of 3.66.

The simulation uses the design value.

### Actual simulated PCB dimensions

The source code calculates:

```matlab
substrate.width = patch.W + max(xpos) - min(xpos) + 4*patch.l;
substrate.length = 3*patch.l + patch.L;
```

For a single antenna with `xpos = [0]`:

**PCB width = 43.85 mm**

**PCB length = 33.29 mm**

We initially discussed 60 × 60 mm, but subsequently decided to preserve the simulated **43.85 × 33.29 mm** dimensions for V1.

Do not silently change the ground-plane dimensions.

---

## 5. Final openEMS simulation results

Final optimized patch length: **15.29 mm**

| Parameter | Result |
|---|---:|
| Resonant frequency | **4.951 GHz** |
| Minimum S11 | **−30.17 dB** |
| S11 at 4.900 GHz | −11.34 dB |
| S11 at 4.950 GHz | −30.01 dB |
| S11 at 5.000 GHz | −11.58 dB |
| Worst S11 across operating band | −11.34 dB |
| Input impedance at 4.950 GHz | **50.31 − j3.15 Ω** |

**Result: PASS**

The antenna meets our simulated requirement:

\[
S_{11}\leq-10\text{ dB}
\]

throughout 4.900–5.000 GHz.

The matching is nearly symmetrical at both band edges.

However, the margin is only approximately 1.34 dB at the worst frequency. Manufacturing tolerances could affect performance.

### Previous geometry comparison

| Parameter | L = 15.26 mm | L = 15.29 mm |
|---|---:|---:|
| Resonant frequency | 4.959 GHz | 4.951 GHz |
| Minimum S11 | −28.55 dB | −30.17 dB |
| Worst band S11 | −10.12 dB | −11.34 dB |
| Zin at 4.95 GHz | 46.52 − j4.92 Ω | 50.31 − j3.15 Ω |

The 15.29 mm geometry is our selected V1.

---

## 6. Simulation accuracy and limitations

FDTD configuration:

```matlab
f0 = 4.95e9;
fc = 0.5e9;

FDTD = InitFDTD('EndCriteria', 1e-4);
FDTD = SetGaussExcite(FDTD, f0, fc);
```

Simulation parameters:

| Parameter | Value |
|---|---:|
| Mesh | 131 × 98 × 84 |
| Total FDTD cells | 1,078,392 |
| Substrate Z cells | 16 |
| Local inset-feed Y mesh | 17 points |
| Stopping criterion | −40 dB |
| Actual final energy level | −42.62 dB |
| Simulation iterations | 48,300 |
| Simulation time | Approximately 11.5 minutes |

We previously compared 9-, 13-, and 17-point Y meshes. Results were approaching convergence, although strict numerical convergence was not conclusively demonstrated.

### Important modeling limitation

The patch and ground plane are modeled as **zero-thickness perfect electric conductors (PEC)**.

Example:

```matlab
CSX = AddMetal(CSX, 'patch');
CSX = AddMetal(CSX, 'gnd');
```

The real PCB will have finite-thickness copper with finite conductivity.

The simulation also uses an **ideal lumped feed port**. It does not include the physical SMA connector or its launch transition.

Therefore, the simulated S11 is not guaranteed to match the manufactured antenna exactly.

---

## 7. Selected SMA connector

We decided to use an **edge-mounted SMA connector**, not a vertical connector.

Current candidate:

**Cinch Connectivity Solutions / Johnson 142-0701-801**

Mouser part number:

**530-142-0701-801**

Connector characteristics:

| Parameter | Value |
|---|---|
| Type | SMA female, edge-launch |
| Impedance | 50 Ω |
| Frequency range | DC–18 GHz |
| Nominal PCB thickness | 1.57 mm |
| Dimension A | 1.73 mm |
| Dimension B | 1.85 mm |
| Center contact diameter | 0.76 mm |
| Connector body width | 9.52 mm |

The manufacturer drawing was examined.

Our substrate thickness is 1.524 mm, which is close to the connector's specified nominal PCB thickness of 1.57 mm.

We consider it a promising mechanical fit, but the **finished board thickness and recommended RF launch geometry still need verification**.

The connector's 0.76 mm center contact must transition to our 3.3 mm microstrip feed.

Do not assume that this transition is automatically impedance matched.

---

## 8. Current manufacturing plan

We want to manufacture approximately **five identical single-element antennas**.

Preferred manufacturer: JLCPCB.

Planned workflow:

1. Verify the exact JLCPCB RO4350B stackup and available copper thickness.
2. Confirm SMA connector compatibility with finished PCB thickness.
3. Obtain or construct the connector's recommended KiCad footprint.
4. Create the antenna PCB in KiCad 10.
5. Preserve the simulated patch and feed geometry.
6. Design the SMA-to-microstrip transition.
7. Verify PCB dimensions, copper geometry, ground plane, and manufacturing rules.
8. Export Gerber and drill files.
9. Order five prototype PCBs.
10. Assemble and measure S11 using LibreVNA.

**Current stage: Selecting and implementing the SMA connector footprint in KiCad.**

The connector candidate has been identified, but the PCB layout has not yet been created.

---

## 9. Future phased-array development

At 4.95 GHz:

\[
\lambda\approx60.6\text{ mm}
\]

A reasonable initial element spacing is:

\[
d=\lambda/2\approx30.3\text{ mm}
\]

Approximate array dimensions:

| Array | Estimated size |
|---|---|
| 4×4 | 120 × 120 mm |
| 8×8 | 250 × 250 mm |

An 8×8 array contains 64 radiating elements.

Estimated broadside HPBW for a uniformly excited 8-element aperture is approximately 13° per principal plane, subject to element pattern, coupling, and feed design.

**Important:** The single-element PCB width is 43.85 mm, larger than the intended 30.3 mm array spacing.

Therefore, we cannot simply arrange 64 identical single-element PCBs into an 8×8 array with half-wavelength spacing.

The array will require a redesigned shared substrate and ground plane, including mutual-coupling analysis.

Array design is a later project stage.

---

## 10. What we are NOT doing yet

Do not start these tasks until the first PCB prototypes are ready or I explicitly request them:

- 4×4 array design.
- 8×8 array design.
- Beamforming network design.
- ADAR1000 integration.
- Radar signal processing.
- Radiation-pattern optimization.
- Further patch-length optimization.

We will evaluate radiation patterns, gain, efficiency, and array characteristics later.

---

## 11. Immediate next task for ChatGPT

**Help me create the first manufacturable 4.95 GHz patch antenna PCB in KiCad 10.**

Start by verifying the Cinch 142-0701-801 SMA connector's recommended PCB footprint and RF launch geometry.

Then guide me through creating the PCB layout.

Work **one step at a time**.

Avoid unnecessary explanations and do not suggest additional simulations unless they are technically justified.

Do not invent component dimensions, PCB manufacturing capabilities, or manufacturer specifications. Verify them from manufacturer documentation when necessary.