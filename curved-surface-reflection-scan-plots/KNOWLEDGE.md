---
name: curved-surface-reflection-scan-plots
description: "Reproducible plotting workflow for Curved_surface a0=*/1D/45,thick reflection.nc scans: HHG efficiency and Sx peak amplification vs L."
version: 1.1.0
updated_at: 2026-07-06
---

# Curved_surface 1D/45,thick reflection scan plots

## Task context

Zhiping requested on Telegram (`main:6420513923:27`, follow-up `:29`) to plot two figures from Tiger data. Initial request excluded `a0=200`; later follow-up (`main:6420513923:40`, resumed at `:46`) required adding `a0=200` and using the NetCDF read/write helpers in `/scratch/gpfs/MIKHAILOVA/zl8336/EM_analyzer/read_write.py`.

- root: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface`
- data: each `a0=*/1D/45,thick/reflection.nc`
- generator reference: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/efficiency_1D_scan.py`
- plotting function required: `/scratch/gpfs/MIKHAILOVA/zl8336/EM_analyzer/plot/plot_1D.py::plot_multiple_1D_fields`
- NetCDF read/write convention: use `EM_analyzer.read_write.read_nc` / `write_fields_to_nc` rather than direct `xarray.open_dataset` in the plotting workflow
- script should live in `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility`

## Created artifacts

Remote Tiger script:

- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot_reflection_1D_scan.py`

Remote outputs:

- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_1D_scan_plots/reflection_efficiency_high_3p5_1000_vs_L_all_a0.png`
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_1D_scan_plots/reflection_Sx_peak_over_incident_vs_L_all_a0.png`
- `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_1D_scan_plots/reflection_1D_scan_summary.csv`

Local copies sent over Telegram:

- Initial (without `a0=200`):
  - `artifacts/reflection_1D_scan_plots/reflection_efficiency_high_3p5_1000_vs_L_all_a0.png` sent as Telegram document `main:6420513923:33`
  - `artifacts/reflection_1D_scan_plots/reflection_Sx_peak_over_incident_vs_L_all_a0.png` sent as Telegram document `main:6420513923:34`
  - `artifacts/reflection_1D_scan_plots/reflection_1D_scan_summary.csv` sent as Telegram document `main:6420513923:35`
  - completion/report message: `main:6420513923:32`
- Updated (with `a0=200`, using `read_write.read_nc`):
  - same local PNG/CSV paths overwritten with updated outputs
  - completion/report message: `main:6420513923:48`
  - efficiency PNG sent as Telegram document `main:6420513923:49`
  - Sx-gain PNG sent as Telegram document `main:6420513923:50`
  - CSV sent as Telegram document `main:6420513923:51`

## Data included

Included a0 directories in the updated run (all had `reflection.nc` with coordinate `L` length 51):

- `a0=3`
- `a0=10`
- `a0=20`
- `a0=50`
- `a0=100`
- `a0=150`
- `a0=200`
- `a0=250`
- `a0=400`

CSV row count after the updated run: 459 rows = 9 a0 values × 51 L values.

Variables found in every `reflection.nc`:

- `reflection_efficiency_total(L)`
- `reflection_efficiency_low(L)` = reflected efficiency [1.5, 2.5]
- `reflection_efficiency_high(L)` = reflected efficiency [3.5, 1000]
- `reflection_Sx_peak(L)` in W/m²

## Plotted quantities

Figure 1:

- x: `L/λ0` (dataset coordinate `L`, dimensionless in the scan convention)
- y: `reflection_efficiency_high`, log scale
- each curve: one `a0`

Figure 2:

- x: `L/λ0`
- y: `reflection_Sx_peak / (a0² * laser_Sc_M)`
- each curve: one `a0`

Normalization in the script matches `efficiency_1D_scan.py`:

```python
laser_lambda = 0.8 * C.micron
laser_omega0 = 2*pi*c / laser_lambda
laser_Bc = m_e * laser_omega0 / e
laser_Ec = laser_Bc * c
laser_Sc = epsilon0 * c * laser_Ec**2 / 2
theta = 45 deg
laser_Sc_M = laser_Sc * cos(theta)**2
Sx_incident = a0**2 * laser_Sc_M
```

The computed value used was:

- `laser_Sc_M = 1.068880665005e22 W/m^2` at θ=45°.

## Ranges printed by updated final run

- `a0=3`: eff_high `[2.9139e-03, 1.2258e-01]`, gain `[1.2443e+00, 1.0048e+01]`
- `a0=10`: eff_high `[1.0428e-02, 1.3137e-01]`, gain `[1.6290e+00, 6.3700e+00]`
- `a0=20`: eff_high `[1.1234e-02, 7.7777e-02]`, gain `[7.3429e-01, 7.3334e+00]`
- `a0=50`: eff_high `[4.8014e-03, 1.0461e-01]`, gain `[4.8165e-01, 1.2838e+01]`
- `a0=100`: eff_high `[4.2096e-03, 1.3534e-01]`, gain `[9.1576e-01, 2.1871e+01]`
- `a0=150`: eff_high `[4.3314e-03, 2.0717e-01]`, gain `[7.6735e-01, 2.8278e+01]`
- `a0=200`: eff_high `[1.6925e-02, 2.0807e-01]`, gain `[2.2923e+00, 2.3216e+01]`; max gain ≈ 23.2165 at `L≈0.16`
- `a0=250`: eff_high `[5.7459e-02, 1.9661e-01]`, gain `[3.3356e+00, 1.8079e+01]`
- `a0=400`: eff_high `[7.6307e-02, 1.9806e-01]`, gain `[3.3574e+00, 2.2371e+01]`

## QA / pitfalls encountered

- First variable-inspection attempt failed due shell here-doc quoting: `NameError: name 'units' is not defined`; fixed by using double quotes inside Python attributes. This was not a data problem and was reported to Zhiping in `main:6420513923:30`.
- First generated figures were valid but had large labels/legend covering data. I used vision QA, then moved legends outside axes and reduced font sizes.
- Polished rerun initially failed due matplotlib mathtext over-escaping: `ParseException ... $L/\\lambda_0$`; fixed to `$L/\lambda_0$` and reported in `main:6420513923:31`.
- Final figures passed vision QA: readable, legend outside data, correct labels/quantities.

## Interpretation note: why normalized Sx gain does not grow strongly with a0

Zhiping asked (`main:6420513923:36`) why `reflection_Sx_peak/(a0^2*laser_Sc_M)` does not show a strong monotonic increase from `a0=20` to `a0=400`.

Answer sent in `main:6420513923:37`:

- The normalized quantity asks whether reflected peak Sx grows faster than the trivial incident-intensity scaling `a0^2`. A plateau means raw `Sx_peak` roughly follows `a0^2` but lacks strong super-quadratic gain.
- In this 1D scan there is no transverse/curved-plasma-mirror focusing; the extra gain is mainly temporal compression / waveform sharpening of the reflected pulse. That factor can saturate at O(10).
- Optimizing over preplasma scale length `L` lets each `a0` find a similar effective reflecting layer in the density gradient, weakening the apparent a0-dependence. This matches the HHG wiki notes: finite gradient can weaken a0-dependence because the laser finds an effective density in the gradient.
- ROM/CSE higher `a0` can increase cutoff or alter spectrum, but `Sx_peak` is an instantaneous coherent time-domain peak; absorption, electron heating, phase noise, finite nanobunch width, multistreaming, or transparency tendencies can offset the narrower/higher-frequency spike.
- Therefore current data suggest reflected peak Sx mainly scales with incident intensity `a0^2`; the extra waveform-compression gain is nonmonotonic/saturated rather than strongly increasing with a0.

Data cited in the answer: max gain values for `a0=20,50,100,150,250,400` were approximately `7.3, 12.8, 21.9, 28.3, 18.1, 22.4`. Suggested follow-up checks: raw `reflection_Sx_peak` vs `a0`, optimal `L` vs `a0`, and fixed-`L` slices vs optimized-over-`L` comparison.

## 2026-07-06 script generalization: `scan_dict` / `ND_a0`

Zhiping asked in Telegram `main:6420513923:52` to mimic the scan-key style in `efficiency_1D_scan.py`:

```python
scan_dict = {
    'ND_a0': 'ND/a0',
    'L':     'L',
}
```

Corrected interpretation from Telegram `main:6420513923:58`: this plotting script should **not** generate/process a single `ND_a0` scan — `efficiency_1D_scan.py` already does that. The plotting script's job is to combine existing summaries from different `a0` cases into all-a0 comparison plots.

The deployed script now supports:

```bash
# Default: original L scan over all a0=*/1D/45,thick summaries.
python /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot_reflection_1D_scan.py

