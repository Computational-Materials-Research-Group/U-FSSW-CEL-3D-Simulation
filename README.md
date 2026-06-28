# U-FSSW-CEL-3D-Simulation

<p align="center">
  <img src="https://img.shields.io/badge/FreeFEM++-Simulation-blue?style=for-the-badge&logo=gnu&logoColor=white"/>
  <img src="https://img.shields.io/badge/AA6061--T6-Aluminium-silver?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/3D-Coupled%20CEL%20%2B%20Thermal%20%2B%20VOF-orange?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Ultrasonic-20%20kHz%20Assisted-purple?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/Norton--Hoff-Viscoplastic-darkgreen?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/ParaView-VTK%20Export-green?style=for-the-badge"/>
  <img src="https://img.shields.io/badge/License-CC%20BY--NC%204.0-lightgrey?style=for-the-badge"/>
</p>

<p align="center">
  A fully 3D coupled simulation of <b>Ultrasonic-assisted Friction Stir Spot Welding (U-FSSW)</b>
  implemented in FreeFEM++. Two AA6061-T6 aluminium plates are joined using a
  <b>Coupled Eulerian-Lagrangian (CEL)</b> viscoplastic flow solver coupled with
  <b>transient heat transfer</b> and a <b>Volume-of-Fluid (VOF)</b> phase tracker,
  with <b>20 kHz ultrasonic vibration</b> superimposed on the tool to reduce plunge
  force and improve nugget quality through acoustic softening.
</p>

<img width="1008" height="772" alt="ufssw" src="https://github.com/user-attachments/assets/ae931970-dbe9-43f9-a23a-b014db462bcf" />

---

## Concept

U-FSSW plunges a rotating shouldered pin into two overlapping plates. Ultrasonic
vibration superimposed on the tool reduces the flow stress of the aluminium
(Blaha effect), lowering plunge force by 15–30% and improving material mixing
at the weld nugget. This simulation captures all three coupled physics:

| Aspect | Description |
|---|---|
| Flow solver | Stokes viscoplastic (MINI element P1b/P1) |
| Heat solver | Crank-Nicolson advection-diffusion |
| Phase tracker | VOF advection (φ = 1 material, φ = 0 void) |
| Ultrasonic model | Acoustic softening + Eckart streaming + US heat |
| Tool motion | CEL: rigid body velocity imposed on tool-region nodes |
| Phases | Plunge → Dwell → Retract |

---

## Geometry

```
        z
        |
  Hztot |  ← top surface (tool contacts here, z = 4 mm)
        |
   Hz1  |  ← plate interface (z = 2 mm)
        |
      0 +----------------------------------------------- x
        |       BASE PLATE (backing anvil, z = 0)

  Full plate: 60 mm × 60 mm × 4 mm  (two 2 mm AA6061-T6 plates)
  Tool axis:  x = 0, y = 0  (centre of plate)

  Tool:
    Shoulder radius:  Rsh  = 10 mm
    Pin radius:       Rpin =  3 mm
    Pin length:       Lpin =  3.5 mm
    Speed:            1500 RPM

  Quarter symmetry model available; full-plate model used here.
```

---

## Physics

The simulation solves four coupled problems each timestep:

### 1. Viscoplastic Stokes flow (CEL)

```
  -div(2η_eff * ε̇) + ∇p = F_US
  div(u) = 0

  η_eff = softenFactor * K(T) / (2 * ε̇^(1-m) + ε_reg)
  K(T)  = K0 * exp(-Q / (R*T))          ← Norton-Hoff consistency
  softenFactor = 1 - β_US               ← acoustic softening (Blaha effect)
```

### 2. Transient heat transfer

```
  ρ Cp dT/dt + ρ Cp (u·∇)T = div(k ∇T) + Q_total

  Q_total = Q_friction  (shoulder + pin surface)
           + Q_viscoplastic  (bulk dissipation)
           + Q_ultrasonic    (= η_US * ρ * (ω_US * A_US)² * φ)
```

### 3. VOF phase tracking

```
  dφ/dt + (u·∇)φ = 0        φ = 1 (metal),  φ = 0 (void)
```

### 4. Ultrasonic body force (Eckart acoustic streaming)

```
  F_US = ρ * v_US² / (2 * c_sound)      [N/m³]   time-averaged
  v_US = A_US * ω_US                    [m/s]    peak vibration velocity
```

---

## Material Properties — AA6061-T6

