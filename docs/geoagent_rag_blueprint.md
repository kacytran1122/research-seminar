# GeoAgent-RAG: Agentic Retrieval-Augmented Planning for Reproducible Remote-Sensing Experiments

## Purpose

This document defines the build plan for **GeoAgent-RAG**, an agentic retrieval-augmented system for planning, auditing, and reporting reproducible remote-sensing experiments. The intended research direction is:

> **GeoAgent-RAG: Agentic Retrieval-Augmented Planning for Reproducible Remote-Sensing Experiments**

The system should help a researcher inspect real experiment artifacts, understand model failures, decide what experiment should run next, generate reproducible training plans, and produce evidence-grounded research reports. It must not fabricate metrics, training results, dataset facts, or claims.

This is a blueprint for Codex, Claude Code, or an engineering team to implement the project from the existing `research-seminar` training assets.

## How This Must Differ From Document-Only Geoscience RAG

GeoAgent-RAG should not be a clone of a documentation question-answering system over NASA manuals, PDFs, or technical documentation. Its main contribution must be **experiment-artifact reasoning**, not document search.

| Document-only RAG | GeoAgent-RAG |
|---|---|
| Retrieves from PDFs, manuals, headings, and figure captions. | Retrieves from datasets, image chips, tabular features, metrics, logs, configs, model outputs, figures, and reproducibility manifests. |
| Answers documentation questions. | Audits trained models and plans next experiments. |
| Main metrics are text retrieval recall and answer faithfulness. | Main metrics include artifact retrieval recall, claim verification accuracy, experiment-plan validity, metric-copy accuracy, failure-analysis quality, and reproducibility. |
| Evidence is mostly text chunks. | Evidence is multimodal: text, tables, JSON metrics, CSV logs, confusion matrices, image chips, run folders, hashes, and scripts. |
| Output is a citation-constrained answer. | Output is a reproducible experiment plan, model-audit report, failure explanation, or manuscript claim check. |

The central novelty should be:

> GeoAgent-RAG retrieves from real executable remote-sensing experiment artifacts and uses that evidence to propose, verify, and document future experiments.

## Research-Seminar Assets To Use

The current `research-seminar` project provides the seed data and real training evidence.

Expected Jupyter project layout:

```text
research-seminar/
├── S2_tiff/
│   ├── s2_vis_04_20191104T194529_20191104T194523_T02CNA.tif
│   ├── s2_vis_06_20191104T194529_20191104T194523_T02CNC.tif
│   └── s2_vis_63_20191126T184459_20191126T184514_T03CWT.tif
├── IS2_Corrected_data/
│   ├── ATL03_20191104195311_05940510_T02CNA_gt1r_labeled_10m_done.csv
│   ├── ATL03_20191104195311_05940510_T02CNA_gt2r_labeled_10m_done.csv
│   ├── ATL03_20191104195311_05940510_T02CNC_gt1r_labeled_10m_done.csv
│   ├── ATL03_20191104195311_05940510_T02CNC_gt2r_labeled_10m_done.csv
│   ├── ATL03_20191126182014_09290510_T03CWT_gt1r_labeled_10m_done.csv
│   └── ATL03_20191126182014_09290510_T03CWT_gt2r_labeled_10m_done.csv
├── real_paired/
│   ├── images/
│   ├── preparation_manifest.csv
│   └── preparation_summary.json
├── research_seminar_corrected_package/
│   ├── requirements-corrected.txt
│   └── verified_training/
│       ├── prepare_real_paired_data.py
│       ├── train_deep_fusion.py
│       └── queue_a6000_training.sh
├── runs/
│   └── <variant>/
│       ├── test_metrics.json
│       ├── training_history.csv
│       ├── confusion_matrix.csv or .json
│       └── config.json or run metadata
└── paper_artifacts_tgrs/
    ├── figures/
    ├── tables/
    └── reproducibility/
```

Already observed completed primary fusion variants:

- `fixed_average`
- `concat`
- `product`
- `original_gate`
- `availability_aware_gate`

Known verified result pattern:

- Five primary variants completed with real A6000 training artifacts.
- Each primary variant has 50 logged epochs.
- `fixed_average` is currently the strongest observed variant by held-out accuracy and macro-F1.
- Existing report artifacts include metric tables, figures, confusion matrices, and reproducibility hashes.

GeoAgent-RAG must parse these files directly before making claims.

## Product Definition

GeoAgent-RAG is a research assistant with four major abilities.

### 1. Artifact-Grounded Question Answering

The system answers questions about the experiment using retrieved evidence from real files.

Example questions:

- Which fusion method performed best?
- Which metric file proves the reported macro-F1?
- Which class is hardest for product fusion?
- Which run has the lowest calibration error?
- Which training curves show overfitting?
- Which figures can be used in the manuscript?

