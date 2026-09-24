# LOWSUN — Portfolio-Focused Project Scope

**Status:** Proposed  
**Project type:** Open-source planetary vision and synthetic-data portfolio project  
**Primary domain:** Planetary autonomy, terrain-relative navigation, perception simulation, and validation  
**Initial duration:** Approximately 15–21 weeks of nights-and-weekends work  
**Long-term direction:** An extensible, validated image-formation pipeline for planetary and orbital vision

**Name and identity**

- Project and package name: `lowsun` (previously REGOLITH, renamed before first commit)
- Tagline: *measured image formation for planetary and orbital vision*
- Rationale: names the illumination regime where planetary surface vision is hardest — terminators, polar landing sites, permanently shadowed rims, raking light for hazard detection — without locking the project to one body, one surface material, or one milestone. Deliberately avoids the `-synth` suffix, which would echo NASA JPL's LuNaSynth and frame the work as generation rather than measurement.
- Namespace status at time of decision: PyPI `lowsun` unclaimed; two repositories on GitHub carry the name. Verify `lowsun.dev` at a registrar and claim the PyPI name before first release.

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

## 2. Why this is a sensible portfolio project

LOWSUN demonstrates several relevant capabilities in one coherent system:

- **Planetary-domain knowledge:** PDS products, SPICE geometry, lunar terrain, illumination, and flight-camera calibration.
- **Procedural terrain synthesis:** statistically grounded surface detail generation with validated recovery of its own input distribution.
- **Geometric vision:** projection models, non-central cameras, distortion, ray generation, pose, reprojection, and localization.
- **Imaging physics:** reflectance, finite-source illumination, exposure, photon statistics, read noise, and quantization.
- **Detector characterisation:** empirical photon-transfer measurement of a real sensor and comparison against both a published characterisation and the project's own image-formation model.
- **Scientific validation:** controlled comparisons, preregistered decision thresholds, analytical checks, ablations, and explicit failure reporting.
- **Production engineering:** declarative scenes, deterministic seeds, manifests, provenance, checksums, resumability, tests, and documented environments.
- **Communication:** a visual demonstration supported by a concise technical report and defensible quantitative results.

The combination is more valuable than a visually impressive renderer alone. It shows an ability to connect mathematical models, physical sensors, real mission data, reliable software, and operational metrics.

This scope is most directly relevant to roles in:

- Spacecraft or aerial autonomy
- Planetary perception and computer vision
- Terrain-relative or map-relative navigation
- Modeling and simulation
- Synthetic-data and ML infrastructure
- Camera systems and calibration
- Robotics validation and test infrastructure

The initial release models framing cameras only. It is therefore not directly applicable to pushbroom or line-scan orbital imagers, where each image line carries its own pose and jitter is a first-class term — a class that includes most modern orbital and Earth-observation instruments, and indeed the stereo products from which the project's own terrain inputs are derived. This limit should be stated rather than left to be discovered.

It is less directly targeted at propulsion, structures, thermal engineering, orbital mechanics, mission operations, or general flight-software roles. LOWSUN should therefore be presented as a deliberate specialization in autonomy, perception, and simulation—not as a generic space portfolio project.

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

## 5. Non-goals for the initial release

The initial project will not attempt to be:

- A closed-loop spacecraft simulator.
- A flight-software-in-the-loop or hardware-in-the-loop system.
- A custom renderer.
- A general-purpose perception library.
- A production GPU farm for millions of frames.
- A complete model of permanently shadowed-region illumination.
- A Mars atmospheric renderer.
- A multi-sensor simulator covering lidar, thermal, event cameras, or multispectral systems.
- A full sim-to-real ML training program across multiple perception tasks.
- A replacement for PANGU, SurRender, LuNaSynth, or mission-specific internal tools.
- A claim of flight qualification or mission readiness.

These exclusions are intentional. The initial release is designed to be finished, validated, understandable, and useful as evidence of engineering judgment.

## 6. Core research contribution

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

## 7. Initial technical scope

### 7.1 Scene and terrain

