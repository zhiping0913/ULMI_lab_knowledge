# JAX-in-Cell final-field-only CPL convergence to 2000 (2026-07-16)

Trigger: Zhiping Telegram `main:6420513923:307-308` said he modified JAX-in-Cell source around `lax.scan` to output only the final field, using `params["solver_parameters"]["snapshot_steps"] = [steps-1]`, and had already tested the change in `/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_jaxincell_final_field.py`. He asked to continue testing higher `CELLS_PER_LAMBDA` up to 2000.

## Launch

Project root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d
```

Continuation scan root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_215429_finalonly_1200_2000
```

Launcher details:

```text
CPL list: 1200 1500 1800 2000
CUDA_VISIBLE_DEVICES=1
XLA_PYTHON_CLIENT_MEM_FRACTION=.95
LASER_A0=20.0
FWHM_T0=1.5
THETA_DEGREE=45.0
X_MIN_LAMBDA=-15
X_MAX_LAMBDA=15
```

The driver was the existing:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/run_cpl_convergence_scan.py
```

It ran `run_jaxincell_final_field.py`, then `SIDE=reflection` and `SIDE=transmission` through:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/utility/spectrum_1D.py
```

## Result

All four final-only high-CPL points completed without OOM:

| CPL | side | total_energy_norm | HHG efficiency | Sx peak | x_at_Sx_peak (m) | runner elapsed |
|---:|---|---:|---:|---:|---:|---:|
| 1200 | reflection | 0.7383639579811371 | 0.3203013189272054 | 1.7002589962022039e25 | 6.775e-06 | 1061.766 s |
| 1200 | transmission | 0.07413819949463368 | 0.06158381904213842 | 7.531846912674663e24 | 6.652333333333331e-06 | 1061.570 s |
| 1500 | reflection | 0.7392117313558291 | 0.3388317398069617 | 2.038425150152874e25 | 6.780533333333333e-06 | 1660.492 s |
| 1500 | transmission | 0.08931107530760225 | 0.07529712326067588 | 1.1728427658182589e25 | 6.653066666666666e-06 | 1660.410 s |
| 1800 | reflection | 0.7510560115348838 | 0.33071896911164483 | 2.150800690080239e25 | 6.779333333333333e-06 | 2545.385 s |
| 1800 | transmission | 0.09548003419316266 | 0.07964414585729006 | 8.490064285403201e24 | 6.665555555555557e-06 | 2545.252 s |
| 2000 | reflection | 0.733004735065134 | 0.31555589874778506 | 2.206206271641699e25 | 6.7782e-06 | 3100.427 s |
| 2000 | transmission | 0.09530951989913795 | 0.07779164784421773 | 1.0733717176730697e25 | 6.663e-06 | 3100.227 s |

Final driver status:

```json
{
  "status": "done",
  "failure": null,
  "metrics_path": "/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_20260715_215429_finalonly_1200_2000/metrics.tsv"
}
```

GPU observation during high-CPL run:
- `cpl=1200/1500`: GPU1 memory around 33 GiB during runner checks.
- `cpl=1800/2000`: GPU1 memory around 66 GiB during runner checks.
- `cpl=2000` completed and released GPU memory by final check.

## Interpretation

Mechanism:
- The previous `cpl=1200` failure in the full-output run was not a fundamental grid-size failure; it was dominated by XLA input/output argument buffer pressure from retaining too much time-history output.
- Zhiping's final-field-only `snapshot_steps=[steps-1]` change removed that memory blocker and let the scan reach `cpl=2000` on adroit-vis A100 80GB.

Numerical convergence:
- Reflection HHG efficiency is in the range `0.3156–0.3388` across `cpl=1200–2000`. The `1800→2000` change is about `-4.6%`, so integrated reflection HHG is reasonably stable at the few-percent level.
- Reflection `Sx_peak` still rises slowly (`2.1508e25 → 2.2062e25`, about `+2.6%` from 1800 to 2000), so local peak fields are less converged than integrated HHG efficiency.
- Transmission HHG is fairly stable from 1800 to 2000 (`0.07964 → 0.07779`, about `-2.3%`), while transmission `Sx_peak` remains more oscillatory/local-peak-sensitive.
- Treat final-field-only output as a good memory solution for high-resolution final diagnostics, but keep local peak metrics' resolution sensitivity in mind.

## Report artifacts

Remote combined report root:

```text
/scratch/network/zl8336/try_PIC_GPU_JAX/jaxincell_epoch1d/outputs/cpl_convergence_combined_20260716_0024_to2000_finalonly
```

Remote files:

```text
combined_convergence_report_to2000.html
combined_convergence_report_to2000.ipynb
combined_convergence_metrics_to2000.png
combined_metrics.tsv
high_cpl_metrics.tsv
adjacent_relative_changes.tsv
summary_to2000.json
```

Local copies:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/
```

Telegram final report and attachments:
- Text report: `main:6420513923:315`
- HTML: `main:6420513923:316`
- Notebook: `main:6420513923:317`
- PNG: `main:6420513923:318`
- combined metrics TSV: `main:6420513923:319`
- adjacent relative changes TSV: `main:6420513923:320`

## Plot revision: one continuous CPL curve (2026-07-16 10:07 EDT)

Trigger: Zhiping Telegram `main:6420513923:321` asked why the line segments were three separate pieces and requested plotting the three rounds together; `main:6420513923:323` clarified not to encode/care about the source.

Action: regenerated the convergence plot so all successful points are sorted by `CELLS_PER_LAMBDA` and connected as one continuous curve, with no source/run visual encoding. The panels are reflection/transmission HHG efficiency and reflection/transmission `Sx_peak`.

Local revised artifacts:

```text
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/combined_convergence_metrics_to2000_one_curve.png
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/combined_convergence_metrics_to2000_one_curve.pdf
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/combined_convergence_report_to2000_one_curve.ipynb
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/combined_convergence_report_to2000_one_curve.html
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/combined_metrics_success_one_curve.tsv
/home/zhiping/ULMI_lab/.lingtai/main/artifacts/jax_pic_adroit_20260716/cpl_convergence_to2000/adjacent_relative_changes_one_curve.tsv
```

Telegram revised delivery:
- Correction text: `main:6420513923:325`
- PNG: `main:6420513923:326`
- Notebook: `main:6420513923:327`
- PDF: `main:6420513923:328`