Every answer must include:

- answer,
- evidence file paths,
- extracted values,
- confidence level,
- unsupported-claim warning if evidence is missing.

### 2. Model-Audit and Failure Analysis

The system retrieves confusion matrices, misclassified samples, training histories, and model outputs to explain model behavior.

Example tasks:

- Identify weak classes.
- Compare fusion variants by class-wise recall.
- Find overconfident wrong predictions.
- Retrieve image chips similar to failed samples.
- Explain why accuracy alone is misleading.

### 3. Claim Verification for Manuscripts

The system checks paper claims against saved artifacts.

Example:

Input claim:

> Fixed-average fusion achieved 0.9567 accuracy and 0.7408 macro-F1.

Expected output:

```json
{
  "claim": "Fixed-average fusion achieved 0.9567 accuracy and 0.7408 macro-F1.",
  "status": "supported",
  "evidence": [
    "runs/fixed_average/test_metrics.json",
    "paper_artifacts_tgrs/tables/research_seminar_metrics_summary.csv"
  ],
  "verified_values": {
    "accuracy": 0.9567,
    "macro_f1": 0.7408
  }
}
```

### 4. Agentic Experiment Planning

The system recommends the next experiment based only on retrieved evidence.

Example:

User asks:

> What should we train next to improve minority-class performance?

GeoAgent-RAG should:

1. retrieve confusion matrices,
2. identify minority-class failures,
3. retrieve configs from previous runs,
4. propose one or more next experiments,
5. produce exact config diffs,
6. explain the expected reason,
7. mark the plan as unproven until trained,
8. never claim improvement before training finishes.

## Non-Negotiable Rules

1. **No fake data.**
   - Never generate fake metrics.
   - Never invent completed training.
   - Never fabricate dataset size, classes, runs, or GPU results.

2. **Evidence-first answers.**
   - Every numerical claim must cite a file.
   - If a value cannot be found, answer `not verified`.

3. **Planning is not proof.**
   - A proposed experiment is a hypothesis, not a result.

4. **No silent overwrite.**
   - Generated configs must be saved as new files.
   - Existing training artifacts must not be edited.

5. **Queue/GPU safety.**
   - The agent can prepare commands and launch only when authorized by the user/project rules.
   - It must not kill other users' jobs.
   - It must not occupy GPUs with dry-run or fake jobs.

6. **Reproducibility.**
   - Every generated plan must include config file path, command, environment, seed, expected output folder, and hash/provenance instructions.

## What Inputs Are Needed From People

### Inputs Needed From User

The user should provide:

1. Target venue:
   - IEEE TGRS, IGARSS, SIGSPATIAL, CIKM, WACV, Pattern Recognition, or another venue.

2. Research priority:
   - highest accuracy,
   - minority-class macro-F1,
   - calibration,
   - reproducibility,
   - interpretability,
   - geospatial split robustness.

3. Compute budget:
   - available GPUs,
   - max GPU-hours,
   - whether queue watchers may launch immediately when a GPU is available.

4. Dataset access:
   - path to `research-seminar`,
   - whether full images can be indexed,
   - whether large artifacts should use Git LFS, external object storage, or local-only storage.

5. Submission privacy:
   - anonymous author naming,
   - files that should not reveal identity,
   - repository visibility.

### Inputs Needed From Research Engineer

The engineer should provide:

1. Confirmed dataset schema:
   - class labels,
   - split file format,
   - image-chip filename convention,
   - relationship between CSV rows and image chips.

2. Training command schema:
   - how to train each fusion model,
   - config file format,
   - output folder convention,
   - GPU selector behavior.

3. Run metadata schema:
   - where metrics are stored,
   - where histories are stored,
   - how confusion matrices are saved,
   - how model checkpoints are named.

4. Evaluation scripts:
   - test-set evaluation,
   - confusion matrix generation,
   - calibration evaluation,
   - per-class reporting.

5. Safety rules:
   - maximum concurrent jobs,
   - queue policy,
   - what counts as GPU availability,
   - how to avoid interrupting other users.

### Inputs Needed From Codex Or Claude Code

The coding agent needs:

1. Local or Jupyter path to the repo.
2. Read access to training artifacts.
3. Permission to create new source files.
4. Permission to run non-training tests.
5. Explicit permission before launching real GPU training.
6. Target implementation stack:
   - Python CLI only,
   - FastAPI backend,
   - Streamlit dashboard,
   - notebook interface,
   - or full web app.

## Recommended Repository Structure

