# ECHO-PPI: Trustworthy AI for Evidence-Bundled Detection of Overlapping Protein Modules in Protein-Protein Interaction Networks

ECHO-PPI is a computational biology framework for overlap-aware community detection in protein-protein interaction (PPI) networks. Its main contribution is not a claim of state-of-the-art predictive F1. Instead, ECHO-PPI adds assignment-level auditability: each protein-module assignment can be inspected through topology, semantic, Gene Ontology (GO), confidence-label, and evidence-bundle fields.

Repository: <https://github.com/MehrdadJalali-AI/ECHO-PPI>

## Overview

The pipeline starts from a weighted PPI graph and GO/text profiles, constructs multimodal evidence features, generates and scores candidate modules, applies overlap-aware seeding and recall-safe supplementation, and exports confidence labels plus evidence bundles for curator-facing review.

On the cleaned Gavin yeast benchmark, exact ClusterONE is the strongest predictive baseline in the current run. A second Krogan 2006 yeast benchmark is included to test transferability across PPI resources. ECHO-PPI remains close to MCL and MCL+overlap while adding assignment-level evidence bundles and confidence labels. The scientifically defensible contribution is transparent overlapping assignment rather than predictive superiority.

## Scientific motivation

PPI modules are noisy and overlapping. Proteins can participate in multiple complexes or pathway contexts, and curated gold standards are incomplete and sensitive to identifier mapping. A single F1 score is useful but insufficient for review workflows, because it does not explain why an individual protein was assigned to a module.

ECHO-PPI addresses this gap by exporting:

- topology support,
- semantic/text support,
- GO support,
- membership/confidence labels,
- evidence-bundle records,
- case-study-ready assignment summaries.

## Repository structure

| Path | Purpose |
|---|---|
| `configs/` | Dataset and fixed-configuration YAML files |
| `data/` | Local input data location; raw third-party datasets should be obtained from original providers |
| `data_manifest/` | Notes about dataset placement and provenance |
| `echo_ppi/` | Main ECHO-PPI pipeline package |
| `src/` | Loaders, baseline utilities, GO TF-IDF, permanence, overlap, and evaluation helpers |
| `scripts/` | Reproducibility, benchmark, evidence-building, and figure-generation scripts |
| `tools/clusterone/` | Official ClusterONE JAR used for the exact Gavin external baseline |
| `results/` | Generated pipeline outputs |
| `tables/` | Manuscript-ready benchmark and auditability tables |
| `figures/` | Manuscript figures in PDF/PNG/SVG where available |
| `manuscript/` | LaTeX source, bibliography, and compiled manuscript PDF |
| `reports/` | Local audit and run reports |

## Installation

```bash
cd ECHO-PPI
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt
```

If using Conda or Mamba:

```bash
conda env create -f environment.yml
conda activate echo-ppi
```

## Data requirements

Place or symlink input files under `data/`.

| File | Description |
|---|---|
| `data/gavin2006_socioaffinities_rescaled.txt` | Gavin yeast weighted socioaffinity network |
| `data/GO.txt` | Saccharomyces Genome Database GO annotation file in GAF-like format |
| `data/gold_standards/cyc2008_yeast.csv` | Yeast complex gold standard used for local benchmarking |
| `data/4932.protein.links.detailed.v11.5.txt` | Optional STRING yeast network for feasibility analysis |
| `data/cache/4932.protein.aliases.v11.5.txt.gz` | Optional STRING alias map |
| `data/biogrid_scerivise.tab3.txt` | BioGRID yeast TAB3 file used to extract Krogan 2006 records (`PUBMED:16554755`) and optional broader BioGRID feasibility runs |

Raw Gavin, GO, STRING, BioGRID, Complex Portal, and related third-party datasets should be obtained from their original providers and used under their respective licences.

## Reproducing the main Gavin benchmark

```bash
export PYTHONPATH="$(pwd):$PYTHONPATH"
bash scripts/reproduce_echo_ppi_final.sh
```

