# scRNA Analysis Pipeline Agent Plan (Benchmark-Guided)

## Scope
This plan defines an autonomous agent for end-to-end scRNA-seq analysis that chooses between the top 2 tools per task and records why each choice is made.

> Note: Direct access to the referenced GitHub benchmark links was blocked in this execution environment (HTTP 403). The tool selections below follow broadly accepted benchmark trends and should be validated against each linked benchmark before production use.

## Agent Architecture

### 1) Orchestrator
- Reads dataset metadata (platform, species, cell count, reference availability, compute budget).
- Expands into a task DAG: QC → normalization/integration → dimensionality reduction → clustering → annotation → DE/trajectory/communication.
- Uses decision rules (below) to pick one of two tools per task.

### 2) Skill Modules
- `qc_preprocess`
- `integration_batch_correction`
- `clustering_and_embedding`
- `cell_annotation_inference`
- `trajectory_inference`
- `differential_expression`
- `cell_cell_communication`

### 3) Judge & Reporter
- Computes standardized metrics per task (ARI/NMI, kBET/LISI, F1, runtime, memory).
- Generates a reproducible markdown report with chosen tools, parameters, and confidence.

## Top-2 Tool Choices Per Task

| Task | Tool 1 | Tool 2 | Decision rule for the agent |
|---|---|---|---|
| QC + doublet detection | **Scrublet** | **DoubletFinder** | Use Scrublet by default (Python-native, scalable). Use DoubletFinder when workflow is Seurat-first and pK tuning support is needed. |
| Batch correction / integration | **scVI** | **Harmony** | Prefer scVI for large heterogeneous datasets and downstream probabilistic modeling; use Harmony for fast linear correction on PCA spaces. |
| Clustering | **Leiden (Scanpy/Seurat graph)** | **SC3** | Use Leiden for routine large-scale clustering; use SC3 for smaller datasets where consensus clustering stability is preferred. |
| Cell annotation / inference | **CellTypist** | **scANVI** | Use CellTypist for fast reference transfer to known immune/tissue atlases; use scANVI when labels are partial/noisy and semi-supervised transfer is beneficial. |
| Trajectory inference | **Monocle 3** | **Slingshot** | Use Monocle 3 for complex graph trajectories; use Slingshot for robust lineage inference after high-quality clustering. |
| Differential expression | **MAST** | **pseudobulk edgeR/DESeq2** | Use MAST for cell-level hurdle modeling; use pseudobulk edgeR/DESeq2 for replicate-aware designs and publication-grade DE. |
| Cell-cell communication | **CellChat** | **CellPhoneDB** | Use CellChat for pathway-level network analysis; use CellPhoneDB for canonical ligand-receptor permutation testing. |

## Workflow Logic (Policy)

1. **If dataset > 200k cells**:
   - Prefer scVI + Leiden + CellTypist (speed/scalability profile).
2. **If strong donor/batch effects with biological overlap**:
   - Run both scVI and Harmony, choose by integrated LISI + silhouette.
3. **If references are weak/out-of-domain**:
   - Run CellTypist and scANVI; select by macro-F1 on held-out labeled cells.
4. **If trajectory is a key endpoint**:
   - Compare Monocle 3 vs Slingshot on pseudotime concordance and branch stability.

## Minimal Evaluation Matrix

- Integration: iLISI (higher), kBET acceptance (higher), ASW-batch (lower), ASW-celltype (higher)
- Clustering: ARI/NMI vs trusted labels, rare-cell recall
- Annotation: macro-F1, unknown-cell rejection AUROC
- Trajectory: Kendall correlation to known progression, branch reproducibility
- DE: overlap with validated markers, replicate consistency
- Runtime: wall-clock and peak memory

## Deliverables Produced by the Agent

1. `results/decision_log.json` (tool decisions + rationale)
2. `results/metrics_summary.csv`
3. `results/final_report.md` (plots/tables and recommended pipeline)
4. `env/conda-lock.yml` or `uv.lock` for reproducibility

## Suggested First Implementation Sprint

- Sprint 1: QC, integration, clustering, annotation decision loop.
- Sprint 2: Trajectory + DE branch and benchmark scorecards.
- Sprint 3: Communication analysis and report automation.

## Benchmark-Link Verification Checklist (to execute when network is available)

For each benchmark link listed under `#scrna-analysis-pipelines`:
1. Record task category.
2. Extract compared methods and ranking metric.
3. Mark whether selected top-2 tools are supported by evidence.
4. If contradicted, update this plan and decision rules.