```text
geoagent_rag/
├── README.md
├── pyproject.toml
├── configs/
│   ├── geoagent.yaml
│   ├── index_local.yaml
│   ├── planner_rules.yaml
│   └── evaluation.yaml
├── geoagent/
│   ├── __init__.py
│   ├── cli.py
│   ├── config.py
│   ├── schemas.py
│   ├── ingest/
│   │   ├── artifact_scanner.py
│   │   ├── metrics_parser.py
│   │   ├── history_parser.py
│   │   ├── confusion_parser.py
│   │   ├── image_indexer.py
│   │   └── provenance.py
│   ├── retrieval/
│   │   ├── vector_store.py
│   │   ├── lexical_store.py
│   │   ├── hybrid_retriever.py
│   │   ├── reranker.py
│   │   └── evidence_pack.py
│   ├── reasoning/
│   │   ├── answerer.py
│   │   ├── claim_verifier.py
│   │   ├── failure_analyzer.py
│   │   └── planner.py
│   ├── experiments/
│   │   ├── config_generator.py
│   │   ├── launch_guard.py
│   │   ├── queue_writer.py
│   │   └── result_monitor.py
│   ├── reporting/
│   │   ├── audit_report.py
│   │   ├── manuscript_check.py
│   │   └── figures.py
│   └── eval/
│       ├── benchmark_loader.py
│       ├── metrics.py
│       └── run_eval.py
├── scripts/
│   ├── ingest_research_seminar.py
│   ├── ask_geoagent.py
│   ├── verify_claims.py
│   ├── propose_next_experiment.py
│   └── build_benchmark.py
├── tests/
│   ├── test_metrics_parser.py
│   ├── test_claim_verifier.py
│   ├── test_planner_guardrails.py
│   └── test_no_fake_metrics.py
├── benchmark/
│   ├── questions.jsonl
│   ├── expected_evidence.jsonl
│   └── grading_rubric.md
└── outputs/
    ├── indexes/
    ├── evidence_packs/
    ├── audit_reports/
    └── proposed_experiments/
```

## Core Data Schemas

### Artifact Record

```python
from dataclasses import dataclass
from typing import Literal, Optional

ArtifactKind = Literal[
    "metric_json",
    "training_history_csv",
    "confusion_matrix",
    "config",
    "script",
    "image_chip",
    "tabular_features",
    "manifest",
    "figure",
    "paper_text",
    "checkpoint"
]

@dataclass
class ArtifactRecord:
    artifact_id: str
    path: str
    kind: ArtifactKind
    run_id: Optional[str]
    variant: Optional[str]
    split: Optional[str]
    hash_sha256: str
    summary: str
    extracted_values: dict
```

### Evidence Pack

```python
@dataclass
class EvidenceItem:
    path: str
    kind: str
    quote_or_value: str
    confidence: float
    hash_sha256: str

@dataclass
class EvidencePack:
    question: str
    answerable: bool
    items: list[EvidenceItem]
    missing_requirements: list[str]
```

### Experiment Plan

```python
@dataclass
class ExperimentPlan:
    plan_id: str
    hypothesis: str
    evidence_paths: list[str]
    config_path: str
    command: str
    expected_outputs: list[str]
    risk_level: str
    status: str  # proposed, queued, running, completed, failed
    must_not_claim_improvement_until_completed: bool = True
```

## Implementation Phase 1: Artifact Scanner

Goal: scan the real `research-seminar` folder and create a machine-readable artifact catalog.

### File: `geoagent/ingest/artifact_scanner.py`

Responsibilities:

- Walk the project folder.
- Detect metrics, histories, confusion matrices, configs, scripts, images, and manifests.
- Compute SHA-256 hash for each file.
- Save `outputs/indexes/artifact_catalog.jsonl`.

Pseudo-code:

```python
from pathlib import Path
import hashlib
import json

def sha256_file(path: Path, chunk_size: int = 1024 * 1024) -> str:
    h = hashlib.sha256()
    with path.open("rb") as f:
        for chunk in iter(lambda: f.read(chunk_size), b""):
            h.update(chunk)
    return h.hexdigest()

def classify_artifact(path: Path) -> str:
    name = path.name.lower()
    suffix = path.suffix.lower()
    if name == "test_metrics.json":
        return "metric_json"
    if name == "training_history.csv":
        return "training_history_csv"
    if "confusion" in name:
        return "confusion_matrix"
    if name.endswith((".yaml", ".yml", ".json")) and "config" in name:
        return "config"
    if suffix in [".py", ".sh"]:
        return "script"
    if suffix in [".png", ".jpg", ".jpeg", ".tif", ".tiff"]:
        return "image_chip"
    if suffix == ".csv":
        return "tabular_features"
    if "manifest" in name:
        return "manifest"
    return "other"

def scan_project(root: str, output_path: str) -> None:
    root_path = Path(root).expanduser().resolve()
    records = []
    for path in root_path.rglob("*"):
        if not path.is_file():
            continue
        kind = classify_artifact(path)
        if kind == "other":
            continue
        records.append({
            "artifact_id": sha256_file(path)[:16],
            "path": str(path),
            "kind": kind,
            "hash_sha256": sha256_file(path),
            "summary": "",
            "extracted_values": {}
        })
    Path(output_path).parent.mkdir(parents=True, exist_ok=True)
    with open(output_path, "w") as f:
        for record in records:
            f.write(json.dumps(record) + "\n")
```

