---
name: agentic-kaggle-skill
description: Kaggle-first end-to-end competition workflow for scored submissions. Use when Codex must run Kaggle or competitive ML workflows through scored submission, including code competitions, validation, metrics, policy-safe public notebook/discussion intel, tabular/text/image modeling, tuning, ensembling/stacking, proactive multi-notebook architectures, producer notebooks that train models and export private Kaggle artifact datasets, downstream consumer notebooks, Kaggle GPU offload, kagglehub access, hidden-test debugging, and public score retrieval.
---

# Agentic Kaggle Skill

## Operating Loop

Treat every competition as a validation problem first and a modeling problem second. The default target platform is Kaggle, so prefer Kaggle-native notebooks/scripts, datasets, model artifacts, competition submissions, and score receipts. For code competitions, assume the final notebook/kernel will be rerun by Kaggle against hidden data unless the competition docs prove otherwise.

1. Read the competition page, classify the submission mode as classic file submission or code/notebook scoring, then inspect the rules, data-use terms, sharing policy, data dictionary, metric, submission format, train/test construction hints, and leakage warnings.
2. If live competition intelligence tools are available, inspect top public open notebook solutions and relevant discussion activity before major architecture choices; treat them as clues, not authority.
3. Identify the task type: binary, multiclass, multilabel, regression, ranking, image, segmentation, text, time series, grouped entities, or a hybrid.
4. Design folds before feature engineering or modeling. Prefer a fold column saved into the training data so every experiment uses the same comparison surface.
5. Build the simplest metric-correct baseline and produce out-of-fold (OOF) predictions plus a valid submission.
6. Proactively plan a stronger architecture once the baseline is trustworthy: diverse model families, feature/embedding producers, augmentation/pseudo-label/distillation stages, calibration/postprocessing, and an ensemble or stacker.
7. Iterate with validation gates: test one meaningful change at a time when possible, but launch several independent producer notebooks in parallel when they create diverse artifacts that can be compared by OOF score or ensemble diversity.
8. Offload heavy training, inference, embedding generation, image/text experiments, or memory-risky jobs to Kaggle notebooks/scripts when local compute may OOM or take too long. Before GPU scheduling or availability claims, fetch fresh quota and apply the GPU quota accounting rule below.
9. For sophisticated architectures, split work into a small Kaggle pipeline: run several independent producer notebooks/scripts first, save each useful producer output as a private Kaggle dataset, then run one consumer notebook/script that attaches those datasets and creates the final OOF/test/submission outputs. When a producer trains a model, its checkpoint, tokenizer/config, fold metadata, OOF/test predictions, and manifest should be exported as a Kaggle dataset; downstream notebooks load the model from `/kaggle/input/...`.
10. Submit the final submission-producing artifact to Kaggle for scoring and retrieve the resulting submission status/score.
11. If Kaggle returns a code-competition error or vague scoring failure, enter the debugging loop: retrieve available logs, classify likely failure mode, patch defensively, rerun the final kernel, resubmit, and repeat until scored or concretely blocked.
12. Track local CV, remote Kaggle run status, Kaggle submission score, public LB, private-risk notes, seed, code version, data version, and artifact paths for every run.
13. Ensemble only with OOF predictions generated without in-fold leakage.

## Resource Map

