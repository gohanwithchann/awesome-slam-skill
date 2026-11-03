# Modern Trends Reference (2021–2026): Learned SLAM, LIVO, Degeneracy, Dynamic Removal, Lifelong Localization

> **Source note**: This module extends the classic references with recent directions curated from the
> owner's private reading notes (Notion 精读 hub「方向 · SLAM 与定位」, each with a 飞书 deep-read).
> Numbers below were re-verified against the papers during that reading; entries marked **My take**
> are the owner's critical judgments, not original paper conclusions.

---

## 1. Learned Dense SLAM: DROID family

### DROID-SLAM (NeurIPS 2021, arXiv:2108.10869)
- **Idea**: dense optical-flow revision → pose + inverse-depth update written as a *differentiable
  Gauss-Newton layer* (DBA); RAFT-style correlation volumes supply matching evidence. Trained purely
  on synthetic TartanAir (mono); EuRoC mono mean ATE 0.022 m, 11/11 zero-failure; ETH-3D 30/32 RGB-D
  sequences tracked (next best 19/32).
- **Mechanics**: states G_t ∈ SE(3) + inverse depth d_t ∈ R^(H×W); 3×3 ConvGRU updater outputs flow
  revision, per-pixel confidence w_ij and per-pixel LM damping λ (softplus, added to depth block C);
  Schur elimination shrinks the solve from 6N+HW to 6N — this is *why* DBA can live inside a network.
- **My take**:
  - Accuracy comes from the backend, not the network (odometry-only 0.186 m vs full 0.022 m on the
    same weights) — an 8.5× gap.
  - **Notation trap**: the paper writes ‖·‖_Σ with Σ=diag(w) as "Mahalanobis"; strictly it should be
    Σ⁻¹. The official code *multiplies* weights onto squared residuals. Read Σ as a *precision*
    (weight) matrix — DROID-W's叙述 depends on this reading.
  - Initialization requires ≥12 frames with flow >16 px → near-static handheld scans never
    initialize. By design, not a bug; DROID-W inherits it.
  - Weak point: V101 (weak texture + exposure swings) — correlation matches are untrustworthy and the
    system has no mechanism to detect that beyond the synthetic-data confidence w.

### DROID-W / DROID-SLAM in the Wild (CVPR 2026, arXiv:2603.19076)
- **Idea**: change only the DBA weight: Σ = diag(w·1/u′), with per-pixel dynamic uncertainty u′
  optimized *online* from cross-view cosine inconsistency of FiT3D (refined DINOv2) features.
  Bonn mean ATE 4.91 → 2.30 cm, ~10 FPS (RTX 3090). **No training** — reuses droid.pth, only a C-dim
  θ is optimized at inference.
- **My take**:
  - Soft weighting only suppresses dynamic pixels 2–4× (static u*≈1.33 vs fully dynamic u*≈4.9 at
    gamma_prior=0.05) — not "pressed to zero".
  - Structural conflict: *accurate uncertainty* vs *accurate tracking* pull in opposite directions at
    high dynamic ratio. On DyCheck each method fails somewhere (haru / pillow / block) — mono dynamic
    SLAM is not solved there.
  - Ablation reality check: the mono-depth prior (Metric3D) contributes +1.00 cm — *more* than the
    decouple+affine+weight-decay terms combined (+0.48). Missing Metric3D weights costs ~43%.
  - Gotchas: repo default `enable_loop: False` (paper accuracy relies on loop+global BA — confirm
    with authors); paper omits the depth→disparity conversion that only exists in code.
  - Beware the m0rtzz blog post on DROID-W: plausible narrative, fabricated numbers (invented TUM/KITTI
    tables). Use the paper, code, or 知乎 p/2083269308042442572.

---

## 2. VIO backends: marginalization-as-factor line (OKVIS2 → OKVIS2-X → Basalt)

### OKVIS2 (arXiv:2202.09199)
- **Idea**: Schur-marginalize sliding-window keyframes into *revivable* 6-DoF pose-graph edges whose
  weight is the marginalized information matrix H\* (not identity). On loop closure the edge can be
  "unpacked" back to landmarks + reprojection errors. EuRoC 0.031 m; TUM-VI outdoors 11.60 m vs
  ORB-SLAM3 17.87 m.