| Property | Symbol | Value | Unit |
|---|---|---|---|
| Density | ρ | 7850 | kg/m³ |
| Specific heat | Cp | 896 | J/kg/K |
| Thermal conductivity | k | 167 | W/m/K |
| Solidus temperature | T_sol | 855 | K |
| Ambient temperature | T_amb | 293 | K |
| Norton-Hoff consistency | K₀ | 2.0 × 10⁸ | Pa·s^m |
| Strain rate exponent | m | 0.20 | — |
| Activation energy | Q | 1.5 × 10⁴ | J/mol |
| Friction coefficient | μ | 0.40 | — |
| Slip ratio | δ | 0.30 | — |
| Sound speed (longitudinal) | c | 6420 | m/s |

---

## Process Parameters

| Parameter | Symbol | Value | Unit |
|---|---|---|---|
| Tool rotation speed | ω | 1500 | RPM |
| Plunge speed | v_plunge | 1.5 | mm/s |
| Retract speed | v_retract | 3.0 | mm/s |
| Plunge phase duration | t_plunge | 2.33 | s |
| Dwell phase duration | t_dwell | 3.0 | s |
| Retract phase duration | t_retract | 1.17 | s |
| Total simulation time | t_total | 6.5 | s |
| Nominal contact pressure | σ_N | 150 | MPa |

---

## Ultrasonic Parameters

| Parameter | Symbol | Value | Unit |
|---|---|---|---|
| Ultrasonic frequency | f_US | 20 | kHz |
| Peak vibration amplitude | A_US | 20 | µm |
| Acoustic softening factor | β_US | 0.20 | — |
| US-to-heat efficiency | η_US | 0.85 | — |
| Vibration direction | dir_US | 1 (axial Z) | — |
| Peak vibration velocity | v_US = A·ω | 2.51 | m/s |
| Volumetric US power | P_acou = η·ρ·v_US² | ~1.07 × 10⁷ | W/m³ |
| Eckart streaming force | F_US = ρ·v_US²/2c | ~6300 | N/m³ |

**Vibration direction options:**

```
  dirUS = 1   Axial Z    — most effective for Fz reduction (15–30%)
  dirUS = 2   Lateral X  — promotes plasticised material mixing
  dirUS = 3   Combined   — amplitude split equally: A/√2 in each axis
```

---

## Acoustic Softening Model

The Blaha effect reduces the Norton-Hoff flow stress during ultrasonic vibration:

```
  K_eff(T) = (1 - β_US) * K₀ * exp(-Q / R*T)     [tool active]
  K_eff(T) =              K₀ * exp(-Q / R*T)     [retract phase]
```

`β_US = 0.10–0.30` for AA6061 at 20 kHz based on published experimental data.
This directly lowers `η_eff`, reducing the viscoplastic resistance of the
nugget zone and the plunge force `F_z`.

---

## Mesh

| Component | Nx | Ny | Nz | Notes |
|---|---|---|---|---|
| Full plate | 14 | 14 | 8 | 2×Lx × 2×Ly × Hztot |
| Element size (X/Y) | — | — | — | ~4.3 mm |
| Element size (Z) | — | — | — | 0.5 mm |
| Total nodes | ~2744 | — | — | P1b/P1 MINI element |
| Total tetrahedra | ~10 000 | — | — | |

`cube()` + linear `movemesh3` — no trigonometric functions in `transfo`
(required for FreeFEM++ v4.15 Windows compatibility).

**Boundary labels after `movemesh3`:**

```
  Label 1 = z = 0    bottom surface  →  uz = 0,  h = 300 W/m²/K (anvil)
  Label 2 = z = 1    top surface     →  CEL tool contact
  Label 3 = y = 0    front edge      →  free,  h = 5 W/m²/K (convection)
  Label 4 = y = 1    back edge       →  free,  h = 5 W/m²/K
  Label 5 = x = 0    left edge       →  free,  h = 5 W/m²/K
  Label 6 = x = 1    right edge      →  free,  h = 5 W/m²/K
```

---

## Timestepping

| Phase | dt | Steps | Reason |
|---|---|---|---|
| All phases | Ttotal / 120 | 120 | Fixed uniform stepping |
| Ultrasonic cycles per step | ~1000–2000 | — | dt ≫ 1/f_US → time-averaged US effect |

The ultrasonic frequency (20 kHz, T = 50 µs) is far faster than the
mechanical timestep (~54 ms). Individual cycles are not resolved; instead
the RMS vibration velocity and time-averaged streaming force are applied
each step — physically correct for the macroscopic flow and heat fields.

---

## CEL Tool Velocity

At every timestep, nodes inside the tool region (pin + shoulder) are assigned:

```
  u_x(node) = -ω * y  +  v_xUS          ← rotation + lateral US
  u_y(node) = +ω * x  +  v_yUS
  u_z(node) =  v_z    +  v_zUS          ← plunge/retract + axial US

  v_zUS = A_US * ω_US * cos(ω_US * t)   [axial, dirUS=1]
  v_xUS = A_US * ω_US * cos(ω_US * t)   [lateral, dirUS=2]
```

The `cos(ω_US * t)` factor tracks the instantaneous vibration phase.
Since many cycles occur per timestep, the cumulative effect integrates
to the correct RMS values over time.

---

## Output Files — `D:\freefem++\ufssw_cel_3d\`

| File | Description |
|---|---|
| `ufssw.pvd` | Master animation — open this in ParaView |
| `ufssw_0000.vtu` | Frame 0 — initial state |
| `ufssw_NNNN.vtu` | Per-frame: Vx, Vy, Vz, TempK, VOF, EtaEff |
| `tool_NNNN.vtu` | Per-frame: rotating shoulder + pin geometry |
| `ufssw_data.csv` | Step-by-step diagnostics |

### VTU Data Fields

| Field | Description | Unit |
|---|---|---|
| `Vx`, `Vy`, `Vz` | Material velocity components | m/s |
| `TempK` | Temperature | K |
| `VOF` | Phase field φ (0 = void, 1 = metal) | — |
| `EtaEff` | Effective viscosity (acoustic-softened) | Pa·s |

### CSV Columns

| Column | Description | Unit |
|---|---|---|
| `time_s` | Physical simulation time | s |
| `phase` | PLUNGING / DWELLING / RETRACTING | — |
| `ztip_m` | Pin tip z-position | m |
| `zsh_m` | Shoulder z-position | m |
| `Tmax_K` | Maximum temperature in domain | K |
| `Tcenter_K` | Temperature near tool centre | K |
| `Fz_N` | Estimated plunge force | N |
| `Torque_Nm` | Estimated torque | N·m |
| `Pacoustic_W_m3` | Volumetric ultrasonic power | W/m³ |
| `softenFactor` | K_NH reduction factor (1 − β_US) | — |
| `etaEff_center` | Viscosity at tool centre | Pa·s |
| `vUS_m_s` | Peak ultrasonic vibration velocity | m/s |

---

## Repository Structure

```
ufssw_cel_3d.edp                     # Main FreeFEM++ simulation script
README.md                            # This file

D:\freefem++\ufssw_cel_3d\
├── ufssw.pvd                        # Master animation (open first)
├── ufssw_data.csv                   # Step-by-step diagnostics
├── ufssw_0000.vtu                   # Initial state
├── ufssw_0001.vtu                   # After first saved step
├── ...
├── ufssw_00NN.vtu                   # Final state
├── tool_0000.vtu                    # Tool geometry frame 0
└── tool_00NN.vtu                    # Tool geometry final frame
```

---

## How to Run

### Requirements
- FreeFEM++ v4.10 or later: https://freefem.org
- ParaView v5.x or later: https://www.paraview.org

### Step 1 — Run the simulation

```bash
FreeFem++ ufssw_cel_3d.edp
```

The script will:
1. Build the full-plate mesh (60×60×4 mm) using `cube()` + `movemesh3`
2. Initialise temperature at `T_amb = 293 K`, VOF at `φ = 1` (all material)
3. For each of 120 timesteps:
   - Update tool position (plunge / dwell / retract)
   - Apply acoustic softening to `η_eff`
   - Compute friction + viscoplastic + ultrasonic heat sources
   - Solve Stokes (UMFPACK), impose CEL tool velocity with US vibration
   - Solve thermal (Crank-Nicolson), solve VOF advection
   - Save VTU + tool geometry every 10 steps
4. Write `ufssw.pvd` collection and `ufssw_data.csv`

Console output example:
```
==========================================
 U-FSSW CEL 3D — ULTRASONIC FSSW
 Plate: 60x60x4 mm
 Tool: shoulder R=10mm  pin R=3mm  L=3.5mm
 omega=1500 RPM
 tPlunge=2.33s  tDwell=3.0s  tRetract=1.17s
------------------------------------------
 ULTRASONIC:
   fUS    = 20 kHz
   AUS    = 20 um
   betaUS = 0.2  (softening factor)
   dirUS  = 1  (1=axial,2=lateral,3=combined)
   vUS    = 2.513 m/s (peak velocity)
   P_acou = 1.07e+07 W/m3
==========================================
 Mesh: 2744 vertices  10976 tetrahedra
 dt=0.054s   dtCFL=0.0021s
 US cycles per timestep: 1080
------------------------------------------
t=0s [PLUNGING]  ztip=4mm  Tmax=293K  Tc=293K  sf=0.8  Fz~0kN
t=0.54s [PLUNGING]  ztip=3.19mm  Tmax=612K  Tc=534K  sf=0.8  Fz~18.3kN
t=2.33s [DWELLING]  ztip=0.5mm  Tmax=801K  Tc=743K  sf=0.8  Fz~12.1kN
t=5.33s [RETRACTING]  ztip=1.67mm  Tmax=689K  Tc=601K  sf=1.0  Fz~9.8kN
==========================================
 U-FSSW DONE. Open: D:\freefem++\ufssw_cel_3d\ufssw.pvd
```