- Read `references/method-map.md` for the neutral workflow map behind the skill.
- Read `references/information-sharing-policy.md` before publishing competition code, notebooks, datasets, models, artifacts, or reports outside the active team or workspace.
- Read `references/competition-intel.md` when a Kaggle competition slug, URL, title, public leaderboard context, open solutions, or discussion activity may inform the approach.
- Read `references/cross-validation-and-metrics.md` when choosing folds, metrics, thresholds, or leakage checks.
- Read `references/tabular-workflow.md` for categorical variables, feature engineering, selection, and hyperparameter tuning.
- Read `references/image-text-workflow.md` for image, segmentation, and NLP competition approaches.
- Read `references/kaggle-code-competition-pipeline.md` when the target is a Kaggle code competition, hidden rerun, final notebook scoring flow, or model artifact handoff between producer and consumer notebooks.
- Read `references/advanced-notebook-architecture.md` when a stronger solution may need multiple model families, staged feature/embedding/model producers, pseudo-labeling, distillation, postprocessing, blending, stacking, or parallel Kaggle GPU notebooks.
- Read `references/kaggle-offload.md` when local runs are heavy, GPU is useful, Kaggle data access is needed, or remote notebook results must be collected.
- Read `references/kaggle-pipeline-datasets.md` when using multiple Kaggle notebooks, intermediate datasets, notebook output sources, or `kagglehub`.
- Read `references/submission-endgame.md` before stopping work; this skill is not done until Kaggle scoring has been attempted and the result has been collected or a concrete blocker is documented.
- Read `references/code-competition-debugging.md` when a Kaggle code competition submission fails, times out, OOMs, produces no score, or reports a vague hidden-run/scoring error.
- Read `references/ensembling-and-reproducibility.md` for project layout, OOF artifacts, stacking, blending, and repeatability.
- Read `references/research/` or `examples/` only when the user asks for historical case studies, Hermes-era patterns, or concrete competition lessons from this repository.
- Run `scripts/scaffold_competition.py` to create a competition workspace.
- Run `scripts/make_folds.py` to add a fold column to a training CSV.
- Run `scripts/prepare_kaggle_kernel.py` to create a Kaggle kernel folder with metadata and retrievable experiment logs.
- Run `scripts/prepare_kaggle_dataset.py` to create metadata and commands for versioned intermediate artifact datasets.

## Default Kaggle Procedure

Start with the metric and validation:

- Reproduce the competition metric locally, including clipping, transforms, averaging mode, thresholds, and sample weights.
- Check whether the competition allows public code, external data, pretrained weights, generated artifacts, and public repositories before sharing anything outside the team.
- Classify the Kaggle submission path early. If it is a code competition or notebook-scored workflow, design the final consumer notebook as the scoring artifact from the start.
- Use `kaggle-competition-intel` MCP tools when available: `top_open_solutions` for score-ranked plus latest high-vote notebooks, and `competition_discussions` for top-voted plus recently active discussion topics before choosing expensive baselines.
- Choose folds to match the hidden test distribution. Use stratification for classification, binned stratification for regression, group splits for repeated entities, time-aware splits for temporal data, and multilabel-aware splits when label co-occurrence matters.
- Save OOF predictions for every model. OOF files are the currency for error analysis, threshold tuning, blending, and stacking.
- Compare mean fold score and fold variance, not only the best fold.
- Treat a public LB jump that disagrees with CV as a validation question before treating it as a modeling victory.

Then build baselines in this order:

1. Metric-only or constant baseline to verify scoring and submission format.
2. Fast classical baseline: LightGBM/CatBoost/XGBoost for tabular, TF-IDF plus linear model for text, pretrained backbone for images.
3. Strong baseline with stable folds, clean artifacts, and OOF predictions.
4. Feature/model experiments with run logs and one primary metric.
5. Ensembling or stacking after several diverse, individually validated models exist.

Do not wait for the user to explicitly ask for sophistication when the competition warrants it. After a stable baseline, propose and execute a stronger staged architecture if public solutions, discussion intel, data modality, metric pressure, or CV plateau suggests that a single notebook/model will underperform. Keep the architecture gated by OOF evidence, artifact manifests, and final Kaggle scoring.

When a run is likely to exceed local RAM/VRAM, require a GPU/TPU, or needs Kaggle-only data mounts, prepare a Kaggle kernel run instead of forcing it locally. The remote run must emit `experiment_log.json`, `metrics.jsonl`, and an artifact manifest so results can be retrieved and interpreted after `kaggle kernels output`.

When the solution has multiple heavy stages, keep the remote graph shallow and inspectable:

- Use fewer than five independent producer notebooks/scripts per wave.
- Give each producer a single responsibility such as feature generation, embedding extraction, fold training, checkpoint training, model distillation, or inference.
- For code competitions, prefer producer notebooks that train models/checkpoints and export them as private Kaggle datasets; the final consumer should attach those datasets, load models from `/kaggle/input/...`, and perform hidden-test-safe inference.
- Save producer notebook outputs as versioned private Kaggle datasets for durable reuse and as stable inputs to downstream notebooks; use direct `kernel_sources` only for short-lived chains where durability/versioning is unnecessary.
- Run one final consumer notebook/script that attaches the produced datasets, reads their artifacts, writes the final OOF, test predictions, blend/stack report, and submission, then submit that final artifact/kernel to Kaggle for scoring.
- Use `kagglehub` inside Python when it is more convenient to download datasets, competition files, notebook outputs, or upload dataset versions programmatically.

## Kaggle GPU quota accounting: avoid the SDK duration serialization bug

Never calculate GPU availability using `str(response)`, `json.loads(str(response))`, serialized SDK duration strings, or `timedelta.seconds`.

Kaggle SDK serialization can drop whole days: 30 hours becomes 6 hours. This previously caused an incorrect claim that GPU quota was exhausted.

Read the live SDK duration objects and use `.total_seconds()` for each field:

```python
allowed_seconds = allowed.total_seconds()
used_seconds = used.total_seconds()
reserved_seconds = reserved.total_seconds()
remaining_seconds = allowed_seconds - used_seconds - reserved_seconds
remaining_hours = remaining_seconds / 3600
```

Fetch the current allowance, usage, reservations, and reset time each time. Do not hard-code the weekly allowance. Report all three inputs alongside the calculated remaining hours and reset time.

If SDK values conflict with the Kaggle UI or look inconsistent, verify using authenticated:

```text
GET https://www.kaggle.com/api/v1/kernels/quota
```

Alternatively:

```text
GET https://api.kaggle.com/v1/kernels/quota
```

For raw duration objects, convert with:

```python
seconds = duration["seconds"] + duration.get("nanos", 0) / 1e9
```

Preserve whole days and fractional seconds throughout the calculation. Missing fields or request failures mean quota is unverified, not zero.

Regression check: an allowance of 30 hours, usage of 7.5 hours, and reservations of 2 hours must yield 20.5 hours remaining. Never declare quota exhausted based on lossy SDK text.

## Definition Of Done

Do not stop at a plan, scaffold, trained model, notebook run, downloaded output, or local validation score. Continue until one of these is true:

- A competition submission has been sent to Kaggle, Kaggle has processed it, and the public score or submission status has been retrieved and recorded.
- A code-competition final notebook/kernel has consumed all required producer artifacts, has been submitted with its kernel/version reference, and the resulting submission status/score has been retrieved and recorded.
- A concrete external blocker prevents scoring: missing Kaggle credentials, rules not accepted, quota exhausted after retry attempts, competition closed, scoring disabled, required manual UI-only action, unavailable kernel version, or Kaggle service failure. Record the exact blocker and the next command/action needed.

GPU quota exhaustion is a blocker only after verification using the GPU quota accounting rule above. Unverified quota must not be reported as exhausted.

## Validation Choices

Use this quick mapping:

- Balanced classification: `StratifiedKFold`.
- Imbalanced classification: stratified folds plus metric-sensitive thresholding or class weights.
- Regression: `KFold` for ordinary targets, binned stratified folds for skewed or multimodal targets.
- Same user, patient, product, session, document, or image source appears multiple times: `GroupKFold` or stratified group splitting.
- Ordered events or forecasting: time split, expanding-window validation, or competition-specific period split.
- Images from same subject or scene: group by subject/scene before augmentation.
- Text with duplicated sources, authors, questions, products, or prompts: group by source key.
- Tiny data: repeated folds or more folds, but keep the final model selection discipline strict.

## Modeling Priorities