- **My take**:
  - The real innovation is **reversible information compression**, not marginalization itself.
  - Copy-paste hazard: must move the RHS term e_p,0 = −H\*⁺ b\* into the residual offset; skipping it
    yields silent drift at the window edge with small residuals — very hard to debug.
  - W = H\* double-counts observations (author-admitted inconsistency) → watch for over-tightening
    after big loops.
  - Real advantage regime = *long sequences + successful loops + slow dynamics* (clouds removed via
    Fast-SCNN sky segmentation). corridor (loses 0.09 vs 0.06) and no-cloud scenes erase the edge.
  - No monocular support (future work) — needs calibrated multi-camera + IMU.

### OKVIS2-X (T-RO 2025, arXiv:2510.04612, DOI 10.1109/TRO.2025.3619051)
- **Idea**: same paradigm extended to dense voxel submaps (frame-to-map / map-to-map alignment
  factors, depth or LiDAR), GNSS 4-DoF global factors, and online extrinsics (factor becomes
  R^(6+6N): augment the GN system with extrinsics *before* landmark marginalization so they survive
  the Schur complement). Map-factor residual normalization: e = d/σ with
  d = L/‖∇L‖, σ_map = L_min/(3‖∇L‖) + sensor σ_d (per-pixel for network depth, constant for LiDAR).
  Dense outliers use Tukey (hard rejection) not Cauchy.
- Key numbers: EuRoC VIO causal 0.066 m (OpenVINS 0.117); VI-SLAM 0.030 m (+BA 0.028);
  VBR 9 km causal VI-LiDAR 1.771 m = 0.06% of trajectory (FAST-LIVO 2.475); Hilti-Oxford 4.1 cm;
  47 Hz on i7 / GPU 3.5 GB; GNSS outage (75 s, 450 m) on 3 km: 0.169 m with final BA.
- **My take**:
  - Hilti *causal* VI-LiDAR score 245.76 **loses** to FAST-LIVO2's 402.93; the win only appears in
    non-causal + final BA (438.10). Several Chinese write-ups conflate the VBR and Hilti conclusions —
    check the scoring regime before quoting.
  - Online extrinsic ablation: no calib 0.139 → online 0.042 → offline 0.034 m (3×).
  - Memory: VBR vil 25.5 GB vs vid 10.8 GB — bottleneck is submap count × volume, not sensors;
    embedded use needs down-config.
  - Stereo ≠ wider coverage: Ours-vid(Stereo) completeness 50.60 < Mono 53.49 (11 cm baseline is
    narrow; MVS multi-view large baselines cover more).

### Basalt / Non-Linear Factor Recovery (RA-L 2020, arXiv:1904.06504)
- **Idea**: reverse the marginalized dense prior into sparse *non-linear factors* (relative pose +
  roll-pitch) via KL minimization; factor weights come from the closed-form projection
  **H_i = ({J_r Σ_o J_rᵀ}_i)⁻¹**. Ablation: identity-weight pose graph (the common practice) fails
  MH03 and degrades V103 to 0.56 — the recovered weights are the method, the factors are just the
  carrier. Global states exclude velocity & bias (2.5× smaller). Stereo-inertial only.
- **My take**: advantage grows with keyframe interval (large scenes); stereo-photometric
  calibration-sensitive; roll-pitch is a *soft* factor with covariance (better than hard-fixing);
  polar projection chosen for d→∞ numerical stability. Note paper (Schur+FEJ) vs code (square-root
  marginalization) mismatch.

---

## 3. LiDAR SLAM backend theory: BALM → BALM 2.0 → LVBA (HKU MARS)

- **BALM 1.0** (RA-L 2021, arXiv:2010.08215): feature-elimination theorem —
  min over plane params of Σ(nᵀ(p_i−q))² equals **λ₃ of the point-cluster covariance** (Rayleigh);
  optimal normal = eigenvector, optimal reference = mean. Lines: λ₂+λ₃. Only poses enter the
  optimizer. Limit: per-iteration point traversal → small windows only.
