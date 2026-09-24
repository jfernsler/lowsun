# LOWSUN M0 — A Research Note for Getting Started

This is a note from your future, slightly more organized research self.

Milestone 0 is not asking you to prove a new theory or build a simulator. It
asks you to make a small set of choices well enough that later work has a
clear target. The output is a defensible trail of evidence and decisions, not
a large volume of reading or code.

Use this alongside [M0.md](M0.md). That file is the sequence of work; this one
is guidance on how to approach it.

## First: what "doing research" means here

For LOWSUN, research means repeatedly doing four simple things:

1. State the question precisely enough that evidence could change your mind.
2. Find the strongest practical source for the answer.
3. Record what the source actually says, where it says it, and what you infer
   from it.
4. Make the smallest decision that the evidence supports; record what remains
   unknown.

That is it. You do not need to sound academic, read every paper cited by a
paper, or hide uncertainty. In fact, explicit uncertainty is one of the signs
that the work is trustworthy.

The central M0 question is deliberately narrow:

> Can a real public lunar dataset support a meaningful, reproducible test of
> when a central camera approximation is adequate, and when it is not?

Everything you collect should help answer that question or make the eventual
test reproducible.

## Your most important habit: separate evidence from interpretation

When taking notes, use these three labels every time:

- **Fact:** directly stated or measured in a source. Include the exact page,
  table, section, product ID, or label field.
- **Interpretation:** your conclusion from those facts. Say why you reached it.
- **Open question:** something material that is not yet established, plus the
  next action needed to resolve it.

For example:

```text
Fact: Product label XYZ records CAHVORE T = 3, P = 0.7, and includes C, A, H,
V, O, R, and E. (product label, GEOMETRIC_CAMERA_MODEL group, accessed YYYY-MM-DD)

Interpretation: This is a valid candidate for the non-centrality study because
the metadata needed to determine its model behavior is available. Its actual
effect size remains unmeasured.

Open question: Is the calibration associated with a real image sequence whose
pose and terrain overlap are adequate for feature-matching evaluation? Inspect
three representative products and their ancillary metadata.
```

This simple distinction prevents a common early-research failure: a plausible
idea slowly becoming an assumed fact after it is copied between notes.

## A good M0 note is auditable, not polished

Think of every note as something you should be able to revisit in six months
and still understand. Someone else should be able to find the same source and
see why you made the decision.

For any important claim, capture:

- A stable URL, DOI, PDS product identifier, or archive identifier.
- The title, authoring organization, and publication/product date when known.
- Your access date.
- The useful location inside it: page, section, figure, table, or label field.
- A short paraphrase in your own words.
- Whether you are quoting a source, making an inference, or adopting an
  engineering assumption.

Avoid copying long passages into project notes. A short paraphrase with a
precise pointer is easier to use and makes your own thinking visible.

## Start with the source hierarchy

Not all references should carry the same weight. For M0, use this rough order
of preference:

1. **Primary technical sources:** the Gennery camera-model papers, PDS labels,
   instrument/product documentation, mission archive metadata, and official
   data-use terms.
2. **Official institutional documentation:** JPL, NASA, PDS, USGS, or mission
   instrument pages that explain a product or archive.
3. **Peer-reviewed papers:** especially for prior art, terrain statistics, and
   experimental methodology.
4. **Maintained implementation documentation:** helpful for practical file
   reading and library evaluation, but not authority for the camera math.
5. **Repositories, blog posts, and search snippets:** leads to investigate,
   not evidence by themselves.

This does not mean lower levels are useless. It means you should not make a
claim such as "this camera is non-central" based only on a GitHub parser or a
search-result summary when the calibration label and the primary model paper
can settle the question.

## How to read papers without getting buried

You almost never need to read a technical paper front to back on first pass.
Use three passes.

### Pass 1: orient — 10 to 15 minutes

Read the abstract, introduction, conclusion, headings, figures, tables, and
references. Write down:

- What question did this work answer?
- What model or data did it use?
- Which claim is relevant to LOWSUN?
- Is it a primary source, prior art, or an implementation aid?

If it is not clearly relevant after this pass, stop. Put it in a "possibly
useful later" list rather than forcing it into M0.

### Pass 2: extract — 20 to 45 minutes

Read only the sections needed for your M0 question. Take page-linked notes for
definitions, equations, data assumptions, numerical caveats, and validation
methods.

For the CAHVORE papers, focus on the model definitions and the projection/ray
generation procedure. You do not need to understand every calibration-solver
detail before M1.

### Pass 3: apply — 10 minutes

Write one sentence beginning with one of these:

- "This changes LOWSUN because…"
- "This supports the decision to…"
- "This is not yet applicable because…"
- "This creates an open question about…"

If you cannot write one of those sentences, the paper may be interesting but
does not yet belong in the active M0 evidence set.

## The CAHVORE trap to avoid

The most important technical caution in M0 is simple:

> `E` is not a stand-alone certificate of non-centrality.

For every candidate calibration, record all of `C`, `A`, `H`, `V`, `O`, `R`,
`E`, **and** CAHVORE type `T` and linearity parameter `P`. Then use the
primary model source to state what that model type means.

