# AI Content Detection Team Plan

## Purpose

Build an image screening application backed by reproducible research. Version one assesses evidence of full AI generation in photographic images, including unfamiliar generators and images affected by compression or resizing.

Start from zero verified results. Do not reuse accuracy figures, dataset descriptions, or implementation claims from earlier academic documents as experimental evidence.

Support three outcomes: **Likely AI-generated**, **Likely camera-origin**, and **Inconclusive**. Camera origin does not establish whether a depicted event or accompanying story is truthful. Do not present an uncalibrated model score as a verified probability.

Partial AI edits, face swaps, illustrations, screenshots, text, audio, and video are outside the initial supported scope. Evaluate them separately before expanding claims.

## Shared ownership

Every member contributes to data, models, evaluation, application development, and documentation. Leads coordinate decisions and delivery; partners share implementation and understanding. Rotate partners after milestones and record changes in issues.

| Workstream | Lead | Initial partner | Deliverable |
| --- | --- | --- | --- |
| Baseline detector and inference | Gopal Singh | Saubhagya Shubham | Reproduce a released detector, preserve preprocessing, export predictions |
| Dataset quality and splits | Kumar Shivam | Harsh Prabhakar | Traceable manifest, duplicate audit, reproducible generator-aware splits |
| Model improvement | Saubhagya Shubham | Gopal Singh | Frozen-encoder classifier, controlled comparisons, justified fine-tuning experiments |
| Robustness and generalisation | Harsh Prabhakar | Kumar Shivam | Unseen-generator and external-source evaluation; JPEG, resize, and blur tests |
| Calibration and inconclusive decisions | Saubhagya Shubham | Harsh Prabhakar | Confidence evaluation and measured error-versus-coverage trade-offs |
| Application and integration | Kumar Shivam | Everyone | Shared inference connected to FastAPI and an upload interface |

Saubhagya leads model improvement and decision quality while other members lead equally necessary research and engineering work. Gopal and Kumar independently evaluate model improvements before acceptance. Have someone outside the implementation pair review major research conclusions.

At each milestone, record who implemented, reviewed, evaluated, and documented the work. By completion, every member should have a reviewed contribution in data, ML, evaluation, application, and reproducibility.

## Research question and experiments

Under a fixed compute budget, how much do balanced image processing and compression-aware training improve unseen-generator detection, and does abstention reduce errors on difficult images?

| Experiment | Change |
| --- | --- |
| A | Frozen pretrained visual encoder with a small classifier |
| B | A plus matched compression and resolution treatment between classes |
| C | B plus compression and resize augmentation during training |
| D | C plus validation-fitted calibration and inconclusive decisions |

Compare against at least one released detector on the same evaluation images. Keep splits fixed and repeat the main training comparisons with multiple random seeds. Record variability. Change the sequence through documented decisions supported by findings.

This is a research hypothesis, not a promise of improvement or publication novelty. A publication would require a deeper comparison with prior work.

## First team session

Agree together on scope, dataset sources and usage terms, GPU access, review partners, and an actual deadline. Define `camera_origin = 0` and `ai_generated = 1` consistently.

Create four first-week GitHub issues. Each needs an owner, partner, acceptance criteria, and dependencies. The eight-week schedule below is provisional.

## First week

Start with approximately 1,000 traceable images with agreed class balance and generator coverage. Use this pilot for development, not as the final untouched test set.

| Member | Task | Acceptance criteria |
| --- | --- | --- |
| Saubhagya | Build the frozen-encoder classifier experiment and comparison configuration | Rerunnable training configuration; predictions follow the common contract |
| Gopal | Run one released detector | Record checkpoint source/version, preprocessing, score direction, environment, and pilot predictions |
| Kumar | Build manifest, duplicate checks, and split generation | Every image has a source and label; duplicate groups remain in one split; splits are reproducible |
| Harsh | Build evaluation and transformation scripts | Per-generator and clean/JPEG/resize results; metric and label handling checked on known examples |

- Day 1: agree interfaces, audit hardware, select data and baseline, create issues.
- Days 2-3: prepare data and implement inference, training, and evaluation against shared interfaces.
- Days 4-5: compare both approaches on common evaluation images and inspect failures together.
- Days 6-7: fix issues, review results, document reproduction, choose the next experiment.

If training is blocked, complete the released-detector benchmark and record the blocker. Do not invent results or rush into larger downloads.

## Shared interfaces

Use a versioned CSV dataset manifest containing:

`image_id`, `relative_path`, `label`, `source`, `generator`, `width`, `height`, `format`, `sha256`, `duplicate_group`, `split`, `license_reference`.

Paths are relative to a configurable local data root. Unknown generators are explicitly marked `unknown`. A duplicate group includes near-duplicates and derivatives of one original. Record original properties and preserve originals; record transformations separately.

Use one inference function shared by scripts and the API. Its result includes:

`image_id`, `ai_score`, `assessment`, `model_version`, `processing_time_ms`, `warnings`.

Document score scale/direction and timing boundaries. Save raw scores and version the thresholds and calibration separately. Frontend/backend work may use clearly marked mock responses until inference is ready.

## Data and evaluation rules

1. Audit corrupt files, class balance, content differences, resolution, and compression before training.
2. Group duplicates and derivatives before splitting. Keep transformed versions with their original.
3. Use training data for weights and validation data for hyperparameters, calibration, thresholds, and abstention policy.
4. Maintain familiar-generator, held-out-generator, and external-source evaluations. Keep final tests untouched during iteration.
5. Record released detectors' documented training sources. Unseen by our training does not mean unseen by a downloaded model.
6. Apply experimental transformations equally to both classes. Converting JPEG to PNG does not remove earlier compression history.
7. Report balanced accuracy, ROC-AUC, confusion matrices, and synthetic recall at a validation-selected low-false-positive operating point. Report actual test false-positive rates too.
8. Break results down by generator and transformation. Include uncertainty estimates for main comparisons where sample size permits.
9. Report both coverage (fraction receiving a definite decision) and error rate on decided cases. Evaluate calibration on familiar and unfamiliar data separately.
10. Measure latency and memory on actual hardware. Separate inference timing from end-to-end timing.
11. Inspect failures and record unsupported categories. Do not select conclusions solely because aggregate accuracy looks impressive.

## Hardware and budget

Reported resources: RTX 5060 Ti OC with 16 GB VRAM, 16 GB 6000 MHz system RAM, and 500 GB storage capacity. Free disk space and driver/PyTorch compatibility must be verified.

Start locally with zero cloud spending. Use small batches, incremental data loading, and cached frozen-encoder embeddings where useful. After the pilot, consider 10,000-20,000 images as an initial experiment scale. A 30-60 GB allocation is only a planning estimate; check actual download and extraction sizes first. Avoid full benchmark downloads by default.

Initially schedule one heavy GPU job at a time through a shared issue or agreed calendar. Share prediction exports and small permitted samples so evaluation does not require GPU access. Do not expose a development API publicly to share the GPU. Agree on spending only after identifying a measured bottleneck.

## Repository and collaboration

Grow this structure as implementation begins:

```text
src/
  data/
  inference/
  training/
  evaluation/
backend/
frontend/
configs/
tests/
reports/
docs/
```

- Maintain one data pipeline, evaluation pipeline, and application; plug model variants into shared interfaces.
- Use focused branches and pull requests reviewed by another member. Configure branch protection when permissions allow.
- Use a board with To do, In progress, Review, and Done. Hold short check-ins and a weekly working demonstration.
- Record experiment ID, code commit, environment, seed, configuration, manifest/split version, checkpoint identity, metrics, and interpretation.
- Commit small reports, configurations, and appropriate manifests. Exclude datasets, weights, uploads, embeddings, secrets, and environments. Check reports for private paths and images before sharing.
- Add focused checks for split isolation, score direction, metric correctness, preprocessing, and upload validation as components are built.
- Update README setup instructions only after verification. Have a teammate reproduce a run from those instructions.

## Provisional milestones

| Period | Required outcome |
| --- | --- |
| Week 1 | Pilot manifest, released detector, first classifier comparison or documented blocker |
| Week 2 | Reproducible baseline report and thin upload-to-result prototype |
| Weeks 3-4 | Controlled model improvement and held-out-generator evaluation |
| Weeks 5-6 | Robustness, calibration, and abstention experiments with independent review |
| Weeks 7-8 | Usable application, untouched final evaluation, reproduction check, report |

Judge progress by evidence and reproducibility, not promised accuracy. A member who did not implement a component should reproduce its result and explain its limitations before completion.

## Research starting points

Candidate resources require review of terms, training sources, and implementation requirements before use:

- [UniversalFakeDetect official implementation](https://github.com/WisconsinAIVision/UniversalFakeDetect)
- [Community Forensics implementation and data links](https://github.com/JeongsooP/Community-Forensics)
- [GenImage official repository](https://github.com/GenImage-Dataset/GenImage)
- [Fake or JPEG? Dataset bias study](https://arxiv.org/abs/2403.17608)
- [CO-SPY semantic and pixel features](https://arxiv.org/abs/2503.18286)
- [AIGIBench NeurIPS 2025 benchmark](https://papers.neurips.cc/paper_files/paper/2025/hash/fb693c67f61e5321746ffce8b6fdd2d0-Abstract-Datasets_and_Benchmarks_Track.html)
- [On Calibration of Modern Neural Networks](https://proceedings.mlr.press/v70/guo17a.html)
- [DailyBench 2026 preprint](https://arxiv.org/abs/2607.24016)

## Immediate next action

Meet as a team, confirm assignments and deadline, and open the four first-week issues. Verify the environment and pilot data before larger downloads or training runs.