This command runs the fixed Gavin benchmark, runs the Krogan 2006 transferability benchmark, regenerates manuscript tables, regenerates figures, and recompiles `manuscript/echo_ppi_main.pdf`.

The workflow also refreshes BH-dependent caches after nucleus-score coefficient changes, runs exact ClusterONE when `tools/clusterone/cluster_one-1.0.jar` is present, evaluates an SLPA threshold grid, writes oracle and significance-test outputs, records the Gavin embedding backend in `results/run_config.json`, and writes Krogan preprocessing/benchmark reports under `results/krogan/`.

## Krogan 2006 transferability benchmark

The second benchmark is extracted reproducibly from the local BioGRID TAB3 yeast file:

- source filter: `Publication Source == PUBMED:16554755`
- publication: Krogan et al., Nature 2006
- interaction type: physical yeast-yeast interactions
- identifier policy: SGD systematic ORF identifiers from `Systematic Name Interactor A/B`
- edge policy: remove self-loops, deduplicate undirected edges, preserve numeric BioGRID `Score` when available and otherwise assign weight `1.0`

Generated preprocessing reports:

| File | Purpose |
|---|---|
| `results/krogan/cleaning_report.csv` | Raw rows, publication rows, removed self-loops, duplicate edges, clean node/edge counts |
| `results/krogan/id_mapping_report.csv` | Identifier policy and unmapped identifier counts |
| `results/krogan/go_coverage_report.csv` | Non-generic GO coverage after yeast ORF mapping |
| `results/krogan/unmapped_proteins.csv` | Unmapped or invalid interactor identifiers |
| `results/krogan/cleaned_graph_edges.csv` | Cleaned Krogan edge list used by all methods |

The benchmark can also be run directly:

```bash
python3 scripts/evaluate_krogan_benchmark.py
python3 scripts/plot_cross_dataset_figures.py
```

## Main workflow modules

| Module or script | Role |
|---|---|
| `src/gavin_loader.py`, `src/string_loader.py` | Load weighted PPI networks and optional STRING data |
| `echo_ppi/krogan_loader.py` | Extract Krogan 2006 interactions from BioGRID TAB3 and write preprocessing reports |
| `src/go_loader.py` | Parse GO annotation files, taxonomy filters, and yeast ORF synonyms |
| `src/go_tfidf.py` | Compute GO TF-IDF functional signatures |
| `src/permanence.py` | Compute permanence and boundary-topology scores |
| `src/membership_overlap.py` | Compute functional dependency, membership, overlap addition, and transfer checks |
| `echo_ppi/evidence_profiles.py` | Build topology, GO richness, and uncertainty profiles |
| `echo_ppi/semantic_embeddings.py` | Build Sentence-BERT or TF-IDF/SVD semantic embeddings |
| `echo_ppi/candidate_generation.py` | Generate MCL, nucleus, semantic, and hybrid candidate modules |
| `echo_ppi/recall_safe_supplementation.py` | Apply size-gated, evidence-gated supplementation |
| `scripts/evaluate_echo_ppi_final.py` | Produce benchmark, label-validation, and auditability tables |
| `scripts/evaluate_krogan_benchmark.py` | Run Krogan 2006 second-dataset benchmark and auditability transfer analysis |
| `scripts/plot_echo_ppi_figures.py` | Regenerate manuscript figures |
| `scripts/plot_cross_dataset_figures.py` | Regenerate Gavin/Krogan cross-dataset figures |
| `scripts/reproduce_echo_ppi_final.sh` | Run the main reproducibility workflow |
| `tools/clusterone/cluster_one-1.0.jar` | Official ClusterONE 1.0 executable used for the exact Gavin baseline |

## Main scripts