- One lunar site with a public, citable terrain product.
- Tiled terrain ingest from the beginning; no monolithic mesh dependency.
- Documented coordinate reference system, body-fixed frame, units, datum, and vertical convention.
- A procedural sub-DTM detail layer (see 7.1.1).
- One base reflectance model and one simple baseline, such as Hapke versus Lambert.
- Uniform or documented albedo for the first release; spatial albedo recovery is deferred.
- Sun represented as a finite angular source.
- No atmosphere.

#### 7.1.1 Procedural sub-DTM detail

Public lunar DTMs at metre-scale posting are smooth at the scale that hazard detection and close-range vision operate on. Without a synthesised detail layer, slope and shadow labels describe a surface lacking the features those labels exist to detect, and any hazard product is measuring an artefact of the DTM resolution rather than terrain.

The detail layer is therefore in scope for the initial release, but bounded:

- **Rock population** generated from a published lunar rock size-frequency distribution, parameterised by cumulative fractional area and a minimum modelled diameter. Placement seeded and recorded in the scene configuration.
- **Regolith roughness** as a fractal or equivalent displacement layer, with amplitude and spectral slope documented and sourced.
- **Rock instance identity assigned at scatter time**, not recovered from the render. Renderer-derived instance masks, where available, serve only as a cross-check.

Explicitly deferred: crater degradation modelling, boulder fields and ejecta structure, regolith depth variation, and site-specific morphology beyond the selected distribution. One statistically justified distribution at one site is sufficient for the initial release.

**Statistical validation.** The synthesised population must be measurable back out of the scene. Recovering cumulative fractional area and the size-frequency slope from the generated geometry, and comparing against the input parameters and the published distribution, is a required acceptance test. A detail layer that looks plausible but does not reproduce its own input statistics is not usable as label ground truth.

The scope limit to state publicly is that the detail layer is *statistically representative of a published distribution*, not a reconstruction of the actual surface at the selected site.

### 7.2 Camera

- A camera interface used by image formation and every geometry-dependent label.
- Validated CAHV, CAHVOR, and CAHVORE projection and ray generation, implemented from primary sources and carrying the CAHVORE type `T` and linearity parameter `P` explicitly.
- Float64 arithmetic throughout the camera path, with a measured implementation error floor.
- Round-trip and edge-of-field tests using real calibration parameters.
- Central approximation fitting and resampling tools.
- Explicit camera and world frame conventions.
- Synthetic checkerboard tests at known poses.

The project should state that the CAHV family is relevant to JPL planetary imaging and PDS calibration products. It should not claim that every JPL vision product requires nonlinear CAHVOR or CAHVORE processing; many products are distributed in linearized CAHV form.

### 7.3 Illumination and radiometry

- Raw linear floating-point output with all display transforms disabled.
- A declared radiometric convention and units.
- At least one analytically known reference scene for validation.
- Documented emitter, reflectance, exposure, clamping, sampling, and color-management settings.
- Fixed samples per pixel for validation renders.
- No denoising for ground-truth products.

Raw linear renderer values must not be described as physical radiance merely because they are stored in EXR. The project must either establish a tested conversion to physical units or label the output as a calibrated relative-radiance representation.

### 7.4 Sensor model

The initial sensor model will be transparent and narrow rather than nominally comprehensive. It will include only effects supported by public parameters or clearly marked assumptions:

- Exposure integration.
- Photon shot noise.
- Read noise.
- Full-well clipping.
- ADC quantization.
- Optional PRNU and dark current when defensible parameters are available.

Photon conversion must account for the relevant optical and sensor quantities, including bandpass or spectral assumptions, aperture/throughput, exposure time, and pixel area. Applying a quantum-efficiency curve directly to an arbitrary RGB value is insufficient.

If complete flight-sensor calibration is unavailable, the output must be described as:

> Imagery using a real flight optical camera model and a representative sensor model.

It must not be described as an exact simulation of the flight camera.

Rolling shutter, motion blur, blooming, Bayer mosaics, cosmic rays, hot pixels, DSNU, and detailed spectral response are follow-on capabilities unless the selected validation target requires them.

**Empirical validation of the sensor implementation.** The sensor model built here is exercised with representative parameters and is not validated at the point of construction. Its implementation is validated separately in Milestone 3 against a physical detector whose characteristics are independently published. See section 9, Milestone 3b.

The scope of that validation must be stated precisely, because it is narrow and easily overstated:

- It establishes that `lowsun.sensor` correctly transforms a radiance field into digital numbers **given** a parameter set.
- It does **not** establish that the parameters assumed for any flight camera are correct.
- It does **not** make synthetic flight-camera imagery an exact simulation of that camera.

The defensible claim is therefore: *the image-formation implementation reproduces the measured noise behaviour of a real detector to within a published tolerance, using parameters measured from that detector.*

### 7.5 Labels

The initial release will produce:

- Depth or range, with the convention identified.
- Surface normals in a documented frame.
- Local slope.
- Shadow mask.
- Camera pose.
- Rock instance segmentation, with identity assigned at scatter time.
- Optional analytical optical flow from depth and pose delta.

A landability or hazard label is in scope for the initial release, contingent on the detail layer passing its statistical validation. Any such label must be tied to an explicit vehicle footprint and stated operational thresholds — at minimum a footprint dimension, a maximum slope evaluated over that footprint, and a maximum protruding rock height within it. Footprint and thresholds are configuration, never constants buried in code, and the values used in any published figure must appear in its caption.

The hazard product must be described as *computed from a statistically representative synthetic surface under stated thresholds*. It is not a hazard assessment of the real site.

### 7.6 Provenance and reproducibility

A scene will be identified by configuration, seed, source-data identifiers, and code/environment version. Each output will include a manifest record with:

- Scene and frame identifiers.
- Configuration hash.
- Random seed.
- Source-data identifiers and checksums.
- Code revision.
- Container or environment version.
- Camera pose and illumination geometry.
- Renderer, version, device, and settings.
- Output paths and checksums.
- Completion or failure status.

The reproducibility promise will be:

> A pinned runtime and render device reproduce identical output where the backend supports it; supported cross-platform environments reproduce outputs within published numerical and image tolerances.

“Byte-identical forever” will not be claimed across renderer versions, GPU models, drivers, operating systems, or floating-point implementations.

Float32 EXR will be used for primary radiometric and geometric ground truth unless testing justifies another representation. Float16 may be offered as an explicitly derived distribution format. The schema must not simultaneously promise 32-bit radiance and configure only 16-bit storage.

## 8. Implementation architecture

```text
lowsun/
  scene/        Schema, configuration, hashing, provenance
  terrain/      Lunar terrain ingest, reprojection, tiling, geometry
  detail/       Procedural rock scatter, regolith roughness, scatter records
  ephem/        SPICE and body-fixed illumination geometry
  brdf/         Baseline and lunar reflectance models
  camera/       CAHV family, ray generation, fitting, resampling
  render/       Backend adapters and radiometric reference scenes
  sensor/       Exposure and calibrated/representative sensor effects
  labels/       Depth, normals, slope, shadow, pose, instances, hazard, optional flow
  eval/         Camera residuals, image statistics, localization metrics
  viz/          Figures, overlays, contact sheets, and demo assets
  manifest/     Provenance records and verification
configs/
data/
docs/
notebooks/
tests/
```

Dependency rules:

- `scene` contains no renderer-specific concepts.
- `detail` emits geometry and a scatter record; labels read the record, never the renderer.
- `render` consumes the scene and camera abstractions.
- Camera geometry is shared by image formation, labels, and evaluation.
- Evaluation does not depend on a particular renderer.
- Any later farm layer remains generic rather than planetary-science-specific.

The renderer choice will be evidence-driven:

- Use Blender/Cycles when it provides sufficient accuracy and useful throughput.
- Use Mitsuba or another verified path when true non-central rays or radiometric reference rendering is required.
- Do not introduce a second renderer before the CAHVORE decision study establishes the need.
- Do not write a custom renderer.

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

### Milestone 1 — CAHVORE study (2–3 weeks)

**Outputs**

- Tested projection and ray-generation implementation, written from primary sources.
- Real-calibration ingest, carrying `T` and `P` through to the model.
- Residual-versus-range figures, reported **per CAHVORE type** rather than aggregated.
- Operationally weighted central fit.
- Navigation-error propagation result.
- Cross-validation against at least one independent implementation.
- Short public technical note.

**Acceptance criteria**