# All-a0 ND_a0 comparison over existing a0=*/1D/45 summaries.
python /scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot_reflection_1D_scan.py --scan-key ND_a0
```

Implementation notes:

- Uses `EM_analyzer.read_write.read_nc` for summary NetCDF reads.
- Does **not** automatically regenerate missing summary NetCDF files; it plots existing `<side>.nc` summaries. Generate summaries with `efficiency_1D_scan.py` first, then plot them here.
- Search patterns:
  - `L`: `ROOT/a0=*/1D/45,thick/reflection.nc`
  - `ND_a0`: `ROOT/a0=*/1D/45/reflection.nc`
- For mismatched scan grids (observed for `ND_a0`: most a0 have 30 points, `a0=200` has 4 points), the script forms the exact union coordinate and aligns existing points without interpolation; missing values are NaN/blank.
- Final deployed backup predecessor before corrected all-a0 version: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot_reflection_1D_scan.py.bak_20260706_101951`.

Corrected all-a0 `ND_a0` run:

- output dir: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/reflection_ND_a0_scan_plots/`
- local copies: `artifacts/reflection_ND_a0_scan_plots/`
- Telegram corrected report: `main:6420513923:60`
- Telegram corrected documents: efficiency PNG `:61`, normalized Sx PNG `:62`, CSV `:63`
- included a0 values: `3, 10, 20, 50, 100, 150, 200, 250, 400`
- CSV rows: 244 (eight a0 values with 30 points plus `a0=200` with 4 points)
- `a0=200` currently has only `ND_a0=0.05,0.10,0.15,0.20`
- vision QA: titles, axes, legends readable; legend includes all a0 including `a0=200`; no obvious clipping.

## Interpretation note: ultrathin `ND_a0` normalized gain

Zhiping asked in `main:6420513923:64` whether the corrected `ND_a0` data support the statement that, although thick-target reflected-field amplification correlates with `a0`, in ultrathin targets medium-intensity `a0` can be proportionally comparable to extreme `a0`.

Answer sent in `main:6420513923:65`:

- Yes, with the strict qualifier **proportionally / normalized gain**, not absolute reflected field.
- In the corrected `ND_a0` all-a0 CSV, max `Sx_peak/(a0² S_c,M)` values are roughly:
  - `a0=50`: 13.3
  - `a0=100`: 13.2
  - `a0=150`: 14.7
  - `a0=250`: 13.6
  - `a0=400`: 10.5
- Thus medium `a0≈50–150` can match or slightly exceed high `a0≈250–400` in normalized reflected-peak amplification.
- Mechanism: ultrathin targets are controlled more by areal-density matching / relativistic-transparency window (`ND/a0`-like similarity parameter) than by `a0` alone. If the foil remains coherently reflecting/compressing near the matched window, the extra waveform-compression gain can saturate at medium `a0`; higher `a0` can produce stronger absolute field but also transparency, electron heating, dephasing, multistreaming, or phase noise that reduce normalized peak gain.
- Caveats: raw `Sx_peak` still has the `a0²` incident-intensity base, so high `a0` absolute fields remain much larger; `a0=200` has only four `ND_a0` points and should not be treated as a complete optimum scan.

Suggested rigorous wording:

> In the optimized ultrathin-target `ND/a0` scan, reflected peak-field normalized amplification does not increase monotonically with `a0`; medium relativistic intensities can achieve comparable proportional enhancement to very high `a0`, indicating that the controlling parameter is closer to areal-density matching / transparency window than to `a0` alone.

## Interpretation note: expected CHF / focusing effect

Zhiping asked in `main:6420513923:66` how the 1D result changes if wave focusing and Coherent Harmonic Focusing (CHF) are included.

Answer sent in `main:6420513923:67`:

- 1D data suggest `Sx_peak ≈ a0² S_c × G_1D`, with `G_1D ~ O(10)` and approximately not strongly monotonic in `a0` after optimizing `L` or `ND/a0`.
- With coherent focusing, expect a multiplicative focal structure:

  ```text
  Sx_peak,focus ~ a0² S_c,M × G_1D × G_spatial × C_coh
  ```

  where `G_spatial` is transverse focusing gain and `C_coh` is the coherence/wavefront-quality factor.
- Literature grounding from the HHG wiki:
  - Gordienko et al. PRL 2005: coherent focusing of harmonics can constructively add phase-locked harmonics at focus; focal intensity scaling includes strong geometry/harmonic factors.
  - Vincenti PRL 2019 optically curved relativistic plasma mirror: total gain `Γ ~ 10³`, roughly `~5×` Doppler/temporal compression and `~200×` spatial focusing for a realistic 3 PW case.
  - Quéré/Vincenti 2021 review: p-CRM combines wavelength down-conversion, temporal compression, and wavefront curvature; ideal gain can scale strongly with relativistic factor, but wavefront quality is central.
- Therefore, if the 1D `G_1D ~ 10–15` already represents local temporal peak/compression, adding a good spatial focusing factor `~10²` gives a first-expectation normalized focal gain `Sx_peak/(a0² S_c)` of order `10³`.
- But the optimal focal peak need not occur at maximum `a0`: high `a0` gives larger absolute energy/cutoff and a stronger `a0²` base, but can degrade `C_coh` through transparency, surface breakup, electron heating, multistreaming, harmonic-dependent divergence, or phase noise. Medium-to-high `a0` may win if it preserves better coherent wavefronts while already achieving saturated `G_1D`.
- Needed diagnostics: near-field harmonic energy spectrum, phase/wavefront, angular distribution/divergence, Fourier-optics or direct field propagation to focus, and decomposition of `G_1D`, `G_spatial`, `C_coh`.

## Conference abstract read: US IFE 2026 ultrathin-foil harmonic focusing

Zhiping asked in `main:6420513923:68` to read `/home/zhiping/knowledge_base/conference/US IFE Conference/2026/book.md`, section `### Field Enhancement via Harmonic Focusing from Laser-Driven Ultrathin Plasma Foils`.

