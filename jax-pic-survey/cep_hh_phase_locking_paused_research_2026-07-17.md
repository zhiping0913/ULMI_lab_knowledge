---
name: cep-hh-phase-locking-paused-research-2026-07-17
description: Paused research thread on attosecond HH wavepacket carrier phase, CEP scans, ROM/CSE literature, and the sin-like vs cos-like phase question.
date: 2026-07-17
status: paused-by-zhiping
---

# CEP / HH phase-locking research thread — paused snapshot (2026-07-17)

## Trigger and status

Trigger: Zhiping Telegram `main:6420513923:399`.

Zhiping placed a toy visualization notebook on `tiger-vis`:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/harmonic_superposition.ipynb
```

Purpose of that notebook: simple visualization of how harmonic orders superpose into a waveform when the relative harmonic phases are chosen differently.

Zhiping asked to **temporarily put aside** the detailed study of the attosecond HH wavepacket carrier phase at the envelope center, but to record all work so far so the thread can be resumed later.

## Central research question to preserve

The core unresolved question is not merely “are harmonics phase-locked?” but:

> The attosecond HH wavepacket appears phase-locked, but **what absolute carrier phase is locked**, and why do PIC results over a broad parameter range show a **sin-like** attosecond pulse rather than a **cos-like** one?

Concrete tension noted by Zhiping:

- In PIC, for both the present ultra-thin target case and earlier thick-target work, the attosecond pulse / HH carrier at the envelope center appears closer to a **sin-like** wave: real field near zero at envelope/intensity maximum, analytic phase near `-π/2` depending on convention.
- In the simplified CSE / Liénard-Wiechert toy model used in Mikhailova 2012 supplementary material, the pulse is closer to **cos-like**: the real electric field peaks at the pulse center (phase 0 in that model).
- This discrepancy persists over a broad PIC parameter range, including:
  - present ultra-thin foil case: `ND/a0=0.3`, `a0=20`, `θ=45°`, directory `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45`;
  - earlier thick-target case noted by Zhiping: `a0=3`, `N=300`, `L=0.05`, original simulations on `tiger-vis` under `/scratch/gpfs/MIKHAILOVA/zl8336/45thick`.

Future resolution likely requires looking at electron trajectories, source/current terms, boundary response, and propagation/filtering phases — not only final far-field spectra.

## Work completed so far

### 1. JAX-in-Cell convergence / final-field-only workflow

Detailed note: `jaxincell_cpl_to2000_finalonly_2026-07-16.md`.

- Zhiping changed JAX-in-Cell `lax.scan` output to final-field only (`snapshot_steps=[steps-1]`).
- Main continued convergence on `adroit-vis` to `CELLS_PER_LAMBDA=1200,1500,1800,2000` without OOM.
- Key `cpl=2000` values: reflection HHG efficiency `0.31556`, reflection `Sx_peak=2.206e25`, transmission HHG `0.07779`, transmission `Sx_peak=1.073e25`.
- Interpretation: previous OOM at high CPL was mainly from full-output XLA/I/O memory pressure; final-only output removes that blocker through `cpl=2000`. Local peak fields remain more resolution-sensitive than integrated HHG.

### 2. JAX-in-Cell CEP scan at CPL=1200

Detailed note: `jaxincell_cep_scan_cpl1200_2026-07-16.md`.

- Fixed `CELLS_PER_LAMBDA=1200` and scanned `φ_CEP = 0, ±π/4, ±π/2, ±3π/4, π` using final-field-only JAX-in-Cell on both A100 GPUs on `adroit-vis`.
- All 16 reflection/transmission metric rows completed without OOM.
- HH band used in post-processing: `[1.5,100.5] k0_M`; `Sx_peak` normalized by `laser_Sc_M`.
- Key extrema:
  - reflection HHG max at `φ=-π/4` (`0.35193`);
  - reflection `Sx_peak/ScM` max at `φ=π` (`2681.62`, close to `+π/4`);
  - transmission HHG and `Sx_peak/ScM` both max at `φ=-π/4` (`0.09037`, `867.38`).

### 3. EPOCH CEP post-processing for the same nominal case

Detailed note: `epoch_cep_scan_cpl1200_2026-07-16.md`.

- Processed EPOCH outputs under `/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45`.
- Each CEP directory already contained `reflection.nc` and `transmission.nc`.
- Ran Curved_surface `utility/spectrum_1D.py` for all 8 phases × reflection/transmission with HH band `[1.5,100.5] k0_M` and `Sx_peak` normalized by `laser_Sc_M`.
- Key extrema:
  - reflection HHG max at `φ=+3π/4` (`0.29731`);
  - reflection `Sx_peak/ScM` max at `φ=-π/2` (`3966.50`);
  - transmission HHG and `Sx_peak/ScM` both max at `φ=-π/4` (`0.08246`, `1229.12`).
- Comparison caveat: JAX and EPOCH both show CEP-sensitive short-pulse timing, but extrema differ; JAX final-field workflow is a reduced/faster model and not yet a quantitative EPOCH replacement.

### 4. CEP harmonic phase locking at reflection HH envelope peak

Detailed note: `cep_harmonic_phase_locking_2026-07-16.md`.

- Zhiping asked whether per-harmonic phases at the reflection HH envelope peak are locked near `-π/2`, and whether JAX-in-Cell high-order detuning worsens this.
- I patched EPOCH and JAX copies of `spectrum_1D.py` only to save numeric `<side>_per_harmonic_at_peak.tsv` files; backups were kept.
- Main delivery conclusion at that time: EPOCH reflection harmonics show approximate locking near a common phase close to `-π/2` under the current analytic-signal diagnostic at the broadband HH envelope peak. Weighted mean phase was roughly `-0.40π` to `-0.46π`, with deviation `0.07π–0.12π`. JAX CPL=1200 showed the same tendency but with larger scatter and shorter strong-order extent, consistent with high-order detuning / weak high-order coherence degrading first.

### 5. ROM/CSE literature and phase-reference caveat

Detailed note: `cep_phase_reference_rom_cse_2026-07-17.md`.

Zhiping challenged whether `-π/2` has a direct physical meaning. I checked:

- BGP / ROM: Baeva, Gordienko, Pukhov 2006, DOI `10.1103/PhysRevE.74.046404`;
- sliding mirror / phase locking: Pirozhkov 2006, DOI `10.1063/1.2158145`;
- CSE / synchrotron emission: Mikhailova 2012, DOI `10.1103/PhysRevLett.109.245005`;
- front-surface CSE trajectories: Cousens 2020, DOI `10.1103/PhysRevE.101.053210`;
- local HHG wiki / KB / LightRAG leads;
- `spectrum_1D.py` and `EM_analyzer.spectrum` analytic-signal convention.

Refined interpretation:

- In the literature, “phase locking” usually means **linear spectral phase / common emission time**, not a universal absolute phase intercept.
- BGP gives a high-order phase mainly tied to emission time / gamma spike, e.g. single-spike behavior approximately `φ_n ≈ -n Θ_1(t_g)`, not a universal `φ0`.
- Pirozhkov defines perfect phase locking as spectral phase linear in frequency; in time domain, different frequencies are emitted together.
- Current `spectrum_1D.py` samples `np.angle(field_analytic)` at the broadband HH envelope `peak_id`, after constructing an analytic field with `h=1+sign(k)`. Under this convention, a sin-like carrier at the envelope center naturally appears near `-π/2`.
- Therefore, the observed common `-π/2` is currently a **diagnostic / reference statement** about the HH wavepacket carrier phase at the envelope center, not a universal ROM/CSE mechanism constant.

### 6. Real-field-peak phase diagnostic and correction

Detailed note: `epoch_cep_real_peak_phase_fit_2026-07-17.md`.

Zhiping requested a diagnostic that samples per-harmonic phases at the nearest `|E_forward_hh_real|` peak to the broadband HH envelope peak, unwraps phases, and fits strong harmonics. I implemented this in tiger-vis `spectrum_1D.py` temporarily:

- production backup before patch:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260717_1300_real_peak_phase`;
- outputs per CEP case:
  - `reflection_per_harmonic_at_real_peak.tsv`,
  - `reflection_real_peak_phase_fit_summary.tsv`,
  - `reflection_per_harmonic_at_real_peak.png`;
