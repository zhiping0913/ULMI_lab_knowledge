# 1D temporal-compression estimate from `reflection_Sx.nc` — 2026-07-13

Source messages:
- Zhiping Telegram `main:6420513923:235`: requested estimate for `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45/ND_a0=0.30/reflection_Sx.nc` using `EM_analyzer/pretreat_fields.py:get_peak_width` with FWHM (`rel_height=0.5`), comparing `Sx_peak × width` to one-cycle incident flux `a0^2 * laser_Sc_M` as defined in `utility/spectrum_1D.py`.
- Zhiping Telegram `main:6420513923:237`: clarified that `reflection_Sx.nc` horizontal axis is moving-frame `x` coordinate.
- main report `main:6420513923:238`.

## Reproducibility paths

Remote script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/estimate_1d_temporal_compression.py
```

Remote text output:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/temporal_compression_estimates/a0_20_1D_45_ND0p30_reflection_Sx_FWHM_20260713.txt
```

No Slurm job was submitted; all work was read-only except writing the small reproducibility script/output under `utility/`.

## Conventions and formula

`reflection_Sx.nc` contains:

```text
coordinate: x [m]  (moving-frame x coordinate)
field: Sx [W/m^2]
```

Constants used:

```text
a0 = 20       # from input.deck; verified line `laser_a0=20`
theta = 45°
λ0 = 0.8 μm
λ0_M = λ0 / cos45° = 1.13137085e-6 m
T0_M = λ0_M / c = 3.773846939 fs
laser_Sc_M = laser_Sc cos²45° = 1.068880665e22 W/m²
```

Important caveat: `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py` currently hard-codes `laser_a0=200.0`, which does **not** match this `a0=20` case. For this estimate, only the `laser_Sc_M = laser_Sc cos²θ` definition was reused from `spectrum_1D.py`; the denominator used `a0=20` from `input.deck`.

Because the width is a spatial width in moving-frame coordinate, the temporal width is

```text
Δt_FWHM = Δx_FWHM / c
```

and the one-cycle incident-flux comparison is

```text
R = (Sx_peak * Δt_FWHM) / (a0² * laser_Sc_M * T0_M)
  = [Sx_peak / (a0² laser_Sc_M)] * [Δx_FWHM / λ0_M]
```

## Results

Using `get_peak_width(Sx, axis=0, rel_height_each_axis=0.5, coordinate_each_axis=[x/λ0_M])`:

```text
Sx_peak = 3.330022444e25 W/m²
x_peak = 7.72593708 λ0_M
Sx_peak / (a0² laser_Sc_M) = 7.788573956

FWHM = 0.01304800652 λ0_M
     = 1.476213423e-8 m
     = 4.924117947e-17 s
     = 0.01304800652 T0_M

(Sx_peak × FWHM_time) / (a0² laser_Sc_M × T0_M) = 0.1016253638
```

Direct integration over the FWHM window gives a shape-factor cross-check:

```text
direct integral inside FWHM window / one-cycle incident = 0.07152677967
```

For context only, integrating positive `Sx` over the full x domain gives:

```text
positive_integral_all_x / one-cycle incident = 1.864936295
```

The full-domain positive integral includes all reflected structures, not only the main compressed spike, so it should not be confused with the main-spike temporal-compression estimate.

## One-cycle window around the Sx peak

Follow-up request from Zhiping Telegram `main:6420513923:239`: integrate/sum `Sx` in the interval centered on the peak with half a moving-frame period on each side.

Using `x_peak = 7.725937079592 λ0_M`, the interval was

```text
[7.225937079592, 8.225937079592] λ0_M
width = 1.000000000000 λ0_M
```

The integral results were

```text
∫ Sx dx = 2.959231872114e18 W/m
∫ Sx dt = (∫Sx dx)/c ≈ 9.87e9 J/m²
∫_{x_peak±0.5λ0_M} Sx dx / (a0² laser_Sc_M λ0_M) = 0.611765242014
```

`Sx` was positive throughout this window (`min≈3.46e23 W/m²`), so signed and positive-only integrals were identical. Equivalently, the average flux over this one-cycle window is

```text
⟨Sx⟩ / (a0² laser_Sc_M) = 0.611765242014.
```

Comparison:

```text
peak × FWHM estimate                ≈ 0.1016 one-cycle incident fluence
actual integral in FWHM window       ≈ 0.0715 one-cycle incident fluence
actual integral in ±0.5T0_M window   ≈ 0.6118 one-cycle incident fluence
```

Interpretation: a full-cycle window centered on the peak includes strong pedestal/sub-cycle structure around the main spike. It contains about six times the rectangular `peak×FWHM` estimate, but still less than one incident-cycle fluence.

## Thick-target comparison: `45,thick/L=0.10`

Follow-up request from Zhiping Telegram `main:6420513923:241`: repeat the same estimate for

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/1D/45,thick/L=0.10/reflection_Sx.nc
```

Report sent in Telegram `main:6420513923:242`.

Using the same moving-frame normalization with `a0=20`, `θ=45°`:

```text
Sx_peak / (a0² laser_Sc_M) = 3.42755703764
x_peak = 10.196214619667 λ0_M

FWHM = 0.163152482161 λ0_M
     = 0.163152482161 T0_M
     = 6.157124953605e-16 s

peak × FWHM / one-cycle incident fluence = 0.559214438439
actual integral inside FWHM window       = 0.414181320655
```

One-cycle window centered on the peak:

```text
interval = [9.696214619667, 10.696214619667] λ0_M
∫ Sx dx = 4.555184974545e18 W/m
∫ Sx dt = 1.519446154494e10 J/m²
∫ Sx dx / (a0² laser_Sc_M λ0_M) = 0.941698372686
```

`Sx` is positive throughout this one-cycle window, so signed and positive-only integrals are identical.

Thin-vs-thick comparison:

```text
thin ND/a0=0.30:
  peak/(a0²Sc_M) ≈ 7.79
  FWHM ≈ 0.0130 T0_M
  peak×FWHM ≈ 0.102
  one-cycle window integral ≈ 0.612

thick L=0.10:
  peak/(a0²Sc_M) ≈ 3.43
  FWHM ≈ 0.163 T0_M
  peak×FWHM ≈ 0.559
  one-cycle window integral ≈ 0.942
```

Physical takeaway: the thick target reflects a much larger fraction of one incident-cycle fluence (low transmission loss), but the temporal compression is much weaker: lower peak and ~12.5× broader FWHM. The ultrathin target yields a stronger, much narrower spike but smaller one-cycle reflected fluence because substantial energy is transmitted or spread into other structures.

## Physical interpretation

The 1D plasma mirror/CSE response produces a very high peak flux, roughly

```text
Sx_peak ≈ 7.8 × a0² laser_Sc_M
```

but in a very narrow temporal/spatial spike,

```text
FWHM ≈ 0.013 T0_M.
```

Thus the main compressed spike's rectangular `peak × FWHM` fluence is only about `10%` of the incident fluence in one moving-frame optical period, and the actual FWHM-window integral is about `7%`. This is a useful temporal-compression diagnostic: strong peak enhancement does not imply an order-one-cycle fluence in the main spike because the spike is extremely short.