- Round-trip projection tests cover the full field, including corners.
- Results are reported separately for each CAHVORE type present in the calibration set; a conclusion drawn from one type is not generalised to the others.
- Analytical scaling checks agree with numerical results.
- Central-model fitting methodology is reproducible.
- The implementation agrees with at least one independent implementation within a published tolerance, or the divergence is characterised.
- All camera mathematics uses float64. The implementation's own numerical error floor is measured and demonstrated to be well below the decision threshold — prior work on synthetic CAHVOR imagery found that half-pixel rounding of the optical centre, aspect-ratio rounding, and float-precision arithmetic produced sub-pixel errors comparable to the effects under study.
- A rounding audit covers optical centre, image dimensions, and aspect ratio.
- The render-path decision and its justification are recorded.

### Milestone 2 — Lunar vertical slice (7–10 weeks)

**Outputs**

- Tiled terrain ingest for one site.
- Procedural sub-DTM detail layer: seeded rock population from a published size-frequency distribution, plus regolith roughness.
- Declarative scene configuration.
- Finite-source lunar illumination.
- Baseline and lunar reflectance comparison.
- Selected camera render path.
- Narrow sensor model.
- Depth, normals, slope, shadow, pose, and rock-instance labels.
- Hazard/landability label with documented footprint and thresholds.
- Manifest and reproducible runtime.
- One strong still or short descent clip with validation overlays.

**Acceptance criteria**

- A clean pinned environment can regenerate the published example.
- Frame and unit conventions are documented and tested.
- Reference-scene radiometry agrees with the analytical expectation within a published tolerance.
- Camera projection agrees with calibration-based reference calculations within a published tolerance.
- Labels pass synthetic known-geometry tests.
- Cumulative fractional area and size-frequency slope recovered from the generated geometry agree with the input parameters within a published tolerance.
- Rock-instance labels are verified against the scatter-time record, with renderer-derived masks used only as a cross-check.
- Hazard labels are verified on a scene containing planted rocks of known height and position.

### Milestone 3 — Real-versus-synthetic evaluation (4–6 weeks)

**Outputs**

- Matched or approximately matched real and synthetic image subsets.
- Image-statistics comparison.
- Feature detector/matcher or localization benchmark.
- Ablation over at least camera model, reflectance/sensor treatment, and presence or absence of the procedural detail layer.
- Failure-case gallery.
- Versioned small dataset and evaluation harness.

**Acceptance criteria**

- The benchmark runs from a documented command in the pinned environment.
- Metrics are reported with sample counts and uncertainty or distribution plots.
- Real-data limitations are stated explicitly.
- The conclusion identifies which fidelity component mattered, did not matter, or remains unresolved.

### Milestone 3b — Sensor-implementation validation against a measured detector (1 week)

Runs alongside Milestone 3 and shares its evaluation framing. It is scheduled here deliberately: the sensor model exists by this point, the project has momentum, and the work is short, bounded, and independent of the rest of the milestone. It is the only component of the project validated against hardware rather than against public data or analytical expectation.

**Instrument selection**

The reference detector must expose manual exposure, manual gain, and unprocessed frame output with no internal calibration or stacking applied. Consumer smart telescopes — including the Vespera II — do not meet these requirements, as fixed sub-exposures and internal processing make photon-transfer measurement infeasible.

Preference order:

1. A used machine-vision camera (monochrome) from a vendor publishing an EMVA 1288 characterisation report for that model. EMVA 1288 is the industry standard for image-sensor characterisation and covers photon transfer, quantum efficiency, dark current, temporal noise, PRNU, and DSNU. A published report permits a second-order check: the measurement method itself can be compared against the manufacturer's figures.
2. A well-documented development sensor with full manual control and raw Bayer or mono access.
3. A monochrome astronomical camera with full manual control.

Monochrome is preferred so that demosaicing does not have to be modelled or inverted. A flat-field source of documented uniformity is also required; non-uniformity should be characterised rather than assumed.

**Outputs**

- Bias, dark, and flat frame sets across a documented exposure and gain ladder, with temperature logged.
- Measured photon-transfer curve yielding conversion gain, read noise, full-well capacity, and linearity range.
- Measured dark current and, where the data supports it, PRNU.
- Comparison of measured parameters against the manufacturer's published characterisation where one exists, with discrepancies reported rather than reconciled.
- Simulated photon-transfer curve produced by `lowsun.sensor` using the measured parameters, plotted against the measured curve on common axes.
- A short methods note covering measurement protocol, uncertainty, and known deficiencies of the setup.
- Raw frames, analysis notebook, and figures published under `validation/sensor_ptc/`.

