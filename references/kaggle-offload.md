# Kaggle Compute Offload

Use Kaggle as a remote execution target when local work is likely to OOM, exceed local runtime, need GPU/TPU, need competition-mounted data, or require reproducible training/inference in a Kaggle-like environment.

## When To Offload

Prefer Kaggle notebooks/scripts instead of local execution for:

- Deep learning training, transformer inference, image segmentation, embedding generation, large CV folds, or test-time augmentation.
- Jobs requiring GPU, high-memory GPU, TPU, or Kaggle-mounted competition data.
- Full-fold training after a small local smoke test passes.
- Any run where local RAM/VRAM pressure risks killing the session or slowing the user's machine.

Do a tiny local smoke test first when possible: imports, path checks, one batch, one fold subset, metric function, output file names, and submission schema.

## Kaggle Kernel Flow

1. Ensure Kaggle CLI auth is available. If `kaggle` is missing, install or ask the user to install/authenticate it before remote execution.
2. Create a kernel folder with source code and `kernel-metadata.json`. Use `scripts/prepare_kaggle_kernel.py` for a starter folder.
3. Read `information-sharing-policy.md` before making any Kaggle kernel, dataset, model, or downloaded output public.
4. Set `competition_sources`, `dataset_sources`, `kernel_sources`, or `model_sources` in metadata so Kaggle can mount required inputs.
5. Before GPU scheduling, fetch current allowance, usage, reservations, and reset time using the [GPU quota accounting rule in SKILL.md](../SKILL.md#kaggle-gpu-quota-accounting-avoid-the-sdk-duration-serialization-bug). Calculate from live SDK durations with `.total_seconds()`, report all three inputs and remaining hours, and treat missing or failed quota checks as unverified, not exhausted. Request an accelerator in metadata and/or push command when needed.
6. Push and run the kernel.
7. Poll the kernel status until complete or failed.
8. Download output files.
9. Read `experiment_log.json`, `metrics.jsonl`, and the artifact manifest.
10. If this is the final consumer or final single-kernel run, submit the produced file or kernel/version to Kaggle for scoring.
11. Retrieve the Kaggle submission status/score before deciding the work is complete.
12. If scoring fails with a vague code-competition error, read `code-competition-debugging.md`, patch the final kernel, rerun, and resubmit until scored or concretely blocked.

For complex architectures, do not make one giant notebook responsible for every step. Split the run into a small wave of independent producer kernels, then a final consumer kernel. Producer outputs should be either:

- Published as private Kaggle datasets when they need durability, versioning, reuse across experiments, or clean sharing between multiple consumers. This includes the common pattern where a producer notebook's `/kaggle/working` output is saved/versioned as a dataset, then another notebook consumes that dataset as an attached input.
- Attached directly to the final kernel with `kernel_sources` only when the notebook-output dependency is short-lived and does not need dataset versioning.

Useful commands:

```bash
kaggle kernels push -p kaggle_kernels/EXPERIMENT --accelerator NvidiaTeslaT4
kaggle kernels status USERNAME/KERNEL_SLUG
kaggle kernels output USERNAME/KERNEL_SLUG -p remote_outputs/RUN_ID -o
```

Verify exact command help in the current environment with `kaggle kernels push --help`, because Kaggle CLI behavior can change.

## Required Remote Logs

Every offloaded run should write these files to the Kaggle working directory:

- `experiment_log.json`: one JSON object with run ID, timestamp, config, git commit if known, fold, model, metric summary, hardware, data paths, elapsed time, error state, and artifact paths.
- `metrics.jsonl`: one JSON object per fold/epoch/stage with metric name, value, fold, epoch, step, and timestamp.
- `artifacts_manifest.json`: model files, OOF predictions, test predictions, submission files, plots, and any compressed output archives.
- `stdout.txt` and `stderr.txt` when a wrapper subprocess is used.

For OOM or runtime failures, still emit an `experiment_log.json` with `status: "failed"` and the exception summary if the process can recover.

## Retrieval Discipline

- Never rely only on the browser notebook output. Download artifacts with `kaggle kernels output`.
- Keep downloaded outputs under `remote_outputs/RUN_ID/`.
- Compare remote metrics to local CV before submitting.
- If the run was for inference only, verify row count, ID order, probability clipping, label order, and submission filename after download.
- If the remote run trains fold models, retrieve OOF predictions and per-fold metrics before ensembling.
- If a producer notebook output will be reused, create or version a private dataset from that output and record the dataset handle/version in the pipeline manifest before running consumers.

## Metadata Notes

Kaggle kernel metadata should include:

- `id`: `USERNAME/KERNEL_SLUG`.
- `title`: human-readable run name.
- `code_file`: script or notebook file.
- `language`: `python`.
- `kernel_type`: `script` or `notebook`.
- `is_private`: usually `true` for competition experiments; make public only after a rules, license, and IP check.
- `enable_gpu`: `true` when using GPU.
- `machine_shape`: accelerator ID such as `NvidiaTeslaT4`, if available for the account/competition.
- `competition_sources`, `dataset_sources`, `kernel_sources`, and `model_sources` as needed.

If a competition requires Kaggle Code submission, keep inference code inside the kernel and write the final submission artifact in the expected location.

Do not treat a successful `kaggle kernels push` as the end of the work. The end is the processed Kaggle competition submission and retrieved score/status, or a documented external blocker after the debugging loop has been attempted.

## KaggleHub In Python

Use `kagglehub` inside notebooks/scripts when Python-level resource access is simpler than shelling out:

- Download competition files with `kagglehub.competition_download`.
- Download datasets with `kagglehub.dataset_download` or load files directly with `kagglehub.dataset_load` adapters.
- Download prior notebook outputs with `kagglehub.notebook_output_download`.
- Upload/version intermediate artifact datasets with `kagglehub.dataset_upload`.

Prefer explicit output directories and force flags only when the run is intentionally overwriting cached or downloaded artifacts.
