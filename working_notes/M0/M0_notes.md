# Reading for M0

## 1. Executive summary

LOWSUN is a proposed open-source pipeline for producing labeled synthetic imagery for planetary surface and descent vision. Its purpose is to demonstrate technical depth relevant to space-autonomy, planetary-perception, and simulation-infrastructure roles while answering a focused research question:

> When do physically and geometrically accurate camera and sensor models materially improve performance on real planetary imagery?

The broader concept is valid and well aligned with real work in terrain-relative navigation (TRN), hazard detection and avoidance (HDA), camera calibration, synthetic-data generation, and autonomy validation. NASA describes TRN and hazard detection as enabling technologies for precise landings, and JPL's ALHAT work explicitly combines high-fidelity Monte Carlo simulation with field validation. Current autonomy roles similarly emphasize geometric vision, camera calibration, synthetic data, simulation, localization, testing, and production software.

The original M−1 through M8 plan, however, is closer to a small research program than a reliable one-person portfolio project. Its nominal milestones total roughly 45 weeks before allowing for integration, data problems, renderer behavior, scientific review, documentation, or publication delays. Attempting the entire roadmap at once creates a high risk of producing an impressive specification but an incomplete implementation.

The initial project will therefore be a smaller, complete vertical slice:

1. A quantitative study of CAHVORE non-centrality using real public camera calibrations.
2. One lunar site rendered through one well-characterized camera.
3. An end-to-end path from terrain and pose to linear radiometric output, camera projection, sensor output, and labels.
4. Quantitative validation of geometry and radiometry.
5. One real-versus-synthetic localization or feature-matching benchmark.

Completion of this scope is sufficient for LOWSUN to serve as a substantial portfolio centerpiece. Farm-scale generation, permanently shadowed regions, Mars, additional sensors, and full ML training remain follow-on work rather than prerequisites.


## 3. Problem statement

Planetary landing and surface mobility systems must perceive terrain under conditions for which representative flight imagery is sparse or nonexistent. Terrain-relative navigation, hazard detection, visual odometry, and map-relative localization must contend with unfamiliar terrain, unusual illumination, imperfect maps, wide-angle cameras, sensor noise, and limited opportunities for field testing.

Synthetic imagery can help train and validate these systems, but only if its limitations are known. A visually plausible render is not automatically a faithful representation of a flight camera. Differences in geometry, radiometry, temporal sampling, optics, and sensor response can introduce systematic errors that undermine conclusions drawn from the data.

LOWSUN will investigate those differences rather than assume that additional simulation complexity is useful. The first release will concentrate on two linked questions:

1. **Geometric fidelity:** At what ranges and field angles does a central camera approximation cease to be adequate for a real CAHVORE camera?
2. **Operational value:** Do the modeled differences affect feature matching or map-relative localization on real planetary imagery?

## 4. Project objective

Build and publicly document a reproducible lunar-imagery generation and evaluation pipeline that:

- Ingests one public lunar terrain product.
- Resolves physically meaningful sun and camera geometry.
- Produces raw linear renderer output under an explicitly documented radiometric convention.
- Projects imagery through a validated CAHV/CAHVOR/CAHVORE implementation.
- Applies a transparent, calibrated or explicitly representative sensor model.
- Produces depth, surface-normal, slope, shadow, and pose labels through the same camera abstraction.
- Compares central and non-central camera paths quantitatively.
- Evaluates at least one downstream vision metric on public real imagery.
- Can be reproduced in a pinned software environment from published configuration and provenance records.


### 6.1 CAHVORE non-centrality study

CAHVORE extends CAHVOR with an entrance-pupil model. Its `E` terms represent movement of the apparent entrance pupil as a function of viewing angle. That movement produces range-dependent parallax and cannot, in general, be represented exactly by a single-viewpoint renderer.

CAHVORE is not a single model. A type parameter `T` selects among CAHVORE-1, -2, and -3, and a linearity parameter `P` governs the type-3 case, where `0.0` is equivalent to CAHVORE-2 and `1.0` to CAHVORE-1. The three types have different entrance-pupil behaviour, so the study's question has a different answer for each. Every figure, threshold, and conclusion must be attributed to a specific type; a camera with non-zero `E` may still behave centrally depending on its type.

The study will distinguish two effects:

- **Lens distortion:** Range-independent image warping that can be reproduced by rendering a suitable central panorama or cubemap and resampling it.
- **Non-centrality:** Field-angle-dependent ray origins that create range-dependent parallax and require true per-pixel ray origins for exact rendering.

The comparison will include:

1. True CAHVORE rays as the reference.
2. A best-fit central CAHVOR model plus resampling.
3. A naive pinhole model as a baseline.

The central approximation must be fitted rather than defined as “all rays begin at `C`,” because a fitted central model can absorb part of the discrepancy into its distortion parameters.

### 6.2 Experiments