**Acceptance criteria**

- The measurement protocol is documented sufficiently for independent repetition.
- Simulated and measured variance-versus-signal relationships agree within a published tolerance, or the disagreement is characterised and its likely cause identified.
- The scope limits in section 7.4 are restated wherever the result is presented.
- Uncertainty is reported; single-point agreement claims are not made.

**Scope control**

This milestone is capped at one week. Detector characterisation is tractable, self-contained, and more immediately rewarding than the surrounding work, which makes it a plausible substitute for harder tasks. If agreement is not achieved within the allotted time, the discrepancy is documented as an open item and the milestone closes. It does not block Milestone 4.

### Milestone 4 — Portfolio release (1 week, overlapping documentation)

**Outputs**

- Public repository with installation instructions and tests.
- Two-minute demonstration video.
- Concise technical report or paper submission.
- Architecture and conventions documentation.
- Reproducible figures.
- Project page summarizing the problem, result, limitations, and next work.

**Acceptance criteria**

- A technically knowledgeable reader can understand the main result without running the code.
- A new user can reproduce the small example without private inputs.
- Known discrepancies and negative results are visible rather than buried.

## 10. Success criteria

### Controllable criteria

The initial project is successful when:

- The CAHVORE study produces a defensible render-path decision.
- One complete lunar example is reproducible from public inputs.
- The procedural detail layer reproduces its input size-frequency statistics within a published tolerance.
- Geometry, radiometry, and labels have quantitative validation tests.
- One downstream real-image benchmark and at least one ablation are published.
- The sensor implementation is compared against a measured detector, with either agreement inside a published tolerance or a characterised discrepancy.
- The repository, dataset sample, report, and demo are publicly accessible.
- At least two relevant engineers or researchers provide technical review.

### Aspirational outcomes

These are valuable but not treated as completion requirements:

- Paper acceptance.
- External contributors.
- An issue or inquiry from a mission organization.
- Adoption by a research or flight project.
- A conference presentation.
- A job interview or offer attributable to the work.

The distinction matters because publication schedules, external engagement, and hiring outcomes are not controlled by the implementation plan.

## 11. Principal risks and mitigations

| Risk | Consequence | Mitigation |
|---|---|---|
| Scope expands into the complete long-term platform | The repository remains unfinished | Freeze the initial release to one body, site, camera, task, and dataset |
| Real imagery lacks usable pose or ground truth | Sim-to-real evaluation becomes subjective | Select and inspect the real evaluation sequence during Milestone 0 |
| Public calibration lacks full sensor parameters | Claims of physical accuracy become indefensible | Separate real optical calibration from a representative sensor model |
| Renderer output is treated as physical radiance without validation | The central scientific claim fails | Use analytical scenes, document units, and publish tolerance results |
| CAHVORE `E` terms are negligible for available cameras | The proposed differentiator disappears | Publish the negative result and pivot to distortion/resampling accuracy |
| Camera selected on `E` alone without regard to type `T` | Study measures a model that behaves centrally; conclusion is wrong | Record `T` and `P` for every candidate in Milestone 0; report all results per type |
| Implementation rounding or float precision dominates the measured effect | The study measures its own bugs rather than camera geometry | Float64 throughout; rounding audit and measured error floor as Milestone 1 acceptance criteria |
| Vendored third-party camera code is incorrect | Oracle validates a shared error | Use as cross-check only, never as the primary implementation; record the fallback in the dependency ledger |
| Cross-platform renders are not byte-identical | Reproducibility criterion fails unnecessarily | Pin a reference runtime and use numerical tolerance tests elsewhere |
| Two-renderer integration consumes the schedule | No downstream result is delivered | Add Mitsuba only after the decision gate demonstrates a need |
| Large dataset generation substitutes scale for evidence | Impressive visuals but weak research value | Keep the initial corpus small enough to validate and ablate thoroughly |
| Input licensing is assumed from public availability | Dataset cannot be distributed as planned | Maintain a product-level rights and attribution ledger |
| Procedural detail layer expands into a terrain-authoring system | Milestone 2 absorbs the schedule | One distribution, one site, no crater or ejecta modelling; statistical acceptance test is the stopping condition |
| Detail layer looks plausible but does not reproduce its input statistics | Labels are unusable as ground truth | Recovery of CFA and size-frequency slope is a required acceptance test, not a visual check |
| Detector characterisation displaces harder milestone work | Tractable measurement substitutes for the core result | Hard one-week cap on Milestone 3b; unresolved discrepancies close as documented open items |
| Sensor-validation result is over-claimed | Implementation validation is read as flight-camera fidelity | Restate the section 7.4 scope limits wherever the result appears |
| Reference detector arrives late or lacks published characterisation | Second-order method check is unavailable | Order during Milestone 0; proceed with measurement-only validation if no report exists |
| Paper ambition delays the usable release | Work remains hidden and unverifiable | Release the technical report and reproducible artifact before review completes |

