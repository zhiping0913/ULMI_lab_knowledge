# Figure / claim map — Curved_surface ultrathin-foil CHF project

Use this file to connect figures to scientific claims, evidence level, and missing controls. This prevents mixing CSE gain, spatial focusing, and wavefront quality into one ambiguous enhancement number.

## Central planned figure: factorization roadmap

Claim:

```text
S_focus^peak/(a0^2 S_c,M) = G_CSE/1D × G_spatial × C_wavefront
```

Figure concept:

| Panel | Quantity | Evidence source | What it proves | Missing / needed |
|---|---|---|---|---|
| A | `G_CSE/1D` vs `L` or `ND/a0` for multiple `a0` | 1D summaries and plots | Local normalized reflected peak can reach O(10), weak optimized-a0 dependence | Confirm conventions and numeric table; include caveat for sparse `a0=200`. |
| B | Near-field reflected/harmonic wavefront and spectrum for curved foil | 2D/3D outputs, not yet inventoried | Curved foil produces usable harmonic field with phase/wavefront information | Need paths, diagnostics, harmonic filter definition. |
| C | Propagated focal intensity / spot size | 2D/3D propagation or direct focal diagnostics | Curvature adds transverse spatial focusing `G_spatial` | Need propagation method and reference plane. |
| D | Actual focus vs ideal coherent focus | Actual field vs ideal same-spectrum/energy coherent reference | Quantifies `C_wavefront` / Strehl-like wavefront penalty | Need precise ideal-reference construction. |

## Existing figures / outputs

### All-a0 ND/a0 efficiency plot

File:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_ND_a0_scan_plots/reflection_efficiency_high_3p5_1000_vs_ND_a0_all_a0.png
```

Claim support:

- Shows high-harmonic/reflection efficiency trends versus `ND/a0` across available `a0` cases.
- Supports areal-density matching / transparency-window framing.

Caveats:

- Need verify exact frequency/harmonic integration range definition (`3.5–1000`) when writing paper text.
- `a0=200` sparse.

### All-a0 ND/a0 normalized peak plot

File:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_ND_a0_scan_plots/reflection_Sx_peak_over_incident_vs_ND_a0_all_a0.png
```

Claim support:

- Direct evidence for `G_CSE/1D` style normalized local reflected peak gain.
- Supports the careful statement that medium `a0` can be comparable to very high `a0` in normalized gain, not absolute intensity.

Caveats:

- Need preserve normalization exactly: `Sx_peak/(a0^2 * laser_Sc_M)` / `S_c,M` convention.
- Do not infer absolute field equality between different `a0`.
- Do not fabricate `reflection_Sx` from naive `Ey*Bz/mu0`.

### CSV summary

File:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_ND_a0_scan_plots/reflection_1D_ND_a0_scan_summary.csv
```

Use:

- Quantitative source for values in plots and tables.
- Should be read before quoting exact maxima.

### 1D-vs-2D reflection spectrum comparison at `a0=20`, `D=0.02`

Files:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan_long.csv
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_ND0p30_vs_2D_D0p02_K_scan.ipynb
```

Claim support:

- Diagnostic comparison of normalized reflected spectrum shape/content between the 1D `ND/a0=0.30` reference and 2D reflected radial spectra for `D=0.02,L=0` across `K=0,-0.002,-0.005,-0.008`.
- Useful for panel B / near-field spectrum comparison in the factorization roadmap.

Caveats:

- The 1D coordinate variable is `f_over_f0_M`; 2D radial coordinate is `k_rho_over_k0`. Both are normalized. After Zhiping's style instruction (`main:6420513923:120`), the rendered x-axis label is `k/k0` and the y-axis label is `I/I0`.
- Legend entries intentionally omit `D=0.02` because this case corresponds to the `ND/a0=0.30` condition; Zhiping noted the actual input-deck thickness is `D=0.017142857142857144`.
- This plot does not prove focal gain `G_spatial` or wavefront quality `C_wavefront`; it only compares spectra before propagation/focal diagnostics.

### 1D-vs-2D reflection spectrum comparison across multiple `ND/a0` values at `a0=20`

Files:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_compare_plots/reflection_spectrum_1D_vs_2D_ND_scan_a0_20_long.csv
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/reflection_spectrum_1D_vs_2D_ND_scan_a0_20.ipynb
```

Source:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_reflection_spectrum_compare_a0_20.py
```

Claim support:

- Diagnostic 2×2 comparison of normalized reflected spectra for `ND/a0=0.10,0.30,0.50,1.00` at `a0=20`.
- Each panel compares the corresponding 1D reference to 2D radial spectra for `K=+0.000,-0.002,-0.005,-0.008`.
- Supports selecting/understanding areal-density dependence of spectral content before focusing/wavefront diagnostics.

Caveats:

- The script computes the 2D directory `D` from `D = ND_a0*a0/N` and `%3.2f` formatting, with current `N=350`; do not turn this into a memorized correspondence table.
- Still only spectral-shape evidence. It does not establish `G_spatial` or `C_wavefront` without propagation/focal-plane or ideal-focus diagnostics.
- Legends identify `1D` and `K` only; panel titles carry `ND/a0` and formatted `D`.

## Claims that still need evidence

1. `G_spatial ~ 10^2` or other magnitude for this specific curved ultrathin foil setup.
   - Need 2D/3D propagation or direct focal diagnostics.
   - Literature p-CRM gives useful scale but not proof for this system.

2. `C_wavefront` close to unity or acceptable.
   - Need actual vs ideal coherent focus comparison.
   - Need harmonic phase/wavefront diagnostics.

3. Hundreds-fold focal enhancement reference.
   - Need define denominator: incident focus intensity, reflected unfocused intensity, same-power direct OAP focus, or another reference.

4. Moderate `a0` advantage after CHF.
   - Need maximize `a0^2 × G_CSE/1D × G_spatial(a0) × C_wavefront(a0)`, not just compare `G_CSE/1D`.
