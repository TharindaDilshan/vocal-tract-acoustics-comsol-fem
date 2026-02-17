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