## 12. Collaboration and ecosystem strategy

NASA JPL's existing LuNaSynth project already generates lunar terrain from DEMs, adds rocks and craters, and renders datasets. LOWSUN should evaluate three possible relationships early:

1. Contribute camera, radiometric, sensor, or validation capabilities upstream.
2. Build an interoperable extension using LuNaSynth terrain outputs.
3. Maintain a separate project where architectural or research requirements genuinely differ.

The decision should follow a documented gap analysis rather than an assumption that no existing tool addresses the problem. Opening a substantive design discussion or making a focused contribution to an existing JPL-led repository could itself produce meaningful hiring signal by demonstrating technical collaboration and upstream engineering.

LOWSUN's public positioning should avoid unsupported priority claims such as “nobody does this” or “the question is unpublished.” Safer and stronger language is:

> Existing public tools address portions of the problem. LOWSUN evaluates a specific combination of non-central camera geometry, documented image formation, reproducible data generation, and downstream validation.

**ROAMS / DARTS Lab.** JPL's ROAMS rover simulator implemented CAHV and CAHVOR image synthesis for real-time closed-loop vision testing, rendering a CAHV image and warping it by a precomputed pixel-correspondence map with bilinear interpolation — the same central-render-and-resample path LOWSUN evaluates in Experiment D, whose interpolation error that work did not quantify. Its verification measured self-consistency between a specified model and one recovered by calibration from the synthesized imagery, with the `E` vector omitted; that methodology cannot detect non-centrality error by construction. CAHVORE was ranked as a high-benefit feature and listed as near-term future work rather than dismissed, and the paper explicitly notes that CAHVOR imagery is often physically unrealistic at image corners without it.

Two consequences:

1. **Check for the follow-up before Milestone 1.** If DARTS Lab subsequently published the CAHVORE implementation and verification, it is primary prior art and LOWSUN extends it rather than opening the question. If no such publication exists, note the absence explicitly rather than implying novelty.
2. **Adopt the honest positioning.** ROAMS assigned fidelity benefit by expert judgment in 2005, targeting classical stereo correlation. An accurate and defensible framing is:

> ROAMS ranked simulation fidelity features by expected benefit to classical stereo vision. LOWSUN measures comparable rankings empirically for learned perception, and quantifies the approximation error in the render-and-resample path that ROAMS used but did not characterise.

Note also that ROAMS ranked realistic texture granularity and sub-centimetre bump mapping as high-benefit features, which supports including the procedural detail layer in the initial release.

## 13. Portfolio presentation

The final portfolio should lead with the measured result, not the module count.

Recommended presentation order:

1. The mission-relevant problem in one paragraph.
2. One image showing real, ideal synthetic, and sensor-modeled synthetic data.
3. The CAHVORE residual figure and the resulting architecture decision.
4. One downstream localization or matching result.
5. An ablation showing which fidelity components mattered.
6. The measured-versus-simulated photon-transfer comparison, with its scope limits stated.
7. A short architecture diagram emphasizing reproducibility and validation.
8. Honest failure cases and remaining uncertainty.

Useful artifacts include:

- A two-minute narrated demo.
- A six-to-eight-page technical report.
- A compact public dataset sample.
- A single-command evaluation run.
- A reproducibility manifest.
- A short engineering retrospective explaining tradeoffs and abandoned approaches.