## Implementation Phase 2: Metrics Parser

Goal: extract real metrics from completed runs only.

### File: `geoagent/ingest/metrics_parser.py`

Responsibilities:

- Parse `runs/*/test_metrics.json`.
- Read only verified completed branch if present.
- Extract accuracy, macro-F1, balanced accuracy, kappa, ECE, per-class scores, and confusion matrices.
- Save normalized tables.

Important rule:

> If the metric file is missing or incomplete, mark the value as unavailable. Do not impute.

Pseudo-code:

```python
import json
from pathlib import Path

PRIMARY_KEYS = [
    "accuracy",
    "macro_f1",
    "balanced_accuracy",
    "cohen_kappa",
    "ece"
]

def read_metric_file(path: Path) -> dict:
    data = json.loads(path.read_text())
    if "complete" in data and isinstance(data["complete"], dict):
        data = data["complete"]
    result = {}
    for key in PRIMARY_KEYS:
        result[key] = data.get(key, None)
    result["source_path"] = str(path)
    result["is_complete"] = all(result[k] is not None for k in PRIMARY_KEYS)
    return result

def parse_all_runs(root: str) -> list[dict]:
    root_path = Path(root).expanduser().resolve()
    results = []
    for metric_file in root_path.glob("runs/*/test_metrics.json"):
        variant = metric_file.parent.name
        row = read_metric_file(metric_file)
        row["variant"] = variant
        results.append(row)
    return results
```

## Implementation Phase 3: Retrieval Index

Goal: retrieve the right evidence for a question.

Use two retrieval layers:

1. **Lexical retrieval** for exact paths, metric names, variants, class labels, and commands.
2. **Vector retrieval** for natural-language questions and summaries.

Recommended stack:

- `sqlite` for artifact catalog.
- `faiss` or `chromadb` for local vector search.
- `rank_bm25` or SQLite FTS for lexical retrieval.
- `sentence-transformers` for local embeddings if allowed.
- CLIP-like image embeddings for image-chip retrieval if GPU/CPU budget allows.

### File: `geoagent/retrieval/hybrid_retriever.py`

Responsibilities:

- Accept a question.
- Retrieve candidate artifacts.
- Rerank by artifact type and query intent.
- Return an evidence pack.

Intent examples:

| Query intent | Preferred artifacts |
|---|---|
| Best model | metric JSON, summary CSV |
| Training stability | training history CSV |
| Failure analysis | confusion matrix, prediction outputs |
| Reproducibility | config, script, hash manifest |
| Image example | image chip, manifest row |
| Paper claim | metric JSON, table, manuscript source |

Pseudo-code:

```python
class HybridRetriever:
    def __init__(self, lexical_store, vector_store):
        self.lexical_store = lexical_store
        self.vector_store = vector_store

    def retrieve(self, query: str, top_k: int = 12) -> list[dict]:
        lexical = self.lexical_store.search(query, top_k=top_k)
        vector = self.vector_store.search(query, top_k=top_k)
        merged = self.merge_and_rerank(query, lexical + vector)
        return merged[:top_k]

    def merge_and_rerank(self, query: str, candidates: list[dict]) -> list[dict]:
        seen = {}
        for item in candidates:
            key = item["path"]
            seen[key] = max(seen.get(key, item), item, key=lambda x: x.get("score", 0))
        return sorted(seen.values(), key=lambda x: x.get("score", 0), reverse=True)
```

## Implementation Phase 4: Evidence-Grounded Answerer

Goal: answer questions only from retrieved artifacts.

### File: `geoagent/reasoning/answerer.py`

Responsibilities:

- Take user question and evidence pack.
- Generate an answer with citations.
- Refuse unsupported numerical claims.
- Distinguish verified facts from interpretations.

Required answer format:

```text
Answer:
<short answer>

Evidence:
- <file path>: <exact value or short extracted support>

Limitations:
- <what could not be verified>
```

Prompt rules for LLM:

```text
You are GeoAgent-RAG. Use only the provided evidence.
Never invent metrics, paths, classes, training status, or experiment outcomes.
If the evidence does not support a claim, say "not verified."
Separate "verified facts" from "interpretation."
Every numeric value must cite a source path.
```

## Implementation Phase 5: Claim Verifier

Goal: check manuscript claims against real artifacts.

### File: `geoagent/reasoning/claim_verifier.py`

