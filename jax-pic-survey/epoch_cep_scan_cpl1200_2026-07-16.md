# EPOCH CEP scan at a0=20, ND/a0=0.3, θ=45° (2026-07-16)

Trigger: Zhiping Telegram `main:6420513923:341` reported that he completed the same CEP simulation using EPOCH on tiger-vis under:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45
```

He had already run the standard pipeline to generate `reflection.nc` and `transmission.nc` for each output, and noted that tiger-vis `/scratch/gpfs/MIKHAILOVA/zl8336/Curved_surface/utility/spectrum_1D.py` is compatible with this directory. He asked to plot efficiency and `Sx` versus CEP again.

## Execution

Phase directories found:

```text
CEP=-0.75pi
CEP=-0.50pi
CEP=-0.25pi
CEP=0.00pi
CEP=0.25pi
CEP=0.50pi
CEP=0.75pi
CEP=1.00pi
```

Each contained both `reflection.nc` and `transmission.nc`.

Post-processing script written remotely:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45/utility/plot/postprocess_epoch_cep_efficiency_sx.py
```

It ran `spectrum_1D.py` for each phase/side with:

```text
HARMONIC_MIN=1.5
HARMONIC_MAX=100.5
LASER_A0=20.0
THETA_DEGREE=45.0
```

Outputs are in:

```text
/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45/utility/cep_scan_plots
/scratch/gpfs/MIKHAILOVA/zl8336/study_phy_cep/a0=20,ND_a0=0.3,45/utility/plot/plot_epoch_cep_efficiency_sx.ipynb
```

Local copies:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/epoch_cep_tiger_20260716/
```

Async job `job-5c4fdedf` completed with exit code 0. All 16 phase/side rows were `ok`.

## Metrics

HHG efficiency is integrated over `k/k0_M ∈ [1.5,100.5]`. `Sx_peak` is normalized by `laser_Sc_M = laser_Sc cos²θ`.

| φ_CEP | side | HHG efficiency | Sx_peak/laser_Sc_M |
|---:|---|---:|---:|
| -3π/4 | reflection | 0.2836035411 | 3721.167982 |
| -3π/4 | transmission | 0.05424954989 | 296.6221389 |
| -π/2 | reflection | 0.2908710145 | 3966.497609 |
| -π/2 | transmission | 0.07536919598 | 934.9702071 |
| -π/4 | reflection | 0.2715463148 | 3680.582635 |
| -π/4 | transmission | 0.08245781447 | 1229.119210 |
| 0 | reflection | 0.2657014044 | 3458.490029 |
| 0 | transmission | 0.07006929963 | 1177.123648 |
| +π/4 | reflection | 0.2715530504 | 3639.353029 |
| +π/4 | transmission | 0.06987747512 | 733.0923134 |
| +π/2 | reflection | 0.2599860062 | 2183.263818 |
| +π/2 | transmission | 0.07613260646 | 464.6671090 |
| +3π/4 | reflection | 0.2973086204 | 1908.477095 |
| +3π/4 | transmission | 0.06089450069 | 200.6626852 |
| π | reflection | 0.2736916576 | 2871.210195 |
| π | transmission | 0.05880910456 | 333.8134633 |

Extrema:
- Reflection HHG efficiency max: `φ=+3π/4`, `0.2973086`.
- Reflection `Sx_peak/laser_Sc_M` max: `φ=-π/2`, `3966.4976`.
- Transmission HHG efficiency max: `φ=-π/4`, `0.0824578`.
- Transmission `Sx_peak/laser_Sc_M` max: `φ=-π/4`, `1229.1192`.

Interpretation sent to Zhiping: as in the JAX scan, CEP changes the short-pulse carrier timing relative to the target. In EPOCH, reflection integrated HHG efficiency and local `Sx_peak` maximize at different CEPs (`+3π/4` vs `-π/2`), consistent with local peak fields being more sensitive to instantaneous waveform/spikes than the band-integrated HHG efficiency. Transmission has both metrics maximized at `-π/4`.

## Deliverables

Remote/local files:

```text
epoch_cep_efficiency_sx_vs_phi.png
epoch_cep_efficiency_sx_vs_phi.pdf
epoch_cep_efficiency_sx_report.html
epoch_cep_metrics.tsv
epoch_cep_metrics_sorted.tsv
epoch_cep_summary.json
plot_epoch_cep_efficiency_sx.ipynb
```

Telegram final delivery:
- Text report: `main:6420513923:344`
- PNG: `main:6420513923:345`
- Notebook: `main:6420513923:346`
- metrics TSV: `main:6420513923:347`
- HTML: `main:6420513923:348`
- PDF: `main:6420513923:349`