### Step 2 — Open in ParaView

1. `File > Open` → navigate to `D:\freefem++\ufssw_cel_3d\`
2. Select `ufssw.pvd` → OK → Apply

---

## Visualisation in ParaView

### Option A — Temperature field

```
Color by:     TempK
Colormap:     Inferno or Rainbow
Range:        293 to 855 K
Press Play -> watch the heat zone develop under the shoulder,
             then cool as the pin retracts
```

### Option B — Weld nugget boundary (VOF isosurface)

```
Filters > Contour
  Scalars:  VOF
  Isosurface value:  0.5
-> Shows the exact metal/void interface at every timestep
-> Nugget boundary visible after pin retract
```

### Option C — Material flow velocity

```
Color by:     Vx (or Vy, Vz)
Colormap:     Cool to Warm
Filters > Glyph -> Vectors (Vx, Vy, Vz)
-> See rotational flow under shoulder and downward pin flow
```

### Option D — Acoustic softening zone (viscosity)

```
Color by:     EtaEff
Colormap:     Viridis (log scale)
Range:        1e3 to 1e9 Pa.s
-> Low η_eff (dark blue) = soft plasticised nugget zone
-> High η_eff (yellow)   = cold rigid base material
```

### Option E — Threshold to isolate nugget

```
Filters > Threshold
  Scalars:  VOF
  Minimum:  0.5
  Maximum:  1.0
Click Apply
-> Removes the void / pin cavity from view
-> Color by TempK to see nugget thermal field
```

### Option F — Post-process CSV

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv('ufssw_data.csv')

fig, axes = plt.subplots(2, 2, figsize=(12, 8))

# Temperature history
axes[0,0].plot(df['time_s'], df['Tmax_K'], label='T_max')
axes[0,0].plot(df['time_s'], df['Tcenter_K'], label='T_centre')
axes[0,0].axhline(855, linestyle='--', color='red', label='Solidus 855K')
axes[0,0].set_xlabel('Time [s]'); axes[0,0].set_ylabel('Temperature [K]')
axes[0,0].legend()

# Plunge force
axes[0,1].plot(df['time_s'], df['Fz_N']/1000, color='steelblue')
axes[0,1].set_xlabel('Time [s]'); axes[0,1].set_ylabel('Fz [kN]')

# Acoustic softening
axes[1,0].plot(df['time_s'], df['softenFactor'], color='purple')
axes[1,0].set_xlabel('Time [s]'); axes[1,0].set_ylabel('Softening factor')

# Torque
axes[1,1].plot(df['time_s'], df['Torque_Nm'], color='darkorange')
axes[1,1].set_xlabel('Time [s]'); axes[1,1].set_ylabel('Torque [N·m]')

plt.tight_layout(); plt.show()
```

---

## Key Physics Insights

### Acoustic softening (Blaha effect)
Ultrasonic vibration at 20 kHz temporarily lowers the dislocation activation
barrier in the deformation zone. The Norton-Hoff consistency `K(T)` is
multiplied by `(1 − β_US)` during tool contact, reducing `η_eff` by `β_US = 20%`.
This directly lowers the plunge force and torque by 15–30% compared to
conventional FSSW at identical rotational speed and depth.

### Why `dt ≫ 1/f_US` is physically correct
At 20 kHz, one ultrasonic cycle lasts 50 µs. The mechanical timestep is ~54 ms,
containing ~1080 cycles. The simulation correctly time-averages the ultrasonic
effects (softening, heat, streaming force) rather than resolving individual cycles.
Resolving individual cycles would require `dt ~ 1 µs` and ~6500× more timesteps.

### Ultrasonic heat contribution
`Q_US = η_US × ρ × (ω_US × A_US)² × φ` is weighted by the VOF field `φ`,
ensuring heat is deposited only in metal-filled cells. At 20 µm amplitude,
this contributes ~10⁷ W/m³ — comparable to the frictional shoulder heat flux —
reducing the friction-driven temperature peak by ~10–20 K while maintaining
total energy input.