Responsibilities:

- Extract numerical claims from markdown or LaTeX manuscript files.
- Retrieve supporting artifact evidence.
- Compare stated numbers with parsed numbers.
- Produce pass/fail/warning report.

Claim statuses:

- `supported`
- `unsupported`
- `contradicted`
- `needs_manual_review`
- `not_numeric`

Pseudo-code:

```python
import re

NUMBER_PATTERN = re.compile(r"\\b\\d+\\.\\d+\\b")

def extract_numeric_claims(text: str) -> list[str]:
    sentences = re.split(r"(?<=[.!?])\\s+", text)
    return [s for s in sentences if NUMBER_PATTERN.search(s)]

def verify_claim(claim: str, metric_table: list[dict]) -> dict:
    numbers = NUMBER_PATTERN.findall(claim)
    evidence = []
    for row in metric_table:
        for key, value in row.items():
            if isinstance(value, float):
                for n in numbers:
                    if abs(float(n) - value) < 1e-4:
                        evidence.append({
                            "metric": key,
                            "value": value,
                            "variant": row.get("variant"),
                            "source_path": row.get("source_path")
                        })
    return {
        "claim": claim,
        "status": "supported" if evidence else "unsupported",
        "evidence": evidence
    }
```

## Implementation Phase 6: Failure Analyzer

Goal: transform metrics and confusion matrices into actionable model-audit findings.

### File: `geoagent/reasoning/failure_analyzer.py`

Responsibilities:

- Load confusion matrices.
- Compute per-class recall and precision.
- Identify largest off-diagonal errors.
- Connect findings to variant names and source files.
- Optionally retrieve image chips related to failure cases if prediction-level outputs exist.

Example output:

```text
Product fusion has strong majority-class recovery but weaker minority-class separation.
The row-normalized confusion matrix shows class-2 examples are often predicted as class 1.
Evidence: runs/product/confusion_matrix.csv
```

## Implementation Phase 7: Agentic Experiment Planner

Goal: propose the next reproducible experiment.

### File: `geoagent/reasoning/planner.py`

Responsibilities:

- Read audit findings.
- Retrieve previous configs.
- Generate a conservative next experiment.
- Save a new config file.
- Save a human-readable plan.
- Require explicit launch approval unless the user has already set an automatic launch policy.

Planner input:

```json
{
  "goal": "improve macro_f1",
  "constraints": {
    "max_gpu_hours": 12,
    "allowed_models": ["fixed_average", "availability_aware_gate"],
    "allowed_changes": ["class_weight", "focal_loss", "learning_rate", "seed"],
    "do_not_modify_existing_artifacts": true
  }
}
```

Planner output:

```json
{
  "plan_id": "geoagent_plan_0001",
  "hypothesis": "Class-weighted loss may improve minority-class macro-F1.",
  "evidence_paths": [
    "runs/product/confusion_matrix.csv",
    "runs/fixed_average/test_metrics.json"
  ],
  "new_config_path": "outputs/proposed_experiments/geoagent_plan_0001.yaml",
  "command": "python -m research_training.train --config outputs/proposed_experiments/geoagent_plan_0001.yaml",
  "status": "proposed_not_trained"
}
```

Planning guardrails:

- The planner may propose experiments.
- The planner may write config files.
- The planner may write launch scripts.
- The planner must not claim results before training.
- The planner must not delete old runs.
- The planner must not overwrite old configs.

## Implementation Phase 8: Launch Guard

Goal: safely launch training only when allowed.

### File: `geoagent/experiments/launch_guard.py`

Responsibilities:

- Check GPU availability.
- Check memory availability.
- Check whether other users are running jobs.
- Check project-specific priority rules.
- Confirm command points to a real config.
- Log the launch decision.

Recommended launch policy:

```yaml
launch_policy:
  allow_auto_launch_when_gpu_available: false
  require_user_confirmation_for_new_experiment: true
  never_kill_other_users_jobs: true
  max_concurrent_user_jobs: 1
  min_free_gpu_memory_gb: 24
  allowed_gpu_names:
    - NVIDIA RTX A6000
```

If the user explicitly says:

> Run when GPU is available.

Then the policy can be changed to:

```yaml
allow_auto_launch_when_gpu_available: true
require_user_confirmation_for_new_experiment: false
```

Even then, the system must not kill or preempt other users.

## Implementation Phase 9: Benchmark

Goal: evaluate GeoAgent-RAG scientifically.

### Benchmark Categories

1. **Metric lookup**
   - Which model has highest accuracy?
   - What is the macro-F1 of fixed-average fusion?

2. **Evidence retrieval**
   - Which files support the reported confusion matrix?
   - Which files prove the dataset preparation?

3. **Failure analysis**
   - Which class has the weakest recall?
   - What is the largest confusion error?

