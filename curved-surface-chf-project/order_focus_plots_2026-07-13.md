# Order-resolved focus plots vs K — 2026-07-13

Zhiping requested in Telegram `main:6420513923:224` to extend the earlier total-field plots from `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_Emax_vs_K_reflection_ND_scan_a0_20.ipynb` to order-resolved results: for three `ND/a0` values, plot 1st, 2nd, and high-order `E_max` and `x_Emax` versus curvature `K`.

## Data source and filters

Data source:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/propagate_2D_time_scan_logs/a0_20_2D_order_focus_pm100_10T_20260712_2126_order_pm100_10T/order_focus_per_band_argmax_summary_latest.tsv
```

Filters used:

```text
side = reflection
ND/a0 = 0.30, 0.50, 1.00
bands = band1, band2, band3
```

Band meanings:

- `band1`: `|k| in [0.5,1.5] k0` (1st order)
- `band2`: `|k| in [1.5,2.5] k0` (2nd order)
- `band3`: `|k| in [2.5,40.5] k0` (higher-order aggregate, not a single harmonic)

## Outputs

Rendered plots:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_Emax_vs_K_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_Emax_vs_K_ND_scan_a0_20.pdf
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_xEmax_vs_K_ND_scan_a0_20.png
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_xEmax_vs_K_ND_scan_a0_20.pdf
```

Reproducible notebook/script:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.ipynb
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/plot/plot_order_focus_vs_K_reflection_ND_scan_a0_20.py
```

Long CSV used by notebook:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/focus_summary_plots/order_reflection_band_focus_vs_K_ND_scan_a0_20_long.csv
```

Local copies for Telegram attachments:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/curved_surface_order_scan_20260712/
```

Telegram report/attachments:

- report: `main:6420513923:226`
- PNG/PDF/notebook/CSV attachments: `main:6420513923:227-232`

## Interpretation caveat

The plots use the 21-point coarse-scan sampled argmax in each band, not a continuous focus optimization. The clearest qualitative trend is that low-order `band1`/`band2` focus positions move closer to the target for stronger negative curvature (`K=-0.008`), while `band3` follows the stronger high-order/total-field focusing structure more closely. If precise per-band focus is needed, refine around each plotted argmax.
