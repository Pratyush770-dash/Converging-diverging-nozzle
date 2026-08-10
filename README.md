# Converging-Diverging Nozzle — Supersonic Flow CFD

A compressible flow simulation built in ANSYS Fluent 2026 R1, targeting supersonic acceleration through a CD nozzle. Peak exit Mach was 2.31. The centreline Mach plot shows the sonic crossing at the throat, which is the main validation check against 1D isentropic theory.

I built this specifically because I wanted to understand how the density-based solver setup differs from what I did in the pipe flow and lid-driven cavity projects — and because LPSC works on liquid propulsion nozzles, so having a nozzle CFD project felt directly relevant to that application.

## Table of Contents
- [What Was Done](#what-was-actually-done)
- [Setup / Software Used](#setup--software-used)
- [Reproducing the Results](#reproducing-the-results)
- [Results](#results)
- [Method, Briefly](#method-briefly)
- [Limitations](#limitations)
- [Repository Structure](#repository-structure)
- [Author](#author)

## What was actually done

- Drew the nozzle as a 2D axisymmetric profile in Onshape — cosine-curve converging wall from r = 67.87 mm at inlet down to r = 5 mm at throat, then a straight diverging wall back up to r = 55 mm, then a constant-radius exit section out to 600 mm total length
- Exported as STEP and imported into ANSYS Fluent via Workbench; had to fix a unit mismatch (Fluent was reading mm geometry as metres, so the domain was showing up as 0.07 m instead of 600 mm)
- Set up the density-based solver with ideal gas, energy equation on, k-ω SST turbulence — this is completely different to the pressure-based incompressible setup used in the pipe flow project and matters a lot for supersonic flow
- Used pressure-inlet and pressure-outlet boundary conditions rather than velocity-inlet, since total pressure and total temperature are the physically correct specifications for compressible inlet flow
- Got exit Mach 2.31, outlet static pressure 1.66×10⁴ Pa, outlet static temperature 379 K — all within about 1% of what isentropic relations predict for those conditions
- Generated Mach, pressure, temperature contours, pathlines, velocity vectors, and a centreline Mach XY plot; the XY plot confirms Mach = 1 at x = 72.45 mm (the throat) and the expected S-curve rising to Mach 2.2 at exit
- Had to place the centreline Line/Rake surface at Y = 0.001 m rather than Y = 0 — placing it exactly on the axis causes an axisymmetric singularity that makes the plot come out wrong

## Setup / Software Used

- Onshape (geometry — free, browser-based; STEP export)
- ANSYS Fluent 2026 R1, Student version (solver, meshing, post-processing)

## Reproducing the Results

1. Open ANSYS Workbench → Fluid Flow (Fluent)
2. Import the nozzle STEP file: right-click Geometry → Import Geometry → confirm units are **Millimeters** in DesignModeler before generating
3. Mesh the 2D axisymmetric domain with a structured quad mesh; apply wall inflation targeting y⁺ ≈ 1 at the throat
4. In Fluent: set solver to **Density-Based**, steady, **2D Axisymmetric**; enable the **Energy equation**; set fluid to **Ideal Gas** (air)
5. Turbulence: **k-ω SST**, default coefficients
6. Spatial discretisation: **Second-Order Upwind** on all equations; **Roe-FDS** flux type
7. Boundary conditions:
   - Inlet → pressure-inlet: total pressure 2.12×10⁵ Pa, total temperature 797 K
   - Outlet → pressure-outlet: gauge pressure 1.66×10⁴ Pa
   - Wall → no-slip, adiabatic
   - Axis → axisymmetric
8. Residual target 1×10⁻⁶ on all equations; run until convergence
9. For the centreline XY plot: New Surface → Line/Rake, set X0=0, Y0=0.001, Z0=0, X1=0.6, Y1=0.001, Z1=0 (in metres); select this surface in the XY Plot dialog with Mach Number on the Y axis and Position on the X axis

## Results

**Nozzle geometry**

| Parameter | Value |
|-----------|-------|
| Total length | 600 mm |
| Converging length | 72.45 mm |
| Diverging length | 137.55 mm |
| Inlet radius | 67.87 mm |
| Throat radius | 5 mm |
| Exit radius | 55 mm |
| Area ratio (exit/throat) | 121 |

**Key flow values**

| Location | Mach | Static Pressure (Pa) | Static Temperature (K) |
|----------|------|----------------------|------------------------|
| Inlet (x = 0) | 0.41 | 2.12 × 10⁵ | 797 |
| Throat (x = 72.45 mm) | ~1.0 | — | — |
| Exit (x = 600 mm) | 2.31 | 1.66 × 10⁴ | 379 |

**Isentropic check at exit (M = 2.31, γ = 1.4):**
- p/p₀ theoretical = 0.079 → p_exit ≈ 1.67×10⁴ Pa (CFD: 1.66×10⁴ Pa, error < 1%)
- T/T₀ theoretical = 0.475 → T_exit ≈ 379 K (CFD: 379 K, matches)

Full contour images, pathlines, velocity vectors, and XY plot are in `results/`.

## Method, briefly

- **Geometry:** 2D axisymmetric profile in Onshape. Converging wall uses cosine interpolation from inlet to throat (smoother than linear, avoids artificial separation at the throat in the solver). Diverging section is straight-walled — not MOC-optimised, which is noted in Limitations.
- **Solver:** Density-based. Not a choice — the pressure-based solver cannot handle the strong density-velocity coupling in supersonic flow. Ideal gas, energy equation on. These two settings are also non-negotiable for compressible simulation.
- **Turbulence:** k-ω SST. Better near-wall behaviour than k-ε on a wall-resolved mesh (y⁺ ≈ 1) without needing wall functions. Same reasoning as the pipe flow project but matters even more here because the boundary layer at the throat is thin and under a strong favourable pressure gradient.
- **Boundary conditions:** pressure-inlet/outlet rather than velocity-inlet. For compressible flow, total pressure and temperature are the physically meaningful specification at the inlet. Fixing velocity instead leads to wrong density at the inlet face.
- **Post-processing:** Mach contour, static pressure contour, static temperature contour, pathlines colored by Mach, velocity vectors colored by Mach, and centreline XY plot. The XY plot is the validation result — the sonic crossing at the throat is the key check.

## Limitations

The diverging section uses a straight wall. A real nozzle designed for maximum thrust would use a Method of Characteristics contour to produce uniform, axially-parallel exit flow. The straight wall causes some radial non-uniformity in the exit Mach field — visible in the contour as a slight Mach difference between the centreline and near-wall regions in the diverging section. For a portfolio/validation study this is fine; for an actual nozzle design it would need fixing.

No grid convergence study (Richardson extrapolation / GCI) was done. Mesh quality was verified through Fluent's quality metrics but not through a formal three-level refinement. Planned as a follow-up.

The external plume is not modelled. The domain ends at the nozzle exit, so any shock structure that would appear downstream under over-expanded conditions is not captured.

## Repository Structure

```
├── README.md
├── CD_Nozzle_CFD_Report
├── geometry/   
│   └── geometry                            → ANSYS Designermodeler    
├── mesh/
│   └── mesh                                → ANSYS Meshing export
├── results/
│   ├── contours/
│   │   ├── contours_of_mach_number         → Mach contour, max 2.31
│   │   ├── contours_of_static_pressure     → Pressure drop inlet to exit
│   │   └── contours_of_static_temperature  → Temperature drop, 797 K → 379 K
│   ├── pathlines/
│   │   └── pathlines_of_mach_number
|   ├── residuals/
|   |   └── scaled_residuals
│   ├── vectors/
│   │   └── velocity_vectors_of_mach_number
│   └── xy_plot/
│       └── xy_plot_centerline               → Validation: sonic at throat, Mach 2.2 at exit
└── case_files/
    └── cd_nozzle.cas.h5                     → Fluent case + data
```

## Author

**Pratyush Dash**


B.Tech Chemical Engineering, KIIT University, Bhubaneswar
  
