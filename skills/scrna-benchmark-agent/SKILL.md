---
name: scrna-benchmark-agent
description: Route scRNA analysis subtasks to two benchmark-backed candidate tools, score both, and return a reproducible winner with rationale.
---

# scRNA Benchmark Agent Skill

## Purpose
Route scRNA analysis tasks to one of two benchmark-backed methods per step, then evaluate and report the winner.

## Inputs
- `adata_path` / `seurat_obj`
- `task` (`integration`, `annotation`, `trajectory`, `de`, `communication`)
- `constraints` (runtime, memory, species, reference atlas availability)

## Routing Table
- integration: `scVI` vs `Harmony`
- annotation: `CellTypist` vs `scANVI`
- trajectory: `Monocle3` vs `Slingshot`
- de: `MAST` vs `pseudobulk_edgeR_DESeq2`
- communication: `CellChat` vs `Liana Plus`

## Core Procedure
1. Run both candidate tools with standard defaults.
2. Compute task-specific benchmark metrics.
3. Pick winner by weighted score: quality 70%, runtime 20%, memory 10%.
4. Save full provenance in `results/decision_log.json`.

## Output Contract
- `winner`
- `metrics`
- `runtime_sec`
- `memory_mb`
- `rationale`

## Guardrails
- If references are out-of-domain, down-weight annotation confidence.
- If no biological replicates, flag DE conclusions as exploratory.
- Always emit parameter settings for reproducibility.
