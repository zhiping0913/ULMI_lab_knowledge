# 2D band1 focal fluence estimate — 2026-07-13

Source messages:
- Zhiping Telegram `main:6420513923:243`: requested estimate from `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=-0.005,ND_a0=0.30,L=0.00/order_focus_pm100_10T/reflection_env_band1_peak_vs_x.nc`; use `width_x` (longitudinal intensity FWHM) and `width_y` (transverse waist) at the x where `E_peak` is maximal. Stored quantity is field `E`, so square it for intensity. Compare with incident scale: longitudinal one cycle, transverse laser waist `laser_w0=12.5 λ0`, intensity `a0=20`.
- Zhiping Telegram `main:6420513923:244`: clarified `width_y` is full width while `laser_w0` is half-width, so denominator transverse width should be `2 laser_w0 = 25 λ0`.
- main reply `main:6420513923:245`.

## Data source and variable conventions

NetCDF path:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/a0=20/2D/K=-0.005,ND_a0=0.30,L=0.00/order_focus_pm100_10T/reflection_env_band1_peak_vs_x.nc
```

Variables inspected:

```text
coordinate: x_at_peak        long_name: x_at_peak / lambda_0
E_peak                       long_name: envelope peak / Ec
width_x                      long_name: peak width along x (intensity FWHM) / lambda_0
width_y                      long_name: peak width along y (1/e) / lambda_0
```

`width_x` and `width_y` are full widths in the output table. `laser_w0=12.5 λ0` is a half-width/waist parameter, so the incident transverse full width for this rectangular comparison is `2w0=25 λ0`.

## Estimate at the Emax point

The maximum `E_peak` row in this 21-point scan is:

```text
imax = 11
x_at_Emax = 68.322 λ0
y_at_Emax = -0.610 λ0
Emax/Ec = 34.6828403437
width_x = 2.62332865356 λ0
width_y = 3.65197397697 λ0
a0 = 20
laser_w0 = 12.5 λ0
```

Use the rectangular estimate requested by Zhiping:

```text
R = (Emax/Ec)^2 * width_x * width_y / [a0^2 * (1 λ0) * (2 w0)]
```

where `Emax^2` estimates intensity and the denominator is one incident longitudinal cycle times full transverse beam width.

Numerically:

```text
(Emax/a0)^2 = 3.00724853577
(width_x * width_y) / (1 * 2w0) = 0.383213119033
R = 1.1524170911
```

Therefore, at the `E_peak` focus point of this `K=-0.005, ND/a0=0.30, band1` case, the rectangular `E^2 × width_x × width_y` estimate is about **1.15 times** the incident `one longitudinal cycle × full transverse waist` fluence scale.

## Product-maximum row (context only)

If instead of fixing to the `E_peak` row one maximizes the product across the 21 scan points, the maximum rectangular product is at:

```text
x ≈ 88.43 λ0
E ≈ 24.238 Ec
width_x ≈ 2.681 λ0
width_y ≈ 8.945 λ0
R ≈ 1.4088   # after using denominator 2w0
```

This is context only; Zhiping's requested primary estimate was at the `E_peak` row.

## Interpretation: why this can exceed the 1D estimate or even the one-cycle incident reference

Follow-up from Zhiping Telegram `main:6420513923:246`: why does this 2D reflected estimate come out larger than 1D and even larger than the incident one-cycle reference? main replied in `main:6420513923:247`.

The key distinction is that this is **not** a global reflected fluence/energy integral. It is a local focal rectangular proxy:

```text
R_2D ≈ (E_peak/a0)^2 × (width_x/1λ0) × (width_y/2w0)
```

For this point,

```text
(E_peak/a0)^2 ≈ 3.01
(width_x × width_y)/(1λ0 × 2w0) ≈ 0.383
R ≈ 1.15
```

Thus the local intensity enhancement is large enough to compensate for the smaller focal area. This is expected from coherent spatial focusing: energy from a larger curved-aperture wavefront is concentrated into a local focal region. It does not by itself imply that total reflected energy exceeds incident energy.

The comparison to 1D is also not one-to-one:

- 1D `Sx` temporal-compression estimates measure time compression per transverse unit; there is no transverse focusing gain.
- 2D focal estimates mix temporal compression with spatial focusing (`G_spatial`) and wavefront/coherence quality (`C_wavefront`).
- Therefore a local 2D focal fluence proxy can exceed a 1D temporal-compression proxy without violating energy conservation.

The strict energy-accounting quantity would be something like

```text
∫∫ Sx_reflected(x,y) dx dy / ∫∫ Sx_incident(x,y) dx dy
```

computed over the same band/full field and over the whole reflected pulse/aperture. That global ratio should obey reflectance/transmission/absorption constraints (modulo particle energy exchange and numerical conventions). The current peak-width proxy is a focal brightness/concentration diagnostic, not that global energy ledger.

## Caveats

- This is a rectangular back-of-envelope estimate, not an integral over the actual 2D focal profile.
- `width_x` is intensity FWHM along propagation; `width_y` is documented as 1/e full width, not necessarily the same shape convention as a Gaussian `w0`.
- The quantity is for band1 filtered field and uses `E_peak^2` as an intensity proxy; it is not a total reflected energy accounting across all harmonics/orders.
- `E_peak` is a bandpass envelope field. Strict flux accounting should use the corresponding Poynting flux (`E×B`) or an explicitly defined envelope-intensity integral.
- The denominator is only one incident optical cycle times the reference full transverse width `2w0`; it is not the full 8 fs input pulse energy.