| Script | Purpose |
|---|---|
| `scripts/build_evidence_profiles.py` | Build protein-level topology, GO, and text profiles |
| `scripts/build_semantic_embeddings.py` | Build semantic embeddings for protein profiles |
| `scripts/discover_black_hole_cores.py` | Rank evidence-potential nuclei |
| `scripts/generate_candidate_modules.py` | Generate candidate modules |
| `scripts/evaluate_echo_ppi_final.py` | Evaluate ECHO-PPI and named ablations |
| `scripts/evaluate_krogan_benchmark.py` | Evaluate the Krogan 2006 transferability benchmark |
| `scripts/plot_echo_ppi_figures.py` | Regenerate publication figures |
| `scripts/plot_cross_dataset_figures.py` | Regenerate cross-dataset benchmark and auditability figures |
| `scripts/reproduce_echo_ppi_final.sh` | Run evaluation, figures, and manuscript compilation |

## Main output files

| Output file | Status | Purpose | Main columns |
|---|---|---|---|
| `tables/table1_echo_ppi_final_benchmark.csv` | current | Main Gavin benchmark | `method`, `f1_mean`, `precision_mean`, `recall_mean`, `mean_size`, `coverage`, `runtime_sec`, `bundle_complete` |
| `tables/table2_echo_ppi_evidence_metrics.csv` | current | Evidence and auditability metrics | `metric`, `value` |
| `tables/table4_echo_ppi_heldout_benchmark.csv` | current | Held-out gold-complex splits | `method`, `f1_mean`, `f1_sd`, `precision_mean`, `recall_mean` |
| `tables/table5_echo_ppi_label_validation.csv` | current | Confidence-label validation | `confidence_label`, `n_assignments`, `membership_mean`, `best_gold_jaccard_mean`, `gold_supported_assignment_fraction` |
| `tables/table6_echo_ppi_auditability_comparison.csv` | current | Baseline auditability comparison | `method`, `f1`, `overlap_output`, `assignment_evidence`, `confidence_labels`, `bundle_complete` |
| `tables/table10_parameter_sensitivity.csv` | current | MCL, SLPA, and ECHO-PPI parameter sensitivity | `parameter`, `value`, `f1`, `precision`, `recall` |
| `results/evaluation/slpa_gavin_grid.csv` | current | SLPA threshold grid | `threshold`, `iterations`, `f1`, `precision`, `recall`, `mean_size`, `overlap_protein_fraction` |
| `results/evaluation/clusterone_status.csv` | current | Exact ClusterONE run status | `baseline`, `status`, `reason`, `jar`, `n_modules` |
| `results/baselines/clusterone_gavin_modules.csv` | current | Exact ClusterONE predicted modules | `community_id`, `protein_id` |
| `results/oracle_analysis.csv` | current | Per-gold candidate oracle analysis | `gold_complex_id`, `best_candidate_id`, `best_jaccard`, `category` |
| `results/significance_tests.csv` | current | Paired held-out Wilcoxon tests | `comparison`, `n_pairs`, `wilcoxon_statistic`, `p_value`, `note` |
| `results/runtime_cached_uncached.csv` | current | Cached and uncached embedding runtime | `method`, `cache_state`, `runtime_sec`, `embedding_backend` |
| `results/run_config.json` | current | Reproducibility manifest | dataset, seeds, embedding backend, ClusterONE status, SLPA grid |
| `results/communities/echo_ppi_refined_gavin.csv` | current | Final ECHO-PPI assignment records | `protein_id`, `community_id`, `membership_type`, `topology_score`, `semantic_score`, `go_score`, `membership_score` |
| `results/candidates/candidate_modules_gavin.csv` | current | Candidate-module memberships before final selection | candidate/module membership fields |
| `results/candidates/candidate_scores_gavin.csv` | current | Candidate evidence scores and selection features | candidate id, score, evidence features |
| `results/evidence_profiles/protein_profiles_gavin.csv` | current | Protein evidence profiles | `protein_id`, `weighted_degree`, `local_clustering`, `k_core_score`, `non_generic_go_terms`, `text_profile` |
| `results/krogan/benchmark_summary.csv` | current | Krogan benchmark summary | `method`, `f1`, `precision`, `recall`, `mean_size`, `overlap_protein_fraction`, `runtime_sec`, `evidence_bundle_completeness` |
| `results/krogan/auditability_summary.csv` | current | Krogan confidence-label and evidence support summary | `confidence_label`, `assignments`, `nonzero_evidence_fraction`, `multi_channel_fraction`, evidence means |
| `results/cross_dataset_auditability.csv` | current | Gavin/Krogan auditability transfer table | dataset, confidence label, evidence-channel fractions |

