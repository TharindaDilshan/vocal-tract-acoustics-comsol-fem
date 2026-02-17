# vocal-tract-acoustics-comsol-fem

Finite Element Method (FEM) workflow for **3D vocal-tract acoustics** in **COMSOL Multiphysics** using *Pressure Acoustics, Frequency Domain* (Helmholtz formulation), including an exterior radiation domain and utilities for extracting **SPL-based transfer functions** and **formants**.  

---

## Overview

This repository shares a **reusable COMSOL workflow** to simulate vocal-tract acoustics from a prepared 3D geometry (airway + enclosed head) via **linear frequency-domain acoustics**:

- Governing equation: Helmholtz (harmonic pressure wave propagation)
- Excitation: prescribed **uniform glottal normal particle velocity**
- Walls: **sound-hard** (rigid) or **thermoviscous/impedance** approximation (optional)
- Radiation: spherical exterior air region + **PML** to suppress reflections
- Readout: **SPL transfer response** at a receiver point in free field (e.g., 15 cm in front of lips)
- Formants: identified as prominent **local maxima** in SPL response over a frequency band

---

## What’s included

- COMSOL `.mph` model(s) implementing the workflow (import geometry → add sphere → PML → BCs → sweep → export SPL)
- Sample geometry files (STL) that run "out of the box"
---

## Requirements

- **COMSOL Multiphysics** with **Acoustics Module**
  - Physics used: *Pressure Acoustics, Frequency Domain*
  - Optional: impedance/thermoviscous boundary approximations depending on your setup
- A COMSOL license that can open `.mph` files

> Note: PMLs are supported for frequency-domain pressure acoustics in COMSOL.  
> If you don’t want PMLs, you can adapt the workflow to use alternative radiation boundaries, but the default here uses PML.  

---

## Quickstart

1. Clone the repository
2. Open the COMSOL model:
    - models/vocal_tract_acoustics_pml.mph
3. (If needed) swap in your geometry:
    - Replace the imported STL in the Geometry sequence with your own head-enclosed vocal tract air volume
4. Run the frequency sweep study

---

## Geometry prerequisites (upstream, not included)

This repo assumes you already have a simulation-ready geometry representing the air volume (fluid domain) of:
1. Vocal tract airway (from MRI segmentation)
2. A closed facial/head enclosure surrounding the mouth opening (to stabilize radiation sensitivity)
3. A clean, watertight surface suitable for tetrahedral meshing

Typical upstream workflow (example only):
- Segment airway from volumetric MRI (e.g., ITK-SNAP)
- Repair to watertight surface and clean artifacts
- Augment with boundary extensions / head enclosure (e.g., Geomagic)
- Final mesh/quality checks (e.g., "mesh doctor" style repairs and ACVD remeshing)

Upstream steps can vary a lot by lab/toolchain — this repo focuses on the COMSOL acoustic simulation once a geometry is ready.

---

## COMSOL workflow

1) Import geometry
    - Global Definitions → Mesh Parts -> Mesh Part 1 → Import
        - Import your STL of the head-enclosed vocal tract air volume
    - Ensure units are correct (meters recommended).

2) Add exterior air domain (sphere)
    - Add a sphere that fully encloses the head/enclosure
        - Typical radius: R_ext = 0.4 [m] (adjust as needed)
    - Perform a Difference to create a combined air domain (airway + exterior)

3) Create the air inlet (glottal) boundary
    - Create a boundary at the glottal end of the airway to assign physics
    - This can be done by creating a small planar surface or using an existing boundary if it’s clean

4) Assign material
    - Material: Air (density, speed of sound as per your conditions)

5) Add physics: Pressure Acoustics, Frequency Domain
    - Pressure Acoustics, Frequency Domain on the air domains.

6) Boundary conditions
    - Walls (choose one):
        - Option A (default): Sound hard / rigid wall
            - Neumann-type: normal velocity = 0 on vocal-tract + head enclosure surfaces
        - Option B (optional): Impedance / thermoviscous approximation
            - Use an impedance boundary if you want to approximate wall losses
    - Glottal excitation:
        - Apply excitation at the glottal termination boundary `Γ_g`
        - Prescribed uniform normal particle velocity:
            - `v_n = 1 [m/s]` (parameterizable)
        - Equivalent relation:
            - `∂p/∂n = -j ω ρ v_n` on `Γ_g`
    - Radiation:
        - Apply a PML (acpr -> Perfectly Matched Boundary) on the outer boundary of the exterior sphere to absorb outgoing waves and minimize reflections

7) Meshing
    - Use tetrahedral mesh.
    - The given examples use a physics-controlled mesh with `normal` element size, but you may need to adjust based on geometry complexity and frequency range.

8) Study: Frequency sweep
    - Frequency range (example):
        - 50–3000 Hz with 10 Hz resolution
    - Run Frequency Domain study.

---

## Default simulation settings

- Physics: Pressure Acoustics, Frequency Domain
- Frequency sweep: 50–3000 Hz, step 10 Hz
- Walls: sound-hard or optional impedance approximation
- Glottis: uniform normal particle velocity `v_n = 1 m/s`
- Exterior domain: sphere radius ~0.4 m
- Radiation: Perfectly Matched Boundary on outer sphere boundary
- Receiver: point located 15 cm anterior to lips (Results -> Datasets -> Cut Point 3D)

---

## Output and postprocessing

- Export a 1D dataset (frequency → SPL at receiver):
    - Results → Derived Values → Point Evaluation at receiver point
    - Expression example (COMSOL-style):
        - `20*log10(abs(acpr.p_t)/20e-6)`
- Identify formants (F1, F2, F3) as local maxima in the SPL response curve
    - Results -> 1D Plot Group -> Table Graph (select the table from the point evaluation dataset)

---

## How to cite