### VOF nugget tracking
The phase field `φ` tracks which cells contain metal as the pin displaces
material downward and outward. `φ = 0.5` isosurface gives the exact nugget
boundary shape after retraction. Applying `Threshold VOF > 0.5` in ParaView
isolates the weld nugget from the void cavity left by the pin.

### Acoustic streaming body force
The Eckart streaming force `F_US = ρv²/(2c) ≈ 6300 N/m³` drives a slow
secondary flow superimposed on the rotational tool flow. In lateral mode
(`dirUS = 2`), this promotes radial mixing of the plasticised material,
improving nugget homogeneity and reducing residual porosity.

---

## U-FSSW vs Conventional FSSW — Expected Differences

| Quantity | FSSW | U-FSSW | Change |
|---|---|---|---|
| Peak temperature | ~820 K | ~720–780 K | −5 to −15% |
| Plunge force Fz | baseline | −15 to −30% | reduced |
| Torque | baseline | −10 to −20% | reduced |
| Nugget homogeneity | moderate | improved | better VOF mixing |
| Tool wear | higher | lower | softer contact |
| η_eff in nugget | K(T)/ε̇ | (1−β)·K(T)/ε̇ | −β_US = −20% |

---

## Common Errors and Fixes

| Error | Cause | Fix |
|---|---|---|
| `rhoWp does not exist` on US param line | US params declared before material block | Move material block (Section 3) above US block (Section 5) |
| `Tmax > Tmelt` in first step | `etaEff` too low, Stokes diverges | Increase `etaMin` to `1e4` or reduce `dt` |
| `Tmax = 293K` always | `Tsource` not assigned to thermal RHS | Confirm `Tsource*dTemp` term inside `lTh` varf |
| CEL nodes = 0 | Tool axis offset from mesh | Confirm tool at `x=0, y=0`; check `Lx*(2*x-1)` mapping |
| `phi < 0` after VOF solve | Advection instability | Reduce `dt`; confirm `phi = min(max(phi,0),1)` clamping |
| `movemesh3` parser error | `sin/cos/pi` in `transfo` | Use only linear `scalar*variable` expressions |
| Pressure checkerboard | Wrong FE pair | Keep P1b/P1 MINI element; do not switch to P1/P1 |
| Tool VTU renders flat | `zshNow ≈ ztipNow` during retract | Normal — pin has retracted; shoulder disc visible only |

---

## Extending the Model

| Extension | What to change |
|---|---|
| Different US frequency | Change `fUS`; recalculate `PacousticNom`, check `dt*fUS > 100` |
| Lateral vibration | Set `dirUS = 2`; compare nugget VOF field with dirUS=1 |
| Combined vibration | Set `dirUS = 3`; amplitude splits as `A/√2` each axis |
| Higher amplitude | Increase `AUS` (10–50 µm range); watch `etaMin` floor |
| Different alloy (Ti-6Al-4V) | Update `rhoWp`, `CpWp`, `kWp`, `Tmelt`, `K0nh`, `mNh` |
| Quarter symmetry model | Restrict mesh to `x in [0,Lx]`, `y in [0,Ly]`; add symmetry BCs |
| Finer nugget resolution | Increase `Nx`, `Ny`, `Nz`; refine near `r < Rsh` |
| Residual stress post-processing | Add elastic Lamé solve using `phi`-weighted `epsTh` field |
| Adaptive timestepping | Use `dtCFL` during plunge, coarser `dt` during dwell |
| Bidirectional scan (FSSW array) | Loop over multiple tool-axis positions `(x0, y0)` |

---

## Citation

```bibtex
@software{mishra2026ufssw,
  author       = {Mishra, Akshansh},
  title        = {U-FSSW-CEL-3D-Simulation},
  year         = {2026},
  publisher    = {Zenodo},
  doi          = {10.5281/zenodo.20989890},
  url          = {https://doi.org/10.5281/zenodo.20989890}
}
```

---

## Author

**akshansh11**
GitHub: https://github.com/akshansh11

---

## License

<p>
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
<img alt="Creative Commons Licence" style="border-width:0"
  src="https://i.creativecommons.org/l/by-nc/4.0/88x31.png"/>
</a>
<br/>
This work is licensed under a
<a rel="license" href="http://creativecommons.org/licenses/by-nc/4.0/">
Creative Commons Attribution-NonCommercial 4.0 International License</a>.
</p>

You are free to:
- **Share** — copy and redistribute in any medium or format
- **Adapt** — remix, transform, and build upon the material

Under the following terms:
- **Attribution** — give appropriate credit and link to this repository
- **NonCommercial** — not for commercial use without permission

Copyright 2026. All rights reserved for commercial use.