4. **Claim verification**
   - Is this manuscript statement supported?
   - Does this number match the metric JSON?

5. **Planning**
   - Given this failure pattern, propose the next experiment.
   - Does the plan cite evidence?
   - Is the generated config valid?

6. **Safety**
   - Does the system refuse to invent missing metrics?
   - Does it avoid claiming an untrained plan improved results?

### Evaluation Metrics

| Metric | Meaning |
|---|---|
| Evidence Recall@k | Whether the correct artifact appears in top-k retrieved files. |
| Answer Exactness | Whether reported values match source files. |
| Citation Precision | Whether cited files actually support the answer. |
| Unsupported-Claim Detection | Whether missing/false claims are flagged. |
| Plan Validity | Whether proposed configs can run. |
| Plan Grounding | Whether plan rationale cites real evidence. |
| No-Fabrication Rate | Percentage of answers without invented values. |
| Human Audit Score | Expert rating of usefulness and correctness. |

## Step-By-Step Build Plan

### Step 0: Freeze Real Inputs

Create a read-only snapshot of the source project.

Checklist:

- Record source path.
- Record file count.
- Record total size.
- Hash important manifests.
- Do not move or edit training outputs.

Expected output:

```text
outputs/indexes/source_snapshot.json
outputs/indexes/source_snapshot.sha256
```

### Step 1: Build Artifact Catalog

Command:

```bash
python scripts/ingest_research_seminar.py \
  --root /net/home/chatran/research-seminar \
  --output outputs/indexes/artifact_catalog.jsonl
```

Expected output:

```text
outputs/indexes/artifact_catalog.jsonl
```

### Step 2: Parse Metrics

Command:

```bash
python scripts/ingest_research_seminar.py \
  --root /net/home/chatran/research-seminar \
  --parse-metrics \
  --metrics-output outputs/indexes/metrics_table.csv
```

Expected output:

```text
outputs/indexes/metrics_table.csv
outputs/indexes/training_history_summary.csv
outputs/indexes/confusion_matrix_summary.json
```

### Step 3: Build Hybrid Index

Command:

```bash
python scripts/build_benchmark.py \
  --catalog outputs/indexes/artifact_catalog.jsonl \
  --metrics outputs/indexes/metrics_table.csv \
  --output outputs/indexes/hybrid_index/
```

Expected output:

```text
outputs/indexes/hybrid_index/
├── lexical.sqlite
├── vectors.faiss
└── metadata.json
```

### Step 4: Test Basic QA

Command:

```bash
python scripts/ask_geoagent.py \
  --question "Which fusion model has the highest macro-F1 and what file proves it?"
```

Expected answer:

- fixed-average fusion,
- macro-F1 value,
- cited metric file,
- no unsupported claims.

### Step 5: Test Claim Verification

Command:

```bash
python scripts/verify_claims.py \
  --manuscript paper/main.tex \
  --metrics outputs/indexes/metrics_table.csv \
  --output outputs/audit_reports/claim_verification.md
```

Expected output:

```text
outputs/audit_reports/claim_verification.md
```

### Step 6: Test Failure Analysis

Command:

```bash
python scripts/ask_geoagent.py \
  --question "Which class is most confused in the product fusion model?"
```

Expected answer:

- cites product confusion matrix,
- reports exact confusion values,
- explains only what the matrix supports.

### Step 7: Generate Next Experiment Plan

Command:

```bash
python scripts/propose_next_experiment.py \
  --goal improve_macro_f1 \
  --root /net/home/chatran/research-seminar \
  --budget-gpu-hours 12 \
  --output outputs/proposed_experiments/
```

Expected output:

```text
outputs/proposed_experiments/geoagent_plan_0001.md
outputs/proposed_experiments/geoagent_plan_0001.yaml
outputs/proposed_experiments/geoagent_plan_0001_launch.sh
```

### Step 8: Human Review Before Training

Before launching, a human should review:

- hypothesis,
- evidence,
- config diff,
- GPU budget,
- output path,
- launch command.

If automatic launch has been authorized, the launch guard may start when GPU conditions are met.

### Step 9: Run Training

Command example:

```bash
bash outputs/proposed_experiments/geoagent_plan_0001_launch.sh
```

Expected output:

```text
runs/geoagent_plan_0001/
├── config.yaml
├── training_history.csv
├── test_metrics.json
├── confusion_matrix.csv
├── provenance.json
└── logs/
```

### Step 10: Compare Results

Command:

```bash
python scripts/ask_geoagent.py \
  --question "Did geoagent_plan_0001 improve macro-F1 compared with fixed_average?"
```

Valid answer before training:

```text
Not verified. geoagent_plan_0001 has not completed training.
```

Valid answer after training:

```text
The plan improved/did not improve macro-F1 based on runs/geoagent_plan_0001/test_metrics.json.
```

## Minimum Viable Paper Contribution

The paper should not claim full autonomous science unless the system truly runs and verifies new experiments. The safer first paper contribution is:

1. A geospatial experiment-artifact RAG system.
2. A claim verifier for remote-sensing manuscripts.
3. A failure analyzer grounded in confusion matrices and metric files.
4. A guarded planner that produces reproducible next-experiment configs.
5. An evaluation benchmark over real research-seminar artifacts.

Recommended paper framing:

> We introduce GeoAgent-RAG, an artifact-grounded retrieval and planning system for reproducible remote-sensing experiments. Unlike document-centered geoscience RAG systems, GeoAgent-RAG indexes executable experiment evidence, including paired Sentinel-2/ICESat-2 data manifests, training histories, test metrics, confusion matrices, model configs, scripts, and provenance hashes. The system supports evidence-grounded question answering, manuscript claim verification, model failure analysis, and guarded next-experiment planning.

## Suggested Experiments For The GeoAgent-RAG Paper

### Experiment 1: Artifact Retrieval Accuracy

Question:

Can the system retrieve the correct evidence file for metric and model questions?

Metrics:

- Recall@1,
- Recall@3,
- Recall@5,
- MRR.

### Experiment 2: Numerical Answer Faithfulness

Question:

Does the system copy metric values correctly?

Metrics:

- exact numeric match,
- absolute error,
- unsupported-claim refusal rate.

### Experiment 3: Manuscript Claim Verification

Question:

Can the system detect unsupported or contradicted claims?

Data:

- real claims from the manuscript,
- edited claims with wrong numbers,
- missing-evidence claims.

Metrics:

- precision,
- recall,
- F1,
- false support rate.

### Experiment 4: Failure Analysis Quality

Question:

Does the system identify real weak classes and confusion patterns?

Metrics:

- agreement with deterministic confusion-matrix parser,
- human usefulness score,
- citation correctness.

### Experiment 5: Experiment Plan Validity

Question:

Are generated experiment plans executable and evidence-grounded?

Metrics:

- config validity,
- command validity,
- evidence grounding,
- safety-rule compliance,
- human expert rating.

### Experiment 6: Optional Real Training Loop

Question:

Can the system propose a next experiment that improves a target metric?

Important:

- This is optional and expensive.
- It should be included only if real GPU training is completed.
- Do not include mock improvements.

## Candidate Next Experiments The Planner Could Propose

The planner should choose from a controlled set of safe modifications.

### Candidate A: Class-Weighted Loss

Hypothesis:

Class imbalance reduces minority-class recall. Class-weighted cross entropy may improve macro-F1.

Required evidence:

- class counts from manifest,
- confusion matrix,
- per-class recall.

Allowed config changes:

- `loss: weighted_cross_entropy`
- `class_weights: computed_from_train_split`

### Candidate B: Focal Loss

Hypothesis:

Focal loss may help if the model is dominated by easy majority-class examples.

Required evidence:

- class imbalance,
- high majority-class recall,
- low minority-class recall.

Allowed config changes:

- `loss: focal`
- `gamma: 2.0`
- `alpha: computed_from_train_split`

### Candidate C: Calibration-Aware Selection

Hypothesis:

The best accuracy model may not be the best calibrated model.

Required evidence:

- ECE values,
- reliability diagram,
- overconfident-error samples.

Allowed config changes:

- temperature scaling,
- validation-only calibration,
- no retraining required for first pass.

### Candidate D: Geographic Split Stress Test

Hypothesis:

Random held-out performance may overestimate generalization if nearby chips leak across splits.

Required evidence:

- chip geolocation,
- tile IDs,
- split metadata.

Allowed config changes:

- create geographic split,
- retrain same model under same budget.

### Candidate E: Photon-Branch Ablation

Hypothesis:

ICESat-2 features improve or hurt certain classes depending on fusion method.

Required evidence:

- image-only baseline,
- photon-only baseline,
- fusion metrics.

Allowed config changes:

- disable photon branch,
- disable image branch,
- compare same split.

## Minimal Code Entry Points

### `scripts/ingest_research_seminar.py`

```python
def main():
    args = parse_args()
    catalog = scan_project(args.root)
    save_catalog(catalog, args.output)
    if args.parse_metrics:
        metrics = parse_all_runs(args.root)
        save_metrics(metrics, args.metrics_output)
```

### `scripts/ask_geoagent.py`

```python
def main():
    args = parse_args()
    retriever = load_retriever(args.index)
    evidence = retriever.retrieve(args.question)
    answer = answer_from_evidence(args.question, evidence)
    print(answer.to_markdown())
```

### `scripts/verify_claims.py`