**Experiment A — Residual versus range**

- Use two or three public calibrations, prioritizing cameras with meaningful `E` terms.
- Evaluate a log-spaced range envelope appropriate to surface and descent imagery.
- Report residual by field angle, including boresight, mid-field, and corners.
- Verify that the residual trend agrees with the first-order relationship between focal length, transverse pupil displacement, and range.

**Experiment B — Operationally weighted fit**

- Define a realistic but clearly documented descent or surface range distribution.
- Fit a central model over samples drawn from that distribution.
- Report residual over the same operating envelope.

**Experiment C — Propagation to navigation metrics**

- Propagate correspondence error into at least one operational metric.
- Preferred metric: map-relative localization position and attitude error.
- Optional metrics: triangulation error or PnP sensitivity.

**Experiment D — Render-path comparison**

- Compare true non-central ray rendering with central rendering plus resampling.
- Measure interpolation and sample-density errors as well as non-centrality.
- Determine whether resampling error dominates camera-model error in the selected envelope.

### 6.3 Preregistered decision rule

Before examining final results, record a threshold for accepting the central approximation. The original provisional value of approximately `0.05 px` may be retained only after documenting why it is appropriate for the chosen feature detector and task.

The decision must be expressed in operational terms:

- If the central approximation remains below the threshold across the target envelope, use the higher-throughput central render-and-resample path.
- If it exceeds the threshold, use a renderer capable of true per-pixel origins for the affected operating regime.
- Preserve both paths when they serve different regimes, rather than declaring one universally correct.


## 9. Milestones and acceptance criteria

### Milestone 0 — Evidence and data audit (1 week)

**Outputs**

- Selected lunar site and terrain product.
- Two or three candidate camera calibrations, each recorded with its **CAHVORE type `T` and linearity parameter `P`** in addition to the C, A, H, V, O, R, E components.
- Data-rights and attribution ledger.
- Target real sequence for evaluation.
- Defined downstream task and metric.
- Recorded CAHVORE decision threshold.
- Dependency decision record (see 9.0.1).
- Reference detector ordered for the Milestone 3b sensor validation (see below). Acquisition only; no measurement work at this stage.

**Acceptance criteria**

- Every input has a stable citation, known format, and documented usage terms.
- For each candidate calibration, `T` and `P` are recorded and the expected entrance-pupil behaviour of that specific model type is stated. A non-zero `E` vector alone is **not** sufficient evidence of non-central behaviour: `T` determines which CAHVORE variant applies, and `P` governs the type-3 case, where `0.0` is equivalent to CAHVORE-2 and `1.0` is equivalent to CAHVORE-1. Selecting a camera on `E` alone risks studying a model that behaves centrally.
- At least one selected calibration has non-negligible non-centrality **given its type**, or the study records that no suitable case was found.
- The real-data evaluation is plausible before renderer development begins.

This milestone prevents the project from building a generator and only later discovering that the intended benchmark has no usable real data or ground truth.

#### 9.0.1 Dependency decisions

Recorded during Milestone 0, before implementation begins. The governing rule:

- **Reimplement** what constitutes the contribution.
- **Depend on or vendor** what is necessary but incidental.
- **Fork** only where the project will extend another product in the same direction — not expected to apply here.

| Component | Decision | Rationale |
|---|---|---|
| CAHV/CAHVOR/CAHVORE projection and ray generation | Reimplement from primary sources | The M1 result rests entirely on this implementation. It must be the project's own, validated against independent implementations and analytical expectations. Implement from Gennery, *Computations for Generalized Camera Model Including Entrance Pupil Movement* (2001) and *Generalized Camera Calibration Including Fish-Eye Lenses* (2002, JPL clearance 03-0869) — not from a third-party port. |
| PDS calibration-label ingest | Evaluate an existing Python package before writing one | Label parsing is incidental effort. `marsimage` exposes a CAHVORE dataclass parsing ODL `GEOMETRIC_CAMERA_MODEL` groups including `E`, `T`, `P` and coordinate-frame handling. Assess maintenance status, licence, and correctness against a known label, then depend or vendor. |
| Photogrammetric ↔ CAHVOR conversion | Vendor or depend, with attribution | Needed to configure a renderer camera from a CAHV model and to fit a central model back out (Experiments B and D). `bvnayak/CAHVOR_camera_model` implements Di & Li (2004) under BSD-2-Clause, which is compatible with Apache-2.0 use. Open issues are unresolved; verify before relying on it. |
| Independent verification oracles | Locate during Milestone 0 | JPL's `cmod` calibration library, if an open release exists, is the authoritative reference; the PDS geometric camera model documentation points to the JPL calibration references page. Any third-party implementation serves as a secondary cross-check. |

**Acceptance criterion for this sub-item:** every external dependency is recorded with its licence, maintenance status, the specific function it serves, and what the project does if it proves incorrect.
