---
name: awesome-slam
description: Architectural decision guide, algorithmic taxonomy, mathematical formulation, and engineering workflows for SLAM (Simultaneous Localization and Mapping), Visual Odometry (VO), Visual-Inertial Odometry (VIO), and LiDAR SLAM derived from the curated Awesome-SLAM collection and academic literature. Use when designing, selecting, implementing, or debugging SLAM systems, choosing between Monocular, Stereo, RGB-D, Visual-Inertial (ORB-SLAM3, VINS-Fusion, Basalt, SVO, DSO), or LiDAR pipelines (Cartographer, LOAM, LeGO-LOAM, LIO-SAM, FAST-LIO), configuring factor graph optimization (g2o, GTSAM, Ceres), applying Lie theory (SO(3), SE(3), Sophus, manif), camera/IMU calibration (Kalibr), or integrating learned and semantic SLAM. Also covers modern trends (2021-2026): learned dense SLAM (DROID-SLAM, DROID-W), marginalization-as-factor backends (OKVIS2, OKVIS2-X, Basalt NFR), LiDAR bundle adjustment (BALM, BALM 2.0, LVBA), modern LIO map structures (Super-LIO, Surfel-LIO, LIMOncello, FAST-LIVO2, Omni-LIVO), degeneracy-aware registration (GenZ-LIO, Azimuth-LIO, BTSA, COIN-LIO), dynamic point removal (DUFOMap, BeautyMap, ITSC23 benchmark), lifelong localization (LiLoc, LL-Localizer, TreeLoc), and evidence-based LIO parameter tuning.
---

# Awesome SLAM: Architecture, Optimization & System Design

Architectural guide, algorithm decision matrix, mathematical foundations, and engineering workflows derived from the curated `SilenceOverflow/Awesome-SLAM` repository and peer-reviewed SLAM literature.

---

## 1. SLAM Modality Decision Matrix

Select the optimal SLAM pipeline based on sensor payload, environmental constraints, compute budget, and accuracy requirements:

| Sensor Modality | Primary Architectures | Compute Profile | Best Suited For | Critical Limitations & Failure Modes |
| :--- | :--- | :--- | :--- | :--- |
| **Monocular Vision** | **[ORB-SLAM3](references/visual-slam.md)**, **[DSO](references/visual-slam.md)**, **[SVO](references/visual-slam.md)** | Low to Medium (Embedded CPU) | Lightweight drones, micro-robotics, AR headsets | Scale ambiguity, metric drift, scale drift without prior baseline, initialization delay |
| **Stereo Vision** | **[ORB-SLAM2/3](references/visual-slam.md)**, **[PL-SLAM](references/visual-slam.md)**, **[OV²SLAM](references/visual-slam.md)** | Medium (CPU / Multi-core) | Autonomous driving, industrial AGVs, outdoor robotics | Fixed baseline limits maximum metric depth range; feature dropout in low-texture areas |
| **RGB-D** | **[ORB-SLAM3](references/visual-slam.md)**, **[RESLAM](references/visual-slam.md)**, **[VDO-SLAM](references/visual-slam.md)** | Medium (CPU + GPU for dense) | Indoor navigation, service robots, manipulation | Outdoor sunlight saturates IR projector; limited to 0.5m–6m range; specular/dark surfaces |
| **Monocular + IMU (Mono VIO)** | **[VINS-Mono](references/visual-inertial-slam.md)**, **[ROVIO](references/visual-inertial-slam.md)**, **[OpenVINS](references/visual-inertial-slam.md)** | Medium (Real-time CPU) | Mobile AR (ARKit/ARCore), agile FPV quadrotors | Requires excitation during initialization to estimate metric scale and accelerometer bias |
| **Stereo + IMU (Stereo VIO)** | **[VINS-Fusion](references/visual-inertial-slam.md)**, **[ORB-SLAM3](references/visual-inertial-slam.md)**, **[Basalt](references/visual-inertial-slam.md)** | Medium to High (Multi-threaded) | Robust UAV flight, high-dynamic ground vehicles | Sensor synchronization latency, camera-IMU calibration drift over thermal fluctuations |
| **3D LiDAR (LIO / LOAM)** | **[FAST-LIO](references/lidar-slam.md)**, **[LIO-SAM](references/lidar-slam.md)**, **[LeGO-LOAM](references/lidar-slam.md)** | Medium (Embedded CPU like Jetson) | All-weather autonomous driving, forestry, mining | Long symmetric tunnels / open fields (geometric degeneracy); heavy sensor weight & cost |
| **2D LiDAR** | **[Cartographer](references/lidar-slam.md)**, **[Gmapping](references/lidar-slam.md)** | Low (Raspberry Pi class) | Indoor vacuum robots, planar warehouse AGVs | 2D planar assumption fails on slopes, ramps, or uneven terrain |
| **Semantic & Learned SLAM** | **[DS-SLAM](references/learning-and-semantic-slam.md)**, **[SuMa++](references/learning-and-semantic-slam.md)**, **[Pop-up SLAM](references/learning-and-semantic-slam.md)** | High (GPU Acceleration) | Dynamic urban environments with pedestrians/cars | High GPU power consumption, latency in real-time edge processing |
| **Dense Learned SLAM (GPU)** | **[DROID-SLAM / DROID-W](references/modern-trends.md)** | High (GPU ≥8 GB, RTX-class) | Robustness-critical tracking, dynamic scenes, offline SfM-grade accuracy | ~10–20 FPS, GPU power; initialization needs ≥12 frames with flow >16 px; mono scale unobservable |
| **Multi-modal LIVO (LiDAR-Visual-Inertial)** | **[FAST-LIVO2](references/modern-trends.md)**, **[Omni-LIVO](references/modern-trends.md)**, **[PL-LIVO](references/modern-trends.md)** | Medium-High (i7/Orin) | Degenerate geometry (dark, corridors) where LiDAR or vision alone fails | Sequential-update conditional independence weakens under LiDAR degeneracy; ARM headroom is thin |
| **Modern LIO (2024–26 map structures)** | **[Super-LIO](references/modern-trends.md)**, **[Surfel-LIO](references/modern-trends.md)**, **[LIMOncello](references/modern-trends.md)**, **[COIN-LIO](references/modern-trends.md)** | Low-Medium (embedded CPU) | Long-duration runs (ikd-Tree's 10.7 GB growth is the ceiling these escape) | Super-LIO trades plane info for bounded memory; COIN-LIO needs ≥32-line LiDAR |
| **Degeneracy-aware / Dynamic-robust LIO** | **[GenZ-LIO](references/modern-trends.md)**, **[Azimuth-LIO](references/modern-trends.md)**, **[BTSA](references/modern-trends.md)**, **[DAMM-LOAM](references/modern-trends.md)** | Low-Medium | Tunnels, rotation-heavy scenes, dynamic pedestrians on roads | Most assume ground-plane (xy) motion — fails on slopes without gravity re-anchoring |
| **Lifelong / Map-maintenance Localization** | **[LiLoc](references/modern-trends.md)**, **[LL-Localizer](references/modern-trends.md)**, **[TreeLoc](references/modern-trends.md)** | Low-Medium | Same-site daily re-mapping (service/mowing robots), forest global localization | Central-session growth unbounded (LiLoc); update policies can't separate parked cars from real changes |

---

## 2. Canonical SLAM Pipeline Architecture

A production-grade modern SLAM system is partitioned into asynchronous, decoupled threads:

```
[Sensors: Camera / IMU / LiDAR]
           │
           ▼
┌──────────────────────────────────────────────┐
│  FRONTEND (High Frequency: 30–200 Hz)        │
│  • IMU Preintegration on SO(3)/SE(3)         │
│  • Feature Extraction (ORB/FAST/Lines) OR    │
│    Direct Photometric Pixel Tracking         │
│  • Short-term Motion Prior & Odometry        │
└──────────────────────┬───────────────────────┘
                       │ Keyframe Decision / Scan Selection
                       ▼
┌──────────────────────────────────────────────┐
│  BACKEND (Low Frequency: 1–10 Hz)            │
│  • Sliding-Window Factor Graph Optimization   │
│  • Visual Reprojection & Inertial Factors    │
│  • Marginalization (Schur Complement Prior)  │
│  • Online Extrinsic & IMU Bias Estimation    │
└──────────────────────┬───────────────────────┘
                       │
         ┌─────────────┴─────────────┐
         ▼                           ▼
┌─────────────────────────┐ ┌─────────────────────────┐
│ LOOP CLOSING (0.1–1 Hz) │ │ GLOBAL MAPPING          │
│ • DBoW2 / Scan Context  │ │ • Sparse / Dense Mesh   │
│ • Geometric Verification│ │ • OctoMap / Surfel Map  │
│ • Sim(3) Pose Graph Opt │ │ • Metric-Semantic Octree│
└─────────────────────────┘ └─────────────────────────┘
```

---

## 3. Mathematical Foundations & Cheat Sheet

### 3.1 Lie Groups and Lie Algebras: $SO(3)$ and $SE(3)$

Rotations and rigid body transforms form smooth Riemannian manifolds:

- **$SO(3)$ Rotation Group**: $R \in \mathbb{R}^{3 \times 3}, \quad R R^T = I, \quad \det(R) = +1$
- **$\mathfrak{so}(3)$ Lie Algebra**: Skew-symmetric matrices $[\boldsymbol{\omega}]_\times \in \mathbb{R}^{3 \times 3}$:
  $$[\boldsymbol{\omega}]_\times = \begin{bmatrix} 0 & -\omega_3 & \omega_2 \\ \omega_3 & 0 & -\omega_1 \\ -\omega_2 & \omega_1 & 0 \end{bmatrix}$$
- **Exponential Map (Rodrigues' Formula)**:
  $$\exp([\boldsymbol{\omega}]_\times) = \cos(\theta) I + (1 - \cos(\theta)) \mathbf{a}\mathbf{a}^T + \sin(\theta) [\mathbf{a}]_\times, \quad \theta = \|\boldsymbol{\omega}\|, \quad \mathbf{a} = \frac{\boldsymbol{\omega}}{\theta}$$
- **Perturbation Model on Lie Manifold**:
  Avoid over-parameterization by applying small perturbations in the tangent space:
  $$R_{new} = R \cdot \exp([\delta \boldsymbol{\phi}]_\times) \approx R (I + [\delta \boldsymbol{\phi}]_\times)$$

### 3.2 Non-Linear Factor Graph Optimization

SLAM states $\mathcal{X} = \{ \mathbf{x}_0, \mathbf{x}_1, \dots, \mathbf{x}_n \}$ are solved via Maximum A Posteriori (MAP) estimation:

$$\mathcal{X}^* = \arg\min_{\mathcal{X}} \left\{ \sum_{k} \|\mathbf{r}_{prior, k}\|^2_{\Sigma_k} + \sum_{i,j} \|\mathbf{r}_{visual}(i,j)\|^2_{\Sigma_{ij}} + \sum_{k} \|\mathbf{r}_{imu}(k, k+1)\|^2_{\Sigma_k} \right\}$$

- **Normal Equations (Gauss-Newton)**:
  $$(J^T \Sigma^{-1} J) \Delta \mathcal{X} = - J^T \Sigma^{-1} \mathbf{r} \implies H \Delta \mathcal{X} = \mathbf{b}$$
- **Levenberg-Marquardt Damping**:
  $$(H + \lambda I) \Delta \mathcal{X} = \mathbf{b} \quad \text{or} \quad (H + \lambda \operatorname{diag}(H)) \Delta \mathcal{X} = \mathbf{b}$$
- **Marginalization via Schur Complement**:
  To bound CPU memory in sliding-window estimators, marginalize older poses/landmarks $\mathbf{x}_m$ while preserving cross-correlations onto remaining states $\mathbf{x}_r$:
  $$\begin{bmatrix} H_{mm} & H_{mr} \\ H_{rm} & H_{rr} \end{bmatrix} \begin{bmatrix} \Delta \mathbf{x}_m \\ \Delta \mathbf{x}_r \end{bmatrix} = \begin{bmatrix} \mathbf{b}_m \\ \mathbf{b}_r \end{bmatrix}$$
  $$H^* = H_{rr} - H_{rm} H_{mm}^{-1} H_{mr}, \quad \mathbf{b}^* = \mathbf{b}_r - H_{rm} H_{mm}^{-1} \mathbf{b}_m$$

---

## 4. Engineering & Implementation Workflows

### Workflow 1: Sensor Calibration and Synchronization
Before compiling or running any SLAM algorithm, calibrate sensors:
1. **Camera Intrinsics & Distortion**:
   - Calibrate using OpenCV / Kalibr with a checkerboard or AprilGrid.
   - For wide-angle / fisheye lenses ($> 120^\circ$ FOV), use Kannala-Brandt (`radtan` vs `equidistant`) distortion models.
2. **Camera-to-IMU Extrinsics ($T_{c}^{b}$) & Time Offset ($t_d$)**:
   - Use **[Kalibr](https://github.com/ethz-asl/kalibr)** with synchronized dataset recording continuous excitation on all 3 rotational axes.
   - Run Allan variance tools (**[kalibr_allan](https://github.com/rpng/kalibr_allan)** or `imu_utils`) to obtain continuous gyroscope/accelerometer noise densities ($q_g, q_a$) and random walk parameters ($q_{bg}, q_{ba}$).

### Workflow 2: Selecting and Deploying a VIO Framework
1. **Real-time Embedded CPU (Raspberry Pi 4 / Drone FCU)**:
   - Choose **ROVIO** (Iterated Extended Kalman Filter with photometric patches) or **MSCKF_VIO**.
2. **Standard Robotics Platform (Intel NUC / Jetson Xavier / Orin)**:
   - Choose **VINS-Fusion** or **ORB-SLAM3**.
   - Configure stereo + IMU for instant initialization without metric scale ambiguity.
3. **High-Accuracy Mapping / Factor Graph Recovery**:
   - Choose **Basalt** (extracts non-linear factor representations and supports multi-camera visual-inertial bundle adjustment).

### Workflow 3: Diagnosing and Fixing Tracking Loss
1. **Pure Rotation Tracking Loss**:
   - Feature-based monocular SLAM fails during pure rotation due to zero parallax (cannot triangulate depth).
   - *Fix*: Switch to Visual-Inertial fusion (IMU handles gyroscope rotation directly) or use a 2-point RANSAC rotation estimator.
2. **Motion Blur in Low Light**:
   - Features smear, causing FAST/ORB detector dropout.
   - *Fix*: Integrate Line features (**PL-VIO**, **StVO-PL**) or increase camera exposure time limit paired with hardware trigger.
3. **Scale Drift in Monocular Pipelines**:
   - Monocular SLAM accumulates scale drift over loops.
   - *Fix*: Enable loop closure with $Sim(3)$ optimization (**ORB-SLAM3**) or incorporate metric depth sensors / IMU.
4. **LiDAR Degeneracy (tunnels, open fields, long corridors)**:
   - Point-to-plane ICP loses rank in symmetric/featureless geometry; the Hessian's minimum eigenvalue → 0.
   - *Fix (graded)*: add IMU short-integration prior as fallback term (**Super Odometry** pattern); add intensity-map photometric patches (**COIN-LIO**, needs ≥32-line LiDAR); use line features (**PL-LIVO** — corridors are line-rich); consider SGal(3) propagation (**LIMOncello**) which keeps covariance consistent under low observability.
5. **Dynamic Contamination (pedestrians, vehicles)**:
   - Detect-then-remove at the *map* layer: void-voxel ray-casting (**DUFOMap**), bitwise-AND occupancy diff (**BeautyMap**), unified benchmark **ITSC23** with AA = √(SA×DA) (never mix with harmonic-mean HA).
   - For the *odometry* layer: stability-gated residuals inside registration (**BTSA**) avoid the circular detect→localize dependency.
   - Known residue: slow-moving objects (parked-then-moved cars, pausing pedestrians) defeat both void and diff criteria — combine with writable-map policies (**LL-Localizer**).
6. **ikd-Tree Unbounded Memory Growth (long-duration LIO)**:
   - Measured 10.7 GB / 248 ms on OS1-128 (**LIMOncello**).
   - *Fix*: voxel-hash family with bounded density — **Super-LIO** OctVox (4.5× ARM speedup), **Surfel-LIO** hVox (pre-computed surfels, 0.05 µs/pt NN), or **Lightning-LM** iVox pipeline.

---

## 5. Detailed Reference Guides

Explore the modular domain references in the `references/` directory:

- 📘 [Visual SLAM Reference](references/visual-slam.md) — Monocular, Stereo, and RGB-D algorithms (ORB-SLAM3, DSO, SVO, OpenVSLAM).
- 🧭 [Visual-Inertial SLAM Reference](references/visual-inertial-slam.md) — Preintegration, sliding-window filtering, MSCKF, VINS-Mono/Fusion, Basalt.
- 📡 [LiDAR-based SLAM Reference](references/lidar-slam.md) — 2D/3D LOAM, LeGO-LOAM, LIO-SAM, FAST-LIO, Cartographer.
- 📐 [Optimization & Math Reference](references/optimization-and-math.md) — Lie groups, factor graphs, g2o, GTSAM, Ceres, and calibration toolkits.
- 🧠 [Learning-based & Semantic SLAM Reference](references/learning-and-semantic-slam.md) — SuperPoint, deep features, dynamic object filtering, semantic surfel mapping.
- 🚀 [Modern Trends (2021–2026) Reference](references/modern-trends.md) — DROID-SLAM/DROID-W learned dense SLAM, OKVIS2/OKVIS2-X/Basalt marginalization-as-factor line, BALM/BALM2.0/LVBA LiDAR BA theory, modern LIO map structures (Super-LIO, Surfel-LIO, LIMOncello, COIN-LIO, FAST-LIVO2), degeneracy-aware registration, dynamic point removal (DUFOMap, BeautyMap, ITSC23 benchmark), lifelong localization (LiLoc, LL-Localizer, TreeLoc), and evidence-based FAST-LIO2/Cartographer tuning.
- 📚 [Local Offline Papers Catalog](references/papers-catalog.md) — Complete registry of the 111 downloaded research papers and documents stored locally in the skill's [`papers/`](papers/) directory.

---

## License & Attribution

This skill is derived from the open-source repository `SilenceOverflow/Awesome-SLAM`, curated by Youjie Xia and licensed under the **MIT License**.
Individual research papers, algorithms, and libraries retain their respective authors' copyrights, open-source licenses, and institutional attributions.
