# arXiv Daily Watchlist: Embodied-AI & SLAM Paper Tracker

> **Source**: [Embodied-AI-Daily 中文知识库](https://docs.robotnewbie.com/guide/embodied-models/Embodied-AI-Daily/README_zh.html)
> (mirror of GitHub [luohongk/embodied-ai-daily](https://github.com/luohongk/embodied-ai-daily), snapshot 2026-09-20).
> Auto-scrapes arXiv daily by keyword; each section = one keyword, top 20 in the README, up to 80 in
> the repo's `papers/` dir. Watch the repo for daily email digests.
>
> **How to use this file**: it is a *discovery layer* — when a design question touches a 2025–2026
> method not yet covered in `modern-trends.md` or the classic references, check here for candidate
> papers and their venue status before searching arXiv. Entries marked ✦ overlap with the owner's
> Notion deep-reads (see `modern-trends.md` for the analyzed versions).

---

## 1. Section map (SLAM-relevant keywords in the tracker)

| 板块 | What it tracks | Fresh examples (snapshot) |
| :--- | :--- | :--- |
| 视觉SLAM | Monocular/stereo/RGB-D systems, robustness evals | RawSLAM (HDR Gaussian SLAM from linear radiance), HuMemSLAM (human-inspired semantic place recognition), *Is Semantic SLAM Ready for Embedded Systems?* (survey), SLAM Adversarial Lab, SLAMSqueezeBench (resource-constrained comparison), ScaRF-SLAM (scale-consistent feed-forward + classical VSLAM), NGD-SLAM (real-time dynamic SLAM without GPU), GLidE-SLAM (GL-accelerated indirect-direct embedded), FastTrack (GPU tracking), Failure-or-Drift corruption benchmark |
| 视觉惯性SLAM | Multi-camera / event / underwater / edge VIs | AERO-VIS (async event-based onboard), Jetson-ORB-SLAM3 (GPU edge), PLED-VINS (point-line event, dynamic), DL-VINS-Factory (modular learned front-ends), GeoFlow-SLAM++ (multi-cam relocalization), PanoAir (panoramic UAV dataset), ✦ OKVIS2-X, ✦ Omni-LIVO, *Benchmarking Egocentric VI-SLAM at City Scale* (ICCV 2025), underwater DVL/acoustic variants |
| Visual Inertial Odometry | VIO frontends, observability, learned inertia | *Quantifying Motion Excitation for Metric Scale Observability* , Minimalist VIO, KLTNet (learned sparse tracking), STAG-VIO (prompt-to-geometry dynamic VIO), OTPL-VIO (optimal-transport line association), TANGO-VIO (triangulation-aware guaranteed observability), DINS-IO (learned inertial odometry via differentiable INS consistency), *Does Robust VIO Need More Learning? Geometry-Verified Visual Measurements under Distribution Shift* , subterranean VIO benchmark under degradation/miscalibration/occlusion |
| Lidar SLAM | Systems, backends, benchmarks, attacks | Chain-SLAM (multi-session chained loop closure), HP2-SLAM (adaptive hybrid ICP), information-aware odometry + retroactive loop closure, PinNet (learned LiDAR loop descriptors), certifiably optimal decentralized pose graph, ✦ Parameter Sensitivity Analysis, AgriGS-SLAM (orchard 4-season GS mapping), **security**: D-SLAMSpoof (dynamic point-cloud injection), MirrorDrift (actuated-mirror attacks) |
| LiDAR Odometry | Scan-matching & odom cores | TRaIL-Odom (radar-IMU-LiDAR continuous-time, adaptive Doppler weighting), LF-GICP (parameter-free degeneracy-aware via voxel-normal localizability field), FAR-LIO (high-speed), semantic-weighted ICP / SOCC-ICP (class-aware residuals), R-VoxelMap (recursive plane fitting), LIO-MARS (non-uniform continuous-time), Hi-LOAM (hierarchical implicit neural fields), Doppler Correspondence (non-iterative scan matching), ✦ DAMM-LOAM, Inland-LOAM (waterway structural semantics) |
| GNSS | Degradation, spoofing, multi-sensor heading | WinTA-GIL (GNSS-IMU-LiDAR heading refinement, intermittent signal), marker-constrained pose-graph georeferencing, spoofing detection (SLM-based, VLM-based), SASGeo (semantic-map localization for GNSS-denied UAVs) |
| 图优化 | Pose graphs, loop verification, GS+optimization | ROVER (loop verification with trajectory prior in repetitive environments), LightSplat (real-time Gaussian SLAM with loop closure), Scal3R / On-the-Fly3R (feed-forward online 3D reconstruction), Gaussian-LIC2 (LiDAR-Inertial-Camera GS SLAM, IJRR) |
| 动态SLAM | Dynamic-object SLAM | ✦ DROID-SLAM in the Wild (CVPR 2026), NGD-SLAM, DynoSLAM (GNN for social navigation), CAD-SLAM (dynamic-static decoupled mapping), D²GSLAM (4D dynamic GS), DynoSAM (open-source dynamic SLAM framework), IL-SLAM (line-assisted, feature awareness), GeneA-SLAM2, D³FlowSLAM (self-supervised flow decomposition + DINO), *The Importance of Coordinate Frames in Dynamic SLAM* |
| 语义SLAM | Semantic/open-vocabulary mapping | RADIO-ViPE & KM-ViPE (online tight vision-language-geometry, open-vocab), GS3LAM / GS4 / RoSe-SLAM (semantic GS), IRIS-SLAM (geo-instance representations), SlideSLAM (lightweight decentralized metric-semantic multi-robot), Tree-SLAM (orchard tree instance SLAM), *3D Active Metric-Semantic SLAM*, semantic VSLAM survey (2025-10) |
| Gaussian SLAM | 3DGS-based SLAM | VGGT-GS SLAM (uncalibrated mono, feed-forward priors), LightSplat, MAGS-SLAM (multi-agent), kilometer-scale outdoor mono GS SLAM, MyGO-Splat (IROS 2026 closed-loop geometric feedback), DiskChunGS (chunk-based memory), SING3R-SLAM (submap + reconstruction priors), HI-SLAM2, MG-SLAM (Manhattan world), VPGS-SLAM (voxel progressive large-scale), UP-SLAM (uncertainty-predicting dynamic GS) |
| Loop Closure Detection | Detection & verification | ROVER, Chain-SLAM, DTIF (Delaunay triangle topology, forests), PinNet, density-map loops, maximal-clique loops, FD-SLAM (radar frequency-domain), 4DRaL (4D radar→LiDAR distillation), MPTF-Net |
| 视觉场所识别 | VPR lifelong/robustness | HuMemSLAM, TRAIL (unordered databases), AdaptVPR (route-aware hard positives), lifelong aerial VPR geometric memory management, FlatVPR (geo-linear residual adapter for foundation-model manifolds), *Are VPR Models Recognizing Places or Conditions?* (condition suppression), SCD4VPR (scene change detection for DB update), token-reduction efficiency study |

Adjacent tracked sections (non-SLAM but embodied): Continuous Learning, Feed-Forward
Reconstruction (GRF-Recon, SURE-Map, RoMa-Ω), World Action Models, Gaussian Splatting,
Autonomous Driving, Kalman Filter, VLA / VLN / WAM.

---

## 2. Cross-checks against this skill's deep-reads (consistency)

The tracker independently lists several papers analyzed in `modern-trends.md`, confirming both
currency and consistency:

- ✦ **OKVIS2-X** (2025-10, IEEE T-RO) — visual-inertial section
- ✦ **Omni-LIVO** (2026-03, accepted) — both VIs and LiDAR-Odometry sections
- ✦ **DAMM-LOAM** (2025-10, accepted) — LiDAR-Odometry section
- ✦ **DROID-SLAM in the Wild** (2026-03, CVPR 2026) — dynamic-SLAM section
- ✦ **Parameter Sensitivity Analysis for Aerial LIO** (2026-09) — LiDAR-SLAM section

## 3. Trend signals worth acting on (snapshot-level reading)

1. **Feed-forward 3D priors entering classical pipelines**: ScaRF-SLAM (feed-forward models +
   classical VSLAM for scale consistency), VGGT-GS SLAM (uncalibrated mono), Topometric
   localization with feed-forward 3D models, On-the-Fly3R/Scal3R. Pattern: learned dense prior as
   initialization/prior, classical backend for consistency — same philosophy as DROID-SLAM's DBA.
2. **Gaussian SLAM industrializing**: memory management (DiskChunGS, VPGS-SLAM), loop closure
   (LightSplat), multi-agent consistency (MAGS-SLAM, GRAND-SLAM), dynamic scenes (UP-SLAM,
   D²GSLAM). The field moved from "can it track" to "can it scale and stay consistent".
3. **Degeneracy is now a first-class benchmark axis**: subterranean VIO under degradation/
   miscalibration/occlusion, LF-GICP localizability field, corruption-robustness evals (Failure-or-
   Drift, LiDAR corruptions). Matches the degeneracy-aware cluster in `modern-trends.md` §5.
4. **Embedded/resource-constraint SLAM quantified**: SLAMSqueezeBench, Jetson-ORB-SLAM3,
   GLidE-SLAM, "Semantic SLAM on embedded?" survey — the question shifted from feasibility to
   systematic comparison under budgets.
5. **Security attacks on LiDAR SLAM emerged** (D-SLAMSpoof, MirrorDrift): relevant for any
   deployed autonomous platform; threat model = adversarial point injection / mirror actuation.
6. **VPR moving to lifelong + condition-disentanglement**: geometric memory management, DB update
   via scene-change detection, condition suppression evals — pairs with the lifelong localization
   cluster in `modern-trends.md` §7.
7. **Learned front-ends packaged as drop-in modules**: DL-VINS-Factory, KLTNet, PinNet,
   E-S2Feat (event features), DINS-IO — the "swap the frontend, keep the backend" engineering
   pattern (also Super-LIO's OctVox swap) is becoming the standard integration style.

## 4. Refresh procedure

The tracker updates daily; this snapshot is 2026-09-20. To refresh:

1. Re-open the source page (browser fetch; `web_extract` may not work on this host — use the
   browser tool and `document.body.innerText`).
2. Diff new entries against §1 (titles + dates), focusing on sections matching the current task's
   modality (VIO / LIO / GS-SLAM / VPR...).
3. Promote any paper that survives 2+ weeks of community attention (acceptance at RA-L/IROS/ICRA,
   open-sourced code) into `modern-trends.md` with a proper deep-read; leave the rest here.
4. Update the snapshot date in this header.