- combined remote artifacts:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/epoch_cep_real_peak_phase_20260717/`;
- local mirror:
  `artifacts/epoch_cep_real_peak_phase_20260717/`.

Key result:

- nearest real-field peak is only `12–20` cells after the HH envelope peak (`0.0042–0.0071 T0_M`, lab `~0.003–0.005 T0`), with `|E_HH_real|/Ec_M≈18–34`;
- strong-harmonic linear fits used `fit_count=99` and had weighted wrapped residual RMS `0.045π–0.091π`.

Zhiping then pointed out the essential correction: the slope at the real-field peak is just a sampling-point translation. Moving the sample by `Δx` adds `n k0 Δx`; removing the fitted slope is essentially shifting the reference back to the envelope / group-delay center. Thus the real-peak diagnostic is only a sanity check, not a replacement for envelope-peak phase locking.

Production script state after this discussion:

- Zhiping asked to revert production `spectrum_1D.py`.
- I restored `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py` from `spectrum_1D.py.bak_20260717_1300_real_peak_phase`; `python3 -m py_compile` passed.
- The real-peak diagnostic copy was preserved as:
  `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py.bak_20260717_1316_before_revert_real_peak_phase`.

### 7. Mikhailova 2012 CSE toy model phase 0 vs PIC `-π/2`

Detailed note: `mikhailova2012_cse_carrier_phase_2026-07-17.md`.

Zhiping asked how to reconcile the PIC/EPOCH HH wavepacket carrier phase near `-π/2` at the envelope center with Mikhailova 2012 supplement Fig. S5, where the simplified CSE model field peaks at the pulse center (phase 0).

Key facts preserved:

- Supplement Fig. S2(d): reflected electric field (cyan) and filtered intensity `|E(t)|^2` for `ω>80ω0` (red). Qualitative figure inspection suggested the PIC reflected field is near a zero crossing at the intensity/envelope marker.
- Supplement text around Fig. S3: at the emission time `t3`, `J_z` is maximal in absolute value and `J_y` is zero.
- Fig. S5 simplified model writes approximately `E(t)= -Ne a_y(t') U(t')/(c^2 R)`, with Gaussian/even `a_y` and positive/even `U`. This construction naturally produces a centered single-sign / cos-like pulse.

Working interpretation:

- CSE conditions fix emission time, envelope center, and spectral phase slope, not a universal carrier-phase intercept.
- Mikhailova Fig. S5 phase 0 follows from the simplified analytic model and its chosen phase reference.
- Self-consistent PIC reflected fields can have extra phase from source/boundary response, current vs acceleration variables, spectral filtering, finite bunch structure, and propagation.
- Therefore our PIC `-π/2` is not automatically inconsistent with CSE; it may mean the self-consistent reflected-field HH carrier is sin-like at the envelope center, while the toy model captures envelope timing/intensity but not spectral phase intercept.

## Current best summary to resume from

If we return to this project, start from this compact statement:

1. The robust evidence so far is **small residual / near-linear spectral phase** of strong harmonics at the HH envelope/group-delay center.
2. The common phase intercept near `-π/2` in EPOCH is a **carrier CEP of the reconstructed HH wavepacket under the current analytic-field convention**, not yet a mechanism-level universal constant.
3. A real-field-peak sampling point produces a linear phase slope consistent with ordinary translation `Δφ_n=n k0 Δx`; removing that slope returns the reference to the envelope center.
4. The open physics problem is why self-consistent PIC reflected HH fields in broad regimes appear sin-like at the envelope center, while simplified CSE/LW acceleration-field toy models are cos-like.
5. The likely missing pieces are electron trajectory/source diagnostics and the mapping from source variables (`J_y`, `dJ_y/dt`, bunch acceleration, surface current, boundary response) to the far-field/reflected HH waveform.

## Future work when resumed

1. Use `harmonic_superposition.ipynb` as a toy visualization only; it can clarify how relative phase intercept changes the waveform from cos-like to sin-like, but it cannot by itself explain the PIC source physics.
2. For the present ultra-thin case, extract electron/bunch diagnostics around the HH emission time if available: trajectory, `J_y`, `J_z`, `dJ_y/dt`, bunch gamma spike, surface density/current, and reflected field at several observation points.
3. For the earlier thick-target case (`/scratch/gpfs/MIKHAILOVA/zl8336/45thick`, `a0=3`, `N=300`, `L=0.05`), locate the original diagnostics and verify that the same sin-like HH carrier behavior appears using the same phase convention.
4. Derive or numerically test how the CSE/LW source field transforms into the measured reflected field in the PIC diagnostic: sign convention, time/space reference, boundary condition, propagation phase, spectral filter, and analytic-signal phase.
5. Keep production `spectrum_1D.py` envelope-peak-only unless Zhiping asks for another diagnostic; put exploratory phase diagnostics in separate notebooks/scripts to avoid perturbing production post-processing.

## Retrieval keywords

CEP scan; JAX-in-Cell; EPOCH; CPL=1200; final-field-only; harmonic phase locking; `-π/2`; sin-like attosecond pulse; cos-like CSE toy model; Mikhailova 2012; BGP ROM; Pirozhkov sliding mirror; Cousens CSE; analytic signal; envelope peak; real-field peak; group delay; `harmonic_superposition.ipynb`; `45thick`.