Do not write any of these statements until M1 has measured them:

- "This camera has a large non-central effect."
- "A central model will fail."
- "The renderer must support per-pixel ray origins."

M0 can say only that a calibration is a credible candidate to test, or that it
lacks the metadata needed for such a test. That restraint makes the later
result much stronger.

## Selecting data is a compatibility problem

Do not select the "best" terrain product, camera, or image sequence in
isolation. You are selecting a compatible trio:

```text
terrain site + camera calibration + real evaluation sequence
                 ↓
       one measurable downstream task
```

A beautiful terrain product is not useful if it does not overlap the imagery.
A sophisticated camera calibration is not useful if it lacks real images or
the needed pose context. A real image sequence is not useful for an objective
benchmark if the available metadata cannot make the comparison plausible.

Use a small candidate table and score only the factors that matter now:

| Criterion | Question |
| --- | --- |
| Complete calibration | Are `C A H V O R E T P`, dimensions, frames, and units available? |
| Candidate behavior | Does the type-specific model justify measuring non-centrality? |
| Terrain compatibility | Can the site and imagery plausibly be placed in compatible frames and coverage? |
| Evaluation feasibility | Are image provenance, pose/calibration context, and a measurable task available? |
| Reproducibility | Are products stable, public, citable, and usable under known terms? |
| Scope cost | Can this be understood and tested by one person in the M1–M3 time budget? |

There is no need to pretend the selection is perfectly objective. Explain the
trade-off. "We chose candidate B because it has a slightly less ideal `E`
profile but complete labels and a practical real sequence" is a good research
decision.

## Keep a rights ledger from the first download

Publicly reachable does not automatically mean redistributable. Before
downloading a product for project use, record:

- What it is and where it came from.
- Its stable identifier and access date.
- The archive's usage terms or licence.
- Required attribution.
- Whether you may redistribute the original, derived data, thumbnails, and
  calibration labels.
- Whether the project will store it, link to it, or require users to download
  it themselves.

When terms are unclear, write "unclear; link only until confirmed." That is a
valid interim decision. It is better than guessing and much easier to repair
in M0 than before a public release.

## A decision threshold is a promise to your future self

The central-approximation threshold must be recorded before viewing M1's final
results. This prevents a subtle temptation: changing "good enough" after
seeing an inconvenient plot.

At M0, a provisional threshold is acceptable. Document:

- The value and units, for example `0.05 px`.
- Which residual or error statistic it applies to.
- The intended field-angle and range envelope.
- Why it is plausible for the intended feature detector or matcher.
- What evidence would justify changing it before final analysis.

The final decision should be operational rather than aesthetic: use the
central render-and-resample path where it stays below the threshold; use a
true non-central path only where the measured evidence requires it.

## Use timeboxes to protect the project

Research can expand forever because every source contains another interesting
reference. M0 is one week because it needs enough evidence to make a decision,
not exhaustive literature coverage.

Useful timeboxes:

- 15 minutes to decide whether a newly found paper deserves a first pass.
- 45 minutes maximum for first-pass reading of a paper before writing an
  applicability note.
- One focused session per camera candidate before comparing candidates.
- One day for deciding the compatible data trio and benchmark.
- One final session to resolve only the questions that block M1.

When a question has no practical answer after its timebox, record it as an
open risk and choose the conservative path. For example, exclude a candidate
whose terms or frame metadata cannot be resolved quickly. Do not let one
ambiguous archive consume the milestone.

## A simple working rhythm

At the beginning of each work session, write three lines in the decision log:

```text
Today's question:
Evidence I expect to inspect:
Decision or next action I want by the end of the session:
```

At the end, write:

```text
What I learned:
Decision made (or why none is justified yet):
Open question and its next action:
```

This is particularly useful when returning after several days. You will not
need to reconstruct your mental state from browser tabs and partial notes.

## When you are allowed to move on

You do not need perfect certainty to finish M0. You need enough confidence to
start M1 without discovering that its data, metadata, or benchmark is
impossible.

M0 is ready to close when you can point to:

- A selected site and terrain product with understood coordinate conventions,
  format, citation, and terms.
- Two or three evaluated camera candidates with complete `T`/`P` treatment.
- At least one defensible selected calibration, or an explicit negative result.
- A real sequence that you have actually inspected, not merely bookmarked.
- One stated downstream task, metric, range/field-angle envelope, and
  pre-results threshold.
- A rights ledger and a dependency record with sources and fallbacks.

The sentence you should be able to say at the end is:

> I know what I will test, why this data can test it, what would count as a
> meaningful outcome, and what I still do not know.

That is a successful research milestone. The remaining uncertainty is not a
failure; it is the reason Milestone 1 exists.

## Final reassurance

Your job is not to prove that CAHVORE matters. Your job is to create a fair
test that can report either result. A finding that the central approximation
is adequate over your selected envelope is valuable, provided the test is
clear and reproducible.

Keep the scope small, write down the evidence trail, and let the measured
result—not the original project pitch—decide what LOWSUN needs to build next.
