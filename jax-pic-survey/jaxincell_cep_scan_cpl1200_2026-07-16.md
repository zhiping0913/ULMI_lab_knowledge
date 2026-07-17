# JAX-in-Cell CEP scan at CPL=1200 (2026-07-16)

Trigger: Zhiping Telegram `main:6420513923:329` requested a CEP scan using the JAX-in-Cell final-field-only workflow, initially saying `cpl=1000`, then corrected in `main:6420513923:331` to use `cpl=1200`. Requested phases: `0` (already done), `±π/4`, `±π/2`, `±3π/4`, `π`; diagnostics: high-order efficiency from `/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py` (now supports `HARMONIC_MIN/HARMONIC_MAX`) and `Sx_peak`, plotted in units of `laser_Sc_M`. Zhiping also asked to use both A100 GPUs.

## Execution

Remote scan root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/phi_cep_scan_cpl1200_20260716_1144
```

Driver:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_phi_cep_scan.py
```

Local artifact copy:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/phi_cep_scan_cpl1200_20260716_1144/
```

Parameters:

```text
CELLS_PER_LAMBDA=1200
LASER_A0=20.0
FWHM_T0=1.5
THETA_DEGREE=45.0
X_MIN_LAMBDA=-15
X_MAX_LAMBDA=15
HARMONIC_MIN=1.5
HARMONIC_MAX=100.5
XLA_PYTHON_CLIENT_MEM_FRACTION=0.80
XLA_PYTHON_CLIENT_PREALLOCATE=false
```

`φ=0` reused existing final-field-only cpl=1200 output:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_215429_finalonly_1200_2000/cpl_1200
```

All other phases were run in this scan using two worker threads pinned with `CUDA_VISIBLE_DEVICES=0/1`. The run completed without OOM/failure at 2026-07-16 12:56 EDT: `ok_rows=16`, `total_rows=16`.

## Metrics

HH efficiency is the energy fraction over `[1.5,100.5] k0_M`. `Sx_peak/ScM` uses `laser_Sc_M = laser_Sc cos^2(theta)` from `spectrum_1D.py`.

| φ_CEP | side | HHG efficiency | Sx_peak/laser_Sc_M |
|---:|---|---:|---:|
| -3π/4 | reflection | 0.3095436273 | 2412.589857 |
| -3π/4 | transmission | 0.05583407679 | 242.2256653 |
| -π/2 | reflection | 0.3230643365 | 1920.296741 |
| -π/2 | transmission | 0.05753149145 | 773.6483573 |
| -π/4 | reflection | 0.3519318010 | 1939.796892 |
| -π/4 | transmission | 0.09036713096 | 867.3764599 |
| 0 | reflection | 0.3203013189 | 1590.691133 |
| 0 | transmission | 0.06158381904 | 704.6480640 |
| +π/4 | reflection | 0.2830238537 | 2674.943517 |
| +π/4 | transmission | 0.08069394361 | 534.3415166 |
| +π/2 | reflection | 0.3206085564 | 2140.718933 |
| +π/2 | transmission | 0.06086584103 | 450.9463988 |
| +3π/4 | reflection | 0.3236629801 | 1869.860663 |
| +3π/4 | transmission | 0.07145676485 | 352.8376861 |
| π | reflection | 0.2671312912 | 2681.617847 |
| π | transmission | 0.05060658799 | 222.1169494 |

Extrema:
- Reflection HHG efficiency max: `φ=-π/4`, `0.3519318`.
- Reflection `Sx_peak/laser_Sc_M` max: `φ=π`, `2681.6178`; `φ=+π/4` is very close (`2674.9435`).
- Transmission HHG efficiency max: `φ=-π/4`, `0.0903671`.
- Transmission `Sx_peak/laser_Sc_M` max: `φ=-π/4`, `867.3765`.

Physical interpretation sent to Zhiping: at `FWHM=1.5T0`, CEP changes the carrier maximum timing relative to the envelope and target; integrated HHG efficiency and local `Sx_peak` need not peak at the same phase. In this scan `-π/4` is favorable for HHG efficiency on both sides, while reflection local peak is largest near `π/+π/4`.

## Deliverables

Local and remote deliverables:

```text
phi_cep_scan_cpl1200_metrics.png
phi_cep_scan_cpl1200_metrics.pdf
phi_cep_scan_cpl1200_report.html
phi_cep_scan_cpl1200_report.ipynb
metrics.tsv
metrics_sorted.tsv
metrics_summary_for_report.tsv
summary.json
scan_config.json
```

Telegram final delivery:
- Text report: `main:6420513923:335`
- PNG: `main:6420513923:336`
- Notebook: `main:6420513923:337`
- metrics TSV: `main:6420513923:338`
- HTML: `main:6420513923:339`
- PDF: `main:6420513923:340`