Reflection/report sent in `main:6420513923:70`:

- Main result/message: use ultrathin foil high harmonic efficiency plus target curvature / coherent harmonic focusing to obtain hundreds-fold focal intensity enhancement for X-ray/radiography source applications in ICF/HEDP.
- Method/parameter claim in abstract: PIC simulations with petawatt laser around `a0~50`, `w0~10 λ0`, `λ0=800 nm`, curved ultrathin target `d~0.04 λ0≈30 nm`, `κ~1/(200 λ0)`, yielding focal intensity `I~6×10^23 W/cm²`; 1D/2D/3D scans over `a0`, `w0`, `κ`, and `d` for robustness.
- Connection to current scans: current 1D `L`/`ND_a0` plots establish the local reflected-peak normalized gain base (`a0² × O(10)` with weak a0-dependence); the abstract's project idea multiplies this base by curved ultrathin foil spatial focusing / CHF to recover transverse focusing gain.
- Mechanistic synthesis: ultrathin foil decouples harmonic-generation efficiency from simply increasing laser intensity through `ND/a0` areal-density matching; curvature supplies wavefront focusing, so moderate-a0 wide-beam drivers can potentially reach high reflected focal intensity.
- Questions to support in future reports: define exactly what `hundreds-fold enhancement` is relative to (incident focus, unfocused reflection, same-power direct OAP focus, etc.); separate `G_1D`, `G_spatial`, and coherence/wavefront factor in plots.
- Minor language issues spotted: `hundreds of times trengthen on focal intensity` → `hundreds-fold enhancement of the focal intensity`; `an petawatt` → `a petawatt`.

## Proposed factorization for reports / next analysis

Zhiping stated in `main:6420513923:71` that the next task is to separately display `1D normalized gain`, `spatial focusing gain`, and `coherence/wavefront quality`; he identified the 1D part as enhancement from CSE radiation in an extremely short time window rather than generic temporal compression.

Response sent in `main:6420513923:72` proposed the diagnostic factorization:

```text
S_focus^peak / (a0² S_c,M) = G_CSE/1D × G_spatial × C_wavefront
```

Definitions:

- `G_CSE/1D`: local CSE / nanobunch short-time-window gain measured by 1D scans (`Sx_peak/(a0²S_c)`), currently `O(10)` and weakly a0-dependent after optimizing `L` or `ND/a0`.
- `G_spatial`: curved foil / CHF geometric focusing gain, measurable by propagating the reflected near-field to the focus and comparing source aperture to focal spot or focal intensity.
- `C_wavefront`: Strehl-like coherence/wavefront-quality penalty, `I_focus,actual / I_focus,ideal_with_same_spectrum_and_energy`, capturing harmonic phase-locking, aberration, harmonic-dependent divergence, foil breakup, and phase noise.

Suggested presentation table:

| factor | physics | how to measure |
|---|---|---|
| `G_CSE/1D` | nanobunch/CSE short-window radiation | 1D scan, near-field `Sx_peak/(a0²S_c)` |
| `G_spatial` | curved foil / CHF wavefront focusing | propagate near-field to focus, compare spot area/intensity |
| `C_wavefront` | phase locking, aberration, divergence penalty | actual focus / ideal coherent focus |

Key story: ultrathin foil gives an a0-insensitive CSE/temporal gain; curvature converts that efficient reflected harmonic field into spatially focused intensity; wavefront-quality analysis shows how close the real foil is to coherent harmonic focusing.

## Safety and side effects

- No Slurm jobs were submitted.
- No simulation data files were modified.
- Remote writes were limited to the requested utility plotting script and its output directories under `Curved_surface/utility`.