- For tabular competitions, prefer tree boosting first unless the data is mostly sparse text or high-cardinality categorical interactions. Add CatBoost when categorical columns are central.
- For linear models and neural networks, scale numeric features and encode categoricals carefully.
- For tree models, start with label/count/frequency encodings and sensible missing values; scaling is usually unnecessary.
- For target encoding, always compute encodings out-of-fold, with smoothing and no validation-row target visibility.
- For image tasks, get dataset, transforms, labels, masks, and metric correct before changing architecture.
- For text tasks, keep a TF-IDF baseline even when using transformers; it is fast, interpretable, and often blends well.
- For hyperparameter search, tune only after folds and baselines are trustworthy. Search learning rate, depth/leaves, regularization, sampling, and number of estimators with early stopping.

## Competition Hygiene

- Keep raw data read-only.
- Keep competition data, private datasets, model checkpoints, downloaded outputs, submissions, credentials, and data-derived artifacts out of public Git repositories unless the competition rules and artifact licenses explicitly allow public redistribution.
- Put all experiment-changing values in config or command-line args.
- Seed Python, NumPy, framework, and model libraries when possible.
- Save model files, OOF predictions, test predictions, fold scores, feature lists, and submission files with run IDs.
- For Kaggle-offloaded jobs, write retrievable logs and artifacts into the notebook working directory, including run ID, git commit, config, fold, metric, hardware, elapsed time, OOM notes, and output file paths.
- For multi-notebook pipelines, maintain a pipeline manifest with producer kernel refs, produced dataset refs, dataset versions, artifact schemas, metrics, and the final consumer run ID.
- Keep producer datasets, kernels, and model artifacts private by default. Make them public only after verifying the target competition rules, data license, third-party IP, and user intent.
- Record submission message, submission command or kernel/version, Kaggle submission timestamp, status, public score if available, and any error returned by Kaggle.
- For vague code-competition failures, record the observed status, available logs, suspected failure class, patch attempted, and next retry command.
- Never train a stacker on in-sample base predictions.
- Never select features, target encoders, scalers, thresholds, or augmentations using validation targets outside the fold boundary.
- Prefer scripts for repeatable training and notebooks for exploration and plots.

## Useful Commands

Create a competition skeleton:

```bash
python3 <skill-dir>/scripts/scaffold_competition.py --root .
```

Create stratified folds:

```bash
python3 <skill-dir>/scripts/make_folds.py \
  --input input/train.csv \
  --output input/train_folds.csv \
  --target target \
  --strategy stratified \
  --n-splits 5
```

Create grouped folds:

```bash
python3 <skill-dir>/scripts/make_folds.py \
  --input input/train.csv \
  --output input/train_folds.csv \
  --target target \
  --group-col patient_id \
  --strategy group
```

Prepare a Kaggle GPU kernel experiment folder:

```bash
python3 <skill-dir>/scripts/prepare_kaggle_kernel.py \
  --output kaggle_kernels/exp_lgbm_gpu \
  --username kaggle-user \
  --slug exp-lgbm-gpu \
  --title "exp lgbm gpu" \
  --competition playground-series-sample \
  --accelerator NvidiaTeslaT4
```

Prepare a private Kaggle dataset folder for intermediate artifacts:

```bash
python3 <skill-dir>/scripts/prepare_kaggle_dataset.py \
  --output kaggle_datasets/exp_features_v1 \
  --username kaggle-user \
  --slug exp-features-v1 \
  --title "exp features v1" \
  --description "OOF-safe feature artifacts for experiment exp_features_v1"
```

Prepare a private Kaggle dataset folder for trained model artifacts:

```bash
python3 <skill-dir>/scripts/prepare_kaggle_dataset.py \
  --output kaggle_datasets/exp_model_v1 \
  --username kaggle-user \
  --slug exp-model-v1 \
  --title "exp model v1" \
  --description "Trained model artifacts for experiment exp_model_v1" \
  --artifact-kind model
```
