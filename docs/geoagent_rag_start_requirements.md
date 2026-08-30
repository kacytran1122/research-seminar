# GeoAgent-RAG Start Requirements

Target venue: **IGARSS**

Project title:

> **GeoAgent-RAG: Agentic Retrieval-Augmented Planning for Reproducible Remote-Sensing Experiments**

This document lists what Codex, Claude Code, and the research engineer need from the user/team before starting the GeoAgent-RAG research implementation.

## Short Answer

Yes, the AI coding agents and engineer need a few inputs from us before starting. The most important are:

1. data location,
2. indexing scope,
3. GPU and launch policy,
4. privacy policy,
5. benchmark/evaluation scope,
6. baseline choices,
7. target submission format.

The first version can start without GPU if it only indexes metrics, logs, configs, manifests, confusion matrices, and manuscript files.

## Recommended First Start

Start with **GeoAgent-RAG v0.1** using:

- metric files,
- training histories,
- confusion matrices,
- configs,
- scripts,
- data manifests,
- reproducibility hashes,
- paper/manuscript files.

Do **not** start with full image-chip indexing unless storage and compute are confirmed.

This first version should support:

- artifact-grounded question answering,
- manuscript claim verification,
- model failure analysis,
- reproducible next-experiment planning,
- IGARSS-style report generation.

## What The User Needs To Confirm

### 1. Target Venue

Confirm:

```text
Target venue: IGARSS
```

This affects:

- paper length,
- writing style,
- evaluation depth,
- figures,
- contribution framing.

### 2. Data Location

Confirm the main Jupyter path:

```text
/net/home/chatran/research-seminar
```

Expected folders:

```text
research-seminar/
├── S2_tiff/
├── IS2_Corrected_data/
├── real_paired/
├── research_seminar_corrected_package/
├── runs/
└── paper_artifacts_tgrs/
```

Codex/Claude needs read access to these files before making claims.

### 3. Indexing Scope

Choose what the first prototype should index.

Recommended first scope:

```text
metrics + logs + configs + manifests + confusion matrices + paper files
```

Optional later scope:

```text
full image-chip indexing from real_paired/images/
```

Reason:

Full image indexing may require more storage, more time, and possibly GPU embeddings. The metadata-first version is faster and safer.

### 4. GPU Policy

Confirm whether the system may only propose experiments or may also launch training.

Safe first policy:

```yaml
allow_auto_launch_when_gpu_available: false
require_user_confirmation_for_new_experiment: true
never_kill_other_users_jobs: true
```

If the user wants automatic launch after plan approval:

```yaml
allow_auto_launch_when_gpu_available: true
require_user_confirmation_for_new_experiment: false
never_kill_other_users_jobs: true
```

Important:

- The agent must not kill other users' jobs.
- The agent must not occupy GPU with fake or dry-run training.
- The agent must not claim a result until real training artifacts exist.

### 5. Compute Budget

Confirm:

- available GPU type,
- maximum GPU-hours,
- maximum concurrent jobs,
- whether GPU memory sharing is allowed,
- minimum free GPU memory required before launch.

Example:

```yaml
gpu_name: NVIDIA RTX A6000
max_gpu_hours_first_round: 12
max_concurrent_user_jobs: 1
min_free_gpu_memory_gb: 24
```

### 6. Privacy Policy

Confirm:

- whether to use `Anonymous Authors`,
- whether to hide Jupyter usernames from manuscript text,
- whether to avoid personal names in file names,
- whether GitHub should contain full data or only hashes/manifests,
- whether raw data can be public.

Recommended:

```text
Use Anonymous Authors.
Do not put personal names/usernames in manuscript claims.
Keep raw-data release separate if files are too large or private.
```

### 7. Baselines

Choose baselines for the IGARSS paper.

Recommended baselines:

1. keyword search over files,
2. text-only RAG over README/manuscript files,
3. artifact-RAG without planning,
4. GeoAgent-RAG with claim verification,
5. GeoAgent-RAG with experiment planner.

Optional baselines:

- image-chip retrieval,
- multimodal image/text RAG,
- deterministic parser only,
- planner without evidence guardrails.

### 8. Benchmark Questions

We need a benchmark of real questions over real artifacts.

Recommended first benchmark size:

```text
50 questions
```

Final paper benchmark:

```text
100+ questions
```

Question categories:

- metric lookup,
- model comparison,
- evidence retrieval,
- failure analysis,
- claim verification,
- experiment planning,
- reproducibility audit.

### 9. Class Labels And Dataset Schema

Engineer/user should confirm:

- class names,
- class IDs,
- train/validation/test split policy,
- image-chip naming scheme,
- relationship between Sentinel-2 chips and ICESat-2 CSV rows,
- whether geolocation metadata is available.

Without this, GeoAgent-RAG can still parse metrics, but spatial/image retrieval will be incomplete.

### 10. Release Strategy

Confirm how outputs should be stored.

Options:

1. GitHub only for code, paper, tables, hashes, and small figures.
2. Git LFS for larger figures/data files.
3. External storage for raw images/TIFFs/checkpoints.
4. Private storage for sensitive or very large artifacts.

Important:

GitHub normal Git may reject files larger than 100 MB. Large remote-sensing data should likely use Git LFS or external dataset storage.

## What Codex Or Claude Code Can Start Without More Input

Codex/Claude can start these immediately once it has file access:

- create repository structure,
- write artifact scanner,
- write metric parser,
- write training-history parser,
- write confusion-matrix parser,
- build artifact catalog,
- build first retrieval index,
- create claim verifier,
- create benchmark template,
- create first CLI,
- write IGARSS paper outline,
- create architecture diagram.

No GPU is required for this first version.

## What Cannot Be Done Honestly Without Confirmation

Do not do these without confirmation:

- claim new training results,
- launch new GPU experiments,
- upload private data publicly,
- index huge image folders into GitHub without checking size,
- invent class labels,
- invent geolocation metadata,
- invent benchmark results,
- overwrite existing training artifacts.

## Inputs Needed From User

```text
1. Confirm IGARSS as target venue.
2. Confirm Jupyter project path.
3. Confirm first indexing scope.
4. Confirm whether full images can be indexed later.
5. Confirm GPU budget and launch policy.
6. Confirm privacy/anonymity rules.
7. Confirm whether raw data can be public or must stay private.
8. Confirm desired benchmark size.
```

## Inputs Needed From Research Engineer

```text
1. Dataset schema.
2. Class labels.
3. Train/validation/test split definition.
4. Training command format.
5. Config schema.
6. Output/run folder schema.
7. Evaluation script behavior.
8. GPU queue policy.
9. Large-file storage policy.
```

## Inputs Needed From AI Coding Agent

The AI coding agent needs:

```text
1. Repo path.
2. Jupyter file path.
3. Permission to read artifacts.
4. Permission to create source files.
5. Permission to run CPU-only tests.
6. Explicit permission before launching real training.
7. Final implementation mode: CLI, notebook, web app, or API.
```

Recommended implementation mode for v0.1:

```text
Python CLI first.
```

Reason:

CLI is easier to test, reproduce, and cite in an IGARSS paper. A web UI can come later.

## First Build Milestone

GeoAgent-RAG v0.1 is complete when:

- `artifact_catalog.jsonl` is created from real files,
- `metrics_table.csv` matches real saved metric files,
- confusion matrices are parsed,
- 50 benchmark questions are created,
- the system answers metric questions with cited evidence,
- manuscript numeric claims are verified,
- one next-experiment plan is generated but marked `proposed_not_trained`,
- no answer contains fake metrics.

## Minimal First Command Set

```bash
python scripts/ingest_research_seminar.py \
  --root /net/home/chatran/research-seminar \
  --output outputs/indexes/artifact_catalog.jsonl \
  --parse-metrics \
  --metrics-output outputs/indexes/metrics_table.csv
```

```bash
python scripts/ask_geoagent.py \
  --question "Which fusion model has the highest macro-F1 and what file proves it?"
```

```bash
python scripts/verify_claims.py \
  --manuscript paper/main.tex \
  --metrics outputs/indexes/metrics_table.csv \
  --output outputs/audit_reports/claim_verification.md
```

```bash
python scripts/propose_next_experiment.py \
  --goal improve_macro_f1 \
  --root /net/home/chatran/research-seminar \
  --budget-gpu-hours 12 \
  --output outputs/proposed_experiments/
```

## Recommended Decision For Now

Start GeoAgent-RAG with:

```text
Target: IGARSS
Mode: Python CLI
Index scope: metrics, logs, configs, manifests, confusion matrices, paper files
No GPU for v0.1
No fake data
No automatic training until a proposed experiment is reviewed
```