- **BALM 2.0** (T-RO 2023, arXiv:2209.08854): **Point Cluster 4×4 matrix** ℜ(C)=[P,v;vᵀ,n] with
  ℜ(TC)=Tℜ(C)Tᵀ and additive merging → cost decoupled from point count (O(1) per cluster);
  exact analytic second-order Hessian H_l = W_l + Σ_{k≠l} (2/(λ_l−λ_k)) g_klᵀ g_kl; proven gauge
  null-space. ~66× faster than Ceres (1000 frames: 1.8 s vs 120 s; 380 MB vs 4.2 GB).
  *My take*: point clusters are the **spatial analogue of IMU preintegration** — fold second moments
  into a fixed-size accumulator; a foundation for all future LiDAR BA / relocalization.
- **LVBA** (IROS 2024, arXiv:2409.10868): two-stage LiDAR-visual BA. Stage 1: BALM 2.0 geometry;
  Stage 2: homography photometric residuals on LiDAR planes + global voxel visibility map (no ray
  casting) + **relative exposure time** ε_i=τ_i/τ₁ with Σ=(1/ε_r²+1/ε_t²)I to kill the blown-out
  trivial solution. vs R³LIVE/FAST-LIVO: 15/15 wins, PSNR +3.09 dB, SSIM +123%; vs Colmap-PCD 14/15.
- **My take on the line**: two-stage = compute compromise; visual never feeds back into LiDAR poses —
  geometric degeneracy cannot be corrected by texture. Lambertian assumption fragile to specular
  paint/glass; 5 ms time offset at 5 m/s = 2.5 cm misalignment absorbed as periodic camera-pose
  deformation (time offset not optimized). Engineering: VOX_HESS/PointCluster hold fixed-size Eigen
  matrices → use `Eigen::aligned_allocator` in STL containers or OpenMP segfaults; λ₂≈λ₃ isotropic
  clusters blow up the Hessian (source guards via eigenvalue-ratio threshold).

---

## 4. LIO map structures & speed (beyond ikd-Tree)

A recurring lesson across 2024–2026 papers: **ikd-Tree's unbounded growth is the practical ceiling**
(LIMOncello measured 10.7 GB / 248 ms on OS1-128). The i-Octree / voxel-hash family is the escape:

| System | Structure | Headline numbers | Best for |
| :--- | :--- | :--- | :--- |
| **Super-LIO** (arXiv:2509.05723) | OctVox: 8 fused sub-voxels/voxel, strict density cap + HKNN search | x86 2.99 ms (F-LIO2 10.90), ARM 18.77 ms (84.74) = 4.5× | embedded CPUs; map module is drop-in replaceable for A/B |
| **Surfel-LIO** (arXiv:2512.03397) | hVox 2-level: L0 centroids only, L1 pre-computed surfels, Z-order hash | 531 FPS (AVIA) vs F-LIO2 125; NN 0.05 µs/pt vs ikd-Tree 1.42 | static scenes; degrades on dynamic pass-through (surfel staleness) |
| **LIMOncello** (arXiv:2512.19567) | SGal(3) manifold IEKF + i-Octree | only system to finish the 400 m tunnel; 14.6 ms backend | degeneracy robustness, not accuracy (wins 1/10 sequences) |
| **Super Odometry** (arXiv:2104.14938) | IMU-centric factor graph, dynamic octree, 3-axis anisotropic weights | DARPA SubT dark/fog/shaft survivor | degenerate multi-modal; borrow the E_imuodom^prior fallback term |
| **LL-Localizer** (arXiv:2504.01583) | Dynamic i-Octree, writable map, dual-mode voxel sizing via H∈[0,1] interpolation | 58.37 ms avg vs static octree 76.02 | lifelong same-site re-mapping (mowers!), map update policy k≤10 is heuristic |
| **Lightning-LM** (GitHub gaoxiang12) | ROS2 pipeline: iVox LIO + loop + 3D→2D g2p5 + sector loading + dynamic/static layers, 100 Hz | engineering product, self-reported numbers only | production skeleton; swap in your own validated front-end |