The strongest interview narrative is not “I built a planetary renderer.” It is:

> I identified an assumption in planetary vision simulation, measured when it breaks, built the smallest validated pipeline needed to test its operational impact, and published the result reproducibly.

## 14. Deferred roadmap

The following capabilities remain strategically valuable after the initial portfolio release:

1. Pushbroom and line-scan camera support with per-line pose and jitter modelling. This is the extension that generalises the work from landers to orbital and Earth-observation imaging, and it is the largest single gap in the initial release.
2. Production job graph, chunking, resume, deduplication, quarantine, and coverage analysis.
3. Crater degradation modelling, ejecta and boulder-field structure, regolith depth variation, and site-specific morphology beyond the single distribution used in the initial release.
4. Permanently shadowed-region illumination, earthshine, long exposure, HDR, and detectability studies.
5. Mars terrain, atmospheric scattering, dust optical depth, and rotorcraft trajectories.
6. Larger sim-to-real studies and synthetic-training ablations.
7. Plume–surface interaction and visual obscuration.
8. Flash lidar or time-of-flight sensor models.
9. Differentiable reflectance inversion.
10. Thermal, multispectral, and event-based sensors.
11. ROS 2, Space ROS, or other distribution/integration layers.
12. Small-body and icy-moon terrain support.

Each deferred milestone should be reconsidered based on user demand, external review, research value, and evidence from the initial release. None is required to claim that the portfolio project is complete.

## 15. References and current relevance

- [NASA — Safe and Precise Landing: Integrated Capabilities Evolution (SPLICE)](https://www.nasa.gov/safe-and-precise-landing-integrated-capabilities-evolution-splice/)
- [NASA — Terrain Relative Navigation impact story](https://www.nasa.gov/directorates/stmd/impact-story-terrain-relative-navigation/)
- [JPL Robotics — Autonomous Landing and Hazard Avoidance Technology](https://robotics.jpl.nasa.gov/what-we-do/research-tasks/alhat-autonomous-landing-and-hazard-avoidance-technology/)
- [NASA PDS — CAHVORE model definition](https://pds.nasa.gov/datastandards/documents/dd/current/PDS4_PDS_DD_1J00/webhelp/all/ch12s03.html)
- [JPL — Mars Exploration Rover Engineering Cameras](https://www-robotics.jpl.nasa.gov/media/documents/maki-jgr-2003je002077.pdf)
- [NASA JPL — LuNaSynth](https://github.com/nasa-jpl/lunasynth)
- Gennery, D. — *Computations for Generalized Camera Model Including Entrance Pupil Movement* (2001)
- Gennery, D. — *Generalized Camera Calibration Including Fish-Eye Lenses* (2002, JPL clearance 03-0869)
- [MER geometric camera model documentation (CAHV/CAHVOR/CAHVORE, including `T` and `P`)](https://pds-geosciences.wustl.edu/mer/urn-nasa-pds-mer_documentation/document/mission/geometric_cm.txt)
- Di, K. and Li, R. — *CAHVOR camera model and its photogrammetric conversion for planetary applications*, JGR Planets 109.E4 (2004)
- Madison, R., Pomerantz, M., Jain, A. — *Camera Response Simulation for Planetary Exploration*, i-SAIRAS 2005 (ROAMS)
- [marsimage — Python CAHVORE/ODL label handling](https://marsimage.readthedocs.io/)
- [bvnayak/CAHVOR_camera_model — photogrammetric ↔ CAHVOR conversion, BSD-2-Clause](https://github.com/bvnayak/CAHVOR_camera_model)
- [NASA — SMD Science Information Policy FAQ](https://science.nasa.gov/researchers/science-information-policy_faq/)
- [Pivotal — Computer Vision Engineer, Autonomy and Perception](https://jobs.lever.co/pivotal/9fe618b9-cf87-4418-9953-31c5e4da56be)

## 16. Final recommendation

Proceed with LOWSUN, but treat the initial project as a focused research-and-engineering artifact rather than a complete planetary synthetic-data platform.

The committed scope is:

> A validated CAHVORE centrality study plus one reproducible lunar imagery vertical slice and one downstream real-image evaluation.

The full image-formation pipeline remains the vision. The narrower release is the project that should be scheduled, implemented, and used to judge completion.