```python
def main():
    manuscript_text = read_text(args.manuscript)
    claims = extract_numeric_claims(manuscript_text)
    metrics = load_metrics(args.metrics)
    report = [verify_claim(claim, metrics) for claim in claims]
    write_markdown_report(report, args.output)
```

### `scripts/propose_next_experiment.py`

```python
def main():
    evidence = collect_failure_evidence(args.root)
    plan = propose_plan(goal=args.goal, evidence=evidence, budget=args.budget_gpu_hours)
    validate_plan(plan)
    write_plan_files(plan, args.output)
```

## Suggested README Text

```text
# GeoAgent-RAG

GeoAgent-RAG is an artifact-grounded retrieval and planning system for reproducible remote-sensing experiments. It indexes real Sentinel-2/ICESat-2 experiment artifacts, including metrics, training logs, confusion matrices, configs, scripts, data manifests, and figures. The system answers questions with file-level evidence, verifies manuscript claims, analyzes model failures, and proposes guarded next-experiment plans.

GeoAgent-RAG does not fabricate results. Proposed experiments are marked as untrained until real training artifacts are produced.
```

## What The First Prototype Should Do

The first useful prototype should support:

1. index `research-seminar`,
2. answer metric questions,
3. cite evidence paths,
4. verify manuscript numbers,
5. summarize confusion-matrix failures,
6. propose one next config without launching it,
7. generate a reproducibility report.

Do not start with full autonomy. Start with trustworthy retrieval and verification.

## Definition Of Done For Prototype

Prototype is done when:

- `artifact_catalog.jsonl` is created from real files.
- `metrics_table.csv` matches saved run artifacts.
- At least 50 benchmark questions are written.
- Evidence Recall@5 is reported.
- Numeric answer accuracy is reported.
- Claim verification report is generated for the current manuscript.
- One proposed experiment config is generated and validated.
- No answer includes fake metrics.
- All outputs include source paths and hashes.

## Definition Of Done For Publishable Paper

Paper-ready status requires:

- full benchmark with train/test question split,
- baseline comparisons against standard document-RAG and keyword search,
- real artifact retrieval evaluation,
- claim-verification evaluation,
- failure-analysis evaluation,
- planner validity evaluation,
- at least one human audit,
- optional real training loop if compute is available,
- reproducible code and dataset/artifact release plan.

## Baselines To Compare Against

1. Keyword search over files.
2. Text-only RAG over README/manuscript files.
3. Document-only RAG over extracted text.
4. Artifact-only deterministic parser.
5. GeoAgent-RAG without reranking.
6. GeoAgent-RAG without claim-verification guardrails.

## Expected Risks

| Risk | Mitigation |
|---|---|
| Large image files exceed GitHub limits. | Use Git LFS or external dataset storage; keep hashes in Git. |
| LLM invents numbers. | Require deterministic metric parser and file-level citations. |
| Planner proposes invalid configs. | Validate config schema before saving. |
| Agent wastes GPU. | Launch guard, budget limit, explicit status labels. |
| Paper looks like document RAG. | Emphasize experiment artifacts, model auditing, and planning. |
| Incomplete artifacts. | Mark as incomplete; do not impute. |

## Recommended Venue Strategy

Best-fit venues depend on final implementation:

- **IEEE TGRS**: best if the system is deeply tied to remote-sensing experiments and real Sentinel-2/ICESat-2 evidence.
- **IGARSS**: realistic conference path for remote-sensing system and case-study results.
- **ACM SIGSPATIAL**: strong if spatial indexing and geospatial retrieval are central.
- **CIKM / SIGIR**: strong if retrieval and claim verification are central.
- **WACV**: possible if image-chip retrieval and vision-language grounding become central.
- **ACL / EMNLP / NAACL**: possible only if the RAG method itself is novel beyond geospatial application.

Recommended first target:

> **IGARSS or IEEE TGRS**, depending on how complete the evaluation becomes.

## Suggested Paper Title Options

1. **GeoAgent-RAG: Agentic Retrieval-Augmented Planning for Reproducible Remote-Sensing Experiments**
2. **GeoAgent-RAG: Evidence-Grounded Experiment Planning for Multimodal Remote-Sensing Models**
3. **Artifact-Grounded Retrieval-Augmented Planning for Reproducible Geoscience Machine Learning**
4. **From Metrics to Next Experiments: Agentic RAG for Remote-Sensing Model Audits**
5. **Retrieval-Augmented Failure Analysis and Experiment Planning for Sentinel-2/ICESat-2 Fusion Models**

## One-Sentence Summary

GeoAgent-RAG is a geospatial research assistant that retrieves from real remote-sensing experiment artifacts, verifies scientific claims, explains model failures, and proposes reproducible next experiments without fabricating results.

