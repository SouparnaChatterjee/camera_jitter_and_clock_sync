# Timestamp Health in Multi-Sensor Driving Data

**Measuring jitter, drift and offset on KITTI, and what they imply for ROS 2 recording pipelines**

Souparna Chatterjee · Draft v0.1 · Preprint (not peer reviewed)

*Project 1 of a three-part study of silent failure modes in robot data pipelines.*

---

## Summary

Multi-sensor perception stacks associate camera frames, LiDAR sweeps and inertial data by timestamp. When a stamp is wrong, nothing fails: the data is simply mis-associated. This report measures three timing faults (offset, drift and jitter) on three KITTI raw drives (16–33 s) and on synthetic faults with known ground truth.

### Key findings

- **Real data is not as clean as the camera suggests.** Camera and LiDAR stamps are tight (robust interval jitter 0.19 ms and 0.08 ms), but the OXTS inertial stream contains 3 backward timestamp steps, 10 repeated stamps and 2 gaps.
- **Nearest-timestamp pairing breaks under dropped frames.** With 2–10 % dropped frames the drift-estimate error std grows from about 8 ppm to 219–446 ppm. A simple 30 ms pairing gate restores 8–10 ppm.
- **Short sessions cannot resolve small drifts.** In simulation, resolving a 2 ppm drift needs roughly 300 s of data. A health report should be able to say "UNRESOLVED (session too short)".
- **Cost of an uncorrected offset:** 0.45 cm of position and 0.009° of attitude per millisecond on drive 0005 (RMS speed 4.63 m/s).
- **Nearest-neighbour offsets overstate jitter.** Matching to a slower stream (10 ms IMU) produces a saw-tooth of ±5 ms that is quantisation, not clock jitter.
- **Camera–IMU lag: an unverified result.** A gyro-versus-visual yaw estimator reports a consistent apparent lag of 64–78 ms across three drives. This conflicts with the dataset's hardware synchronisation and changed sign when intrinsics were corrected, so it is reported as an unverified estimator output, not a finding about KITTI.

The ROS 2 / MCAP implications in Section 5 are reasoned hypotheses, not measurements. Measuring them is the purpose of Project 3.

## Repository contents

```
.
├── timestamp_health_kitti.tex     # LaTeX source of the report
├── fig1_intervals.png             # Inter-sample intervals (camera, IMU, LiDAR)
├── fig2_offsets.png               # Nearest-neighbour offsets to LiDAR and IMU
├── fig3_slope.png                 # Camera-to-LiDAR offset vs time, LS fit
├── fig4_drops_and_length.png      # Dropped-frame error; session length vs drift noise
├── fig5_correlation.png           # Gyro/visual yaw correlation vs candidate lag
├── fig6_cost.png                  # Cost functions after calibrated intrinsics
└── README.md
```

> The analysis notebook is not yet included. See [Reproducibility](#reproducibility).

## Building the paper

Place the `.tex` file and the six `fig*.png` files in the same folder, then run:

```bash
pdflatex timestamp_health_kitti.tex
pdflatex timestamp_health_kitti.tex   # second pass for cross-references
```

It uses only standard packages (`amsmath`, `booktabs`, `graphicx`, `caption`, `enumitem`, `microtype`, `hyperref`, `lmodern`) and compiles on Overleaf or a standard TeX Live install. If a figure file is missing, a labelled placeholder box is shown instead of failing the build.

## Data

The experiments use three [KITTI raw](https://www.cvlibs.net/datasets/kitti/raw_data.php) drives from 2011-09-26:

| Drive | Frames | Duration | Used for |
|-------|--------|----------|----------|
| `2011_09_26_drive_0005` | 160 | 16.4 s | All LiDAR experiments, camera–IMU estimate |
| `2011_09_26_drive_0011` | 239 | 24.6 s | Camera–IMU replication |
| `2011_09_26_drive_0014` | 320 | 33.0 s | Camera–IMU replication |

**KITTI data is not redistributed here.** Download it from the KITTI website; it is licensed under CC BY-NC-SA 3.0.

## Methods at a glance

- **Stamp hygiene:** inter-sample intervals, non-positive intervals, gaps (>15 ms on the 10 ms IMU stream), repeated stamps with payload comparison; robust jitter = 1.4826 × MAD.
- **Offset and drift:** nearest-stamp pairing, median offset, least-squares slope with block-bootstrap interval (blocks of 10); Theil–Sen as a robust comparison; pairs >30 ms from the median rejected as mis-pairs.
- **Fault injection:** known offset, linear drift (ppm), Gaussian jitter and dropped frames (isolated or bursts of 3), fixed seeds, 100–200 Monte-Carlo runs.
- **Offset cost:** shifted-stamp pose error from OXTS, compared with the analytic *v · Δt*.
- **Camera–IMU lag:** ORB features, essential matrix with RANSAC on calibrated intrinsics, visual yaw increments vs integrated gyro yaw over candidate shifts of ±120 ms.

## Limitations

- Short, hardware-triggered, clean drives; real multi-hour sessions without a trigger may behave very differently.
- Injected jitter is Gaussian and drift linear; real faults are structured.
- The camera–IMU result is **unverified** and may reflect systematic estimator bias.
- The fusion proxy uses OXTS yaw as truth from the same IMU as the gyro, single run.
- The health-score weights are uncalibrated; report the components and the drift verdict instead.
- No ROS 2 measurements are included.

## Reproducibility

All results come from a single Python notebook run on CPU (Kaggle) using NumPy, SciPy, pandas, OpenCV and matplotlib, with fixed random seeds for injection and bootstrap.

- Notebook and figure files: **https://github.com/SouparnaChatterjee/camera_jitter_and_clock_sync**
- Archive: **@misc{chatterjee_timestamp_health,
  author = {Chatterjee, Souparna},
  title  = {Timestamp Health in Multi-Sensor Driving Data: Measuring Jitter, Drift and Offset on {KITTI}, and What They Imply for {ROS}~2 Recording Pipelines},
  note   = {Draft v0.1, preprint (not peer reviewed)},
  doi    = {[Zenodo DOI to be added]}
}**

Figures 1–3, 5 and 6 are direct notebook output; Figure 4 is plotted from the values in Tables 3 and 4.

## The three-part study

1. **Project 1 (this repo):** timestamp health: jitter, drift, offset.
2. **Project 2:** silent data corruption and dataset hygiene (checksums, near-duplicates, split leakage, label conventions).
3. **Project 3:** provenance, calibration tracking and ROS 2 latency.

Each project is written to stand alone.