## Expected CSV files

Compatible pipeline variants may also emit the following schema-style files:

| Expected file | Purpose | Main columns |
|---|---|---|
| `clusters_initial_mcl.csv` | MCL seed assignments | `cluster_id`, `protein_id` |
| `go_term_importance.csv` | GO TF-IDF signatures | `cluster_id`, `go_term`, `tfidf_score` |
| `protein_membership.csv` | Permanence, functional dependency, and membership scores | protein id, cluster id, permanence, functional dependency, membership, intra/extra links |
| `overlap_summary.csv` | Protein-centric overlap counts | protein id, overlap count, module list |
| `evaluation_results.csv` | Benchmark and graph-quality metrics | F1, precision, recall, conductance, density, overlap metrics |

## Evidence-bundle schema

Each assignment-level evidence bundle should include:

| Field | Meaning |
|---|---|
| `protein_id` | Protein/locus identifier |
| `community_id` | Predicted module identifier |
| `membership_type` | Confidence label such as `core`, `inner`, `outer`, or `uncertain` |
| `topology_score` | Local graph/topology support for the assignment |
| `semantic_score` | Semantic-profile support |
| `go_score` | GO-based support |
| `membership_score` | Combined assignment score |
| `top_go_terms` | Non-generic GO terms supporting the module or assignment |
| `evidence_summary` | Human-readable explanation, when generated |
| `stability_freq` | Optional stability frequency across seeds or parameter settings |

Evidence-bundle completeness means that required documentation fields are present. It does not mean that every assignment is biologically correct.

## Interpreting confidence labels

| Label | Interpretation |
|---|---|
| `core` | Stronger topology/evidence support and highest-priority review candidates |
| `inner` | Supported assignment, often useful for overlap-aware review |
| `outer` | Boundary assignment with weaker support |
| `uncertain` | Low-support or hypothesis-only assignment that should not be treated as validated |

Labels are designed for triage. They do not replace experimental validation.

## Limitations

- ECHO-PPI currently trails exact ClusterONE on Gavin predictive F1 and does not outperform tuned MCL.
- Exact ClusterONE is included for Gavin; OSLOM, BIGCLAM, CFinder, and link communities still need full controlled benchmarking.
- SLPA is included as a reproducible sensitivity baseline rather than as a tuned main result.
- Krogan 2006 is now included as a second yeast benchmark. Broader STRING, full BioGRID, BioPlex, CORUM, and Complex Portal analyses still require dataset-specific preprocessing and identifier harmonisation.
- GO and semantic support depend on annotation coverage and quality.
- Case studies are curator-facing illustrations, not biological validation.

## Citation

If you use this repository before formal publication, cite the repository URL and the manuscript source:

```text
ECHO-PPI: Trustworthy AI for Evidence-Bundled Detection of Overlapping Protein Modules in Protein-Protein Interaction Networks.
https://github.com/MehrdadJalali-AI/ECHO-PPI
```

Replace this with the final journal citation and archival DOI when available.

## License / data-use notes

Check the repository licence before redistribution. Raw third-party biological datasets are not relicensed by this repository; obtain Gavin, GO, STRING, BioGRID, Complex Portal, CORUM, BioPlex, and other external resources from their original providers and follow their data-use terms.