- **My take (cross-cutting)**:
  - Super-LIO keeps only means → point-to-point residuals (no plane info) — accuracy slightly traded
    for bounded memory. Its XOR-mirror 8-fold symmetry trick is worth stealing.
  - Surfel-LIO's plane-ness gate ρ=(λ₂−λ₃)/(λ₁+ε) is required; PCA always yields an eigenvector even
    for non-planar junk. After NN drops to 0.05 µs, *update* becomes the new bottleneck (0.03 ms) —
    optimize updates next.
  - LIMOncello: put gravity on **S²** (fix the 9.81 norm), not R³ — an unobservable extra DoF silently
    degrades covariance (same choice as VINS's 2-DoF gravity). SGal(3) propagation keeps
    translation-rotation coupling so covariance doesn't collapse in low observability.
  - LL-Localizer's "writable map" is the right shape for mowers (daily same plot, vegetation drift);
    its update rule can't distinguish parked-car (don't write) from new wall (write) — add a
    spatial-stability second criterion.
  - Lightning-LM's dynamic/static *layering with 3 policies* (transient / medium / permanent) is
    engineering wisdom worth copying as-is.

---

## 5. Degeneracy & adaptive registration

- **GenZ-LIO** — scene-scale metric m̄_t → **closed-loop PID on voxel size** (control target: points
  per voxel). The "parameter adaptation as control" pattern transfers to any scene-dependent tuning
  (downsample rate, residual weights, search radius). 18-dim state keeps gravity as R³ (contrast
  LIMOncello's S²). Validation gap: no citable NarrowWide numbers.
- **Azimuth-LIO** (LRA 2026, DOI 10.1109/LRA.2026.3655291): per-voxel azimuth split into N=8 sectors,
  anisotropic 3D Gaussian each, direction-weighted D2D residual + geometric-weighted Bayesian merge.
  rot01/rot02: 0.118/0.150 m vs FAST-LIO2 1.290/1.201 (8–10.9×). *Assumes motion in global xy-plane*
  → fails on slopes (re-anchor to gravity frame first). Memory: 1 m voxels OK, 0.2 m × 8 sectors =
  125× — not embedded-feasible. "Bayesian fusion" is mostly neighbor-weighted accumulation.
- **DAMM-LOAM** (arXiv:2510.13287): 5-way structural classification (ground/wall/roof/edge/non-planar)
  with per-class error metrics in weighted LS-ICP + Scan Context loop. Core borrowable frame: *choose
  error metric by geometric class* — works on any LIO (swap FAST-LIO2's uniform point-to-plane).
  Caveats: normal sign flipping (dot with viewing ray) is the make-or-break detail; per-point
  degeneracy weights and class thresholds are not published — reproducibility is low.
- **BTSA** (arXiv:2510.22313): 4D space-time plane normal's time component |d|=|sinθ| as stability
  gate *inside* the ICP iteration (no detect-then-localize circular dependency). Ground-Challenge
  ATE 0.89–1.98 m (FAST-LIO2 1.16–1.97, TRLO 1.21–8.82) but loses on open paths (3.65 vs 2.15) —
  wins only in the degenerate∩dynamic intersection. SCC (spatial consistency check) suppresses
  false positives in newly observed areas. Pairs well with map cleaners (next section): BTSA for
  odometry, DUFOMap/BeautyMap for the map layer.
- **COIN-LIO** (arXiv:2310.01235): LiDAR intensity map patches added to FAST-LIO2's ESIKF when
  geometry is degenerate. Negative result worth more than the positive: raw intensity 13.928 m →
  after FIR + range normalization 0.814 m. No camera extrinsics, no time sync, dark-proof. Hard
  requirement: high-density LiDAR (16-line intensity maps are too sparse). Complementary-selection
  ablation: random 1.913 m vs complementary 0.487 m.
- **FAST-LIVO2** (arXiv:2408.14035): sequential ESIKF update (LiDAR then image; conditionally
  independent given state — assumption weakens when LiDAR degenerates). Information-form gain
  inverts 19×19 instead of the observation dimension → 36.52 ms/frame on Hilti (LiDAR part 20.44 ms,
  faster than pure-LiDAR FAST-LIO2's 24.32). Exposure time τ in the state vector beats post-hoc
  compensation. Plane-prior warp replaces constant-depth assumption. ARM ~96 ms = no headroom.
  Pure odometry: needs loop closure for long range.
- **Omni-LIVO** (arXiv:2509.15673): omni multi-camera LIVO; explicit cross-camera temporal
  migration of map points with 5 boolean criteria; affine illumination + vignette compensation.
  Implication: run **one unified ESIKF over all cameras**, not per-camera VIOs fused later.
  Watch-out: the "exclusively visible" criterion fails in 30–50% camera overlap zones — exactly
  where migration is most frequent.
- **PL-LIVO** (RA-L 2024): point+line patch LIVO on FAST-LIVO base; **LM-VDE** assigns visual depth
  by projecting features onto LiDAR map planes (radar-agnostic). "Use the degenerate source as the
  constraint source": corridors kill LiDAR but are line-rich. Circular dependency: degenerate map →
  bad depths. Line features are scarce on grass — benefit must be measured, not assumed.
- **D-LIO** (arXiv:2505.16726): bitset-encoded truncated distance field; distance query = one AND;
  direct point-level BA (no features). L₁ distance by design (matches bitset + no sqrt). VIRAL
  0.074–0.121 m but pincio 7.39 m (structure-repeat + dynamic failure) and 20k pts = 46 ms
  (100k → 0.48 s): point-count sensitive. Distance-scaled Cauchy kernel λ(0.1+0.1‖p‖) is a nice touch.
- **QLIO** (arXiv:2503.07949): quantize *residual vectors* (not point clouds) for coprocessor links;
  quantization-aware MAP (interval likelihood, single-shot). Bit-depth ablation: 3→12 bits barely
  moves ATE — coherence matters, not bit depth. INT8 on raw observations corrupts the map
  (geometric, not photometric, error). Only pays off on dual-chip platforms.
- **RESPLE** (arXiv:2504.11580): B-spline continuous-time 6-DoF (24-dim recursive state, oldest
  control point marginalized), non-error-state IEKF. NTU VIRAL 0.022–0.052 m vs FAST-LIO2
  0.053–0.125. For bumpy platforms (spline > constant-velocity assumption); low-speed platforms gain
  little — fix calibration first.
- **Rotating-LiDAR calibration + EVA-LIO** (arXiv:2601.15946): single-DoF-arm DH model, targetless
  calibration by minimizing smallest eigenvalue (structure coincidence across rotation angles).
  Needs planes in view; degenerate mounts exist even with generic parameterization.

---

## 6. Dynamic point removal (map cleaning) & benchmarks

Unified benchmark: **ITSC23** (arXiv:2307.07260) — point-wise SA/DA with **AA = √(SA×DA)** (geometric
mean; BeautyMap uses *harmonic* mean HA — never mix the two in one table). KITTI 00: Removert
99.44/41.53/64.26, ERASOR 66.70/98.54/81.07, OctoMap+GF 93.06/98.67/95.83 (2.1 s/frame). SA/DA
trade-off is structural, not a tuning knob.

| Method | Mechanism | Numbers | Key limitation |
| :--- | :--- | :--- | :--- |
| **DUFOMap** (arXiv:2403.01449) | ray-cast void voxels; points in void = dynamic | KITTI00 98.34, 0.062 s | d_p couples to odometry error; slow-moving objects never leave a "void" → residue |
| **BeautyMap** (arXiv:2405.07283) | 2D binary occupancy matrix, bitwise-AND scan-vs-map diff | KITTI00 SA/DA/HA 96.76/98.38/97.56, 0.046 s (Python) | ground module fails on slopes (max-elevation criterion); multi-level floors structural gap |
| **BTSA** | stability gate inside registration (odometry layer) | see §5 | complements, not replaces, map cleaning |

- **My take**: DUFOMap/BeautyMap are offline/ quasi-offline cleaners ("run then clean");
  LL-Localizer is "clean while mapping" — closer to production for same-site robots. DUFOMap's
  "one set of parameters" hides the assumption that all scenes have similar odometry accuracy.
  Mower/park scenarios: slowly moving cars and pausing pedestrians are the hard residue case for
  both void-based and diff-based methods.

---

## 7. Lifelong localization & re-localization

- **LiLoc** (arXiv:2409.10172): central session + sub-sessions; adaptive submap stitching for prior,
  vertical-recognition + Scan Context XOR coarse-to-fine init, **egocentric factor graph** (IMU/LIO/
  scan factors) with marginalization. NCLT 0.726 m (FAST-LIO2 1.856, Block-Loc 0.973) at 38.99 ms.
  Risks: submap count grows unbounded (no scaling curve given); single central session = single
  point of failure; Schur-fill-in slows later solves; overlap detection uses xy only (ground-robot
  assumption).
- **LL-Localizer** (arXiv:2504.01583): see §4; the pragmatic choice for daily-same-plot robots.
- **TreeLoc** (arXiv:2602.01501): trunk instances (axis + center + DBH) as scene atoms; TDH 40-dim
  coarse + 2D triangle descriptors fine; 6DoF = 4+2 decomposition (trunk axis fixes roll/pitch).
  Cross-session R@1 0.920 vs SC++ 0.152 (note: SC++ is urban-designed; no ForestLPR cross-session
  baseline). DBH needs ≥32-line LiDAR. Methodology transferable to orchards / repeated structures.
- **Small-GICP** (registration library): header-only, covariance computed once, cache-friendly
  self-written KdTree, OpenMP throughout → 1–10 Hz global relocalization viable. Epsilon in
  diag(1,1,ε) controls in-plane slack — the most under-documented, most consequential knob.
  Relocalization's real bottlenecks remain initial value, degeneracy, and dynamic points — not
  registration speed.

---

## 8. Tuning wisdom (evidence-based)

**Aerial LIO parameter sensitivity** (arXiv:2609.12837; 60 grid-searched cases, FAST-LIO2 +
Cartographer 3D, random-forest permutation importance):
- FAST-LIO2 priority 1: `point_filter_number` & `filter_size_surfaces` (smaller = better while
  framerate holds; defaults 4 / 0.5 m → try 2 / 0.2–0.3 m).
- Priority 2: `acceleration_covariance` — raise to 0.2–0.3 with body vibration/no anti-vibration;
  keep 0.1 on clean IMU. **`gyroscope_covariance` is inert** (importance ≈ 0): don't touch.
- Priority 3: `filter_size_map` (keep 0.5 m for large scenes).
- Cartographer 3D: priority 1 `LiDAR_max_range` (20 m indoor/pipes, 50 m open); priority 2
  `low_resolution` (start 0.5 m); `rotation_weight` 100, `translation_weight` 5, `submap_size` ~1000.
- 94% of cases reach within 5 cm of exhaustive optimum by tuning only the top-2 parameters.
- Physics: gyro is integrated once (high-SNR MEMS fine); accelerometer is integrated twice + carries
  motor resonance → accel covariance decides life or death. Ground robots: bumps ≈ vibration → same
  accel-covariance logic; narrow corridors → same max-range truncation logic.

**Engineering copy list (cross-paper)**:
1. IMU short-integration prior as a degradation fallback term (Super Odometry).
2. Parameter adaptation as closed-loop control on an observable statistic (GenZ-LIO).
3. Fold second moments into fixed-size accumulators: preintegration (time), point clusters (space).
4. Marginalization bookkeeping: keep edge topology so compression can be *reversed* (OKVIS2); always
   move b\* into the residual offset (all Schur schemes).
5. Gravity: 2-DoF / S² parameterization; never estimate the norm.
6. Per-pixel/per-point σ from geometry (OKVIS2-X map factors), per-distance robust-kernel scaling
   (D-LIO).
7. Writable maps + layered dynamic/static policies for same-site robots (LL-Localizer, Lightning-LM).
8. One ESIKF over all cameras, not per-camera fusion (Omni-LIVO).
