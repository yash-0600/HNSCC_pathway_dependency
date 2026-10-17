# HNSCC pathway dependency modeling

A research notebook for modeling pathway dependencies in head and neck squamous cell carcinoma (HNSCC). The analysis builds pathway-level drug-response targets from GDSC, relates them to expression-derived pathway activity, and applies trained models to TCGA HNSC, GSE65858, and GSE103322 single-cell data. Downstream sections examine co-dependency structure, clinical and survival associations, mutation/CNV/protein validation, and main and supplementary figures and tables.

**Reproducibility status:** several required datasets are missing, dependencies are unpinned, and the notebook has known execution-order and namespace problems. A fresh top-to-bottom run is currently blocked. The repository layout and file paths have been checked, but the full scientific analysis has not been reproduced.

## Layout

```text
.
├── analysis_notebook.ipynb     # Main analysis; launch from repository root
├── requirements.txt           # Unpinned Python dependencies
├── README.md
├── data/
│   ├── raw/                   # Original inputs with notebook-compatible names
│   ├── processed/             # Externally prepared inputs; not produced here
│   └── reference/
│       └── duplicates/        # Preserved duplicate source files
└── results/                   # Created locally by the notebook; not uploaded
    ├── Figures_Main/
    └── Figures_Supplementary/
```

All generated CSVs, pickles/models, text summaries, PNGs, and workbooks belong under `results/`, including generated GDSC matrices and TCGA/GSE pathway scores and predictions. `data/processed/` is reserved for the five externally prepared inputs listed below, not notebook outputs. The supplied `data/` directory is versioned; generated results, environments, and checkpoints are excluded from Git.

## Setup and launch

### Download the repository and datasets

Install [Git LFS](https://git-lfs.com/) before cloning (on macOS with Homebrew: `brew install git-lfs`), then run:

```bash
git lfs install
git clone https://github.com/yash-0600/HNSCC_pathway_dependency.git
cd HNSCC_pathway_dependency
git lfs pull
```

The supplied data occupy approximately 602 MB. `data/raw/GSE103322_data.txt` and `data/raw/TCGA HNSC HiSeqV2.txt` are stored in Git LFS; the remaining data files use ordinary Git. Prefer cloning and `git lfs pull` over GitHub's **Download ZIP**: repository-generated archives may contain LFS pointers instead of the actual matrices. A file beginning with `version https://git-lfs.github.com/spec/v1` is a pointer, not usable expression data. LFS downloads depend on the repository owner's GitHub storage/bandwidth allowance.

### Install Python dependencies

Python **3.11 is a recommended setup target**, not a tested compatibility guarantee. From the repository root, create a new environment rather than copying the existing local `jupyter_env/`:

```bash
python3.11 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python -m notebook analysis_notebook.ipynb
```

On Windows, use `py -3.11 -m venv .venv` and activate with `.venv\Scripts\activate` (Command Prompt) or `.venv\Scripts\Activate.ps1` (PowerShell), then use the same `python -m pip install -r requirements.txt` and launch commands. Select the environment's Python kernel in Jupyter.

Run from the root: notebook paths are root-relative. The existing `jupyter_env/` and `.ipynb_checkpoints/` are retained locally, not portable environment specifications, and are excluded from publication. Requirements are **unpinned**; dependency resolution and compatibility with the APIs used in the notebook have not been verified. Installing requirements alone does not resolve the missing inputs or the blockers below.

## Input inventory

“Available” describes the supplied source files included in this repository's `data/` directory; the two LFS-managed matrices require `git lfs pull`. These files are intentionally absent from the code-only ZIP example below. Obtain missing files from their legitimate sources and respect access/redistribution terms. Do not substitute similarly named files without checking cohort, release, identifiers, orientation, and preprocessing.

### Raw inputs (`data/raw/`)

| Canonical filename | Local status / role |
| --- | --- |
| `GDSC1_fitted_dose_response_15Oct19.xlsx` | Available; GDSC1 dose-response source, compatibility filename only (see provenance) |
| `GDSC2_fitted_dose_response_15Oct19.xlsx` | Available; GDSC2 dose-response source, compatibility filename only |
| `screened_compounds_rel_8.1.csv` | Available; compound annotations originally labeled v8.5 |
| `Human MSigDB Hs Symbols v2026.1.gmt` | Available; pathway gene sets |
| `RNA_Seq_All_TPM.csv` | **Missing**; training expression input |
| `phenotype.txt` | Available; TCGA phenotype/clinical annotations |
| `HNSC_mc3_gene_level.txt` | Available; gene-level mutation data |
| `HNSC_mc3.txt` | **Missing**; separate mutation input, not replaced by the gene-level file |
| `Gistic2_CopyNumber_Gistic2_all_thresholded.by_genes` | **Missing**; copy-number input |
| `RPPA_RBN.txt` | **Missing**; protein/RPPA input |
| `GSE103322_data.txt` | Available; single-cell matrix with five leading metadata rows |
| `GSE65858_expression_genes.csv` | **Missing**; external-cohort expression |
| `GSE65858_family.soft.gz` | **Missing**; GEO metadata used to derive HPV annotations |
| `TCGA HNSC HiSeqV2.txt` | Available raw source candidate; **not automatically converted** to the prepared expression input |
| `HNSC_survival.txt` | Available raw source candidate; **not automatically converted** to prepared survival data |

### Externally prepared inputs (`data/processed/`)

All five files below are **missing locally** and have no notebook producer. Preparing them requires explicit, scientifically appropriate preprocessing; renaming the raw TCGA sources is not a conversion.

| Filename | Required organization / fields |
| --- | --- |
| `tcga_expression_ready_for_gsva.csv` | Genes × samples; gene identifiers as row index and sample identifiers as columns |
| `tcga_50pathway_scores.csv` | Samples × the 50 training pathways; row identifiers and pathway labels must align with model inputs |
| `tcga_survival_ready_for_pfi.csv` | Sample-indexed survival table with `PFI` and `PFI.time` for PFI analyses |
| `gse65858_survival.csv` | Sample-indexed survival table with `OS` and `OS.time` for OS analyses |
| `igf1r_repeated_cv.csv` | Repeated cross-validation results with `Pathway`, `R2`, `Spearman`, and `Seed` columns |

Sample identifiers must align across expression, pathway, phenotype, prediction, and survival tables. Event coding, time units, normalization, gene mapping, and cohort inclusion must be established from the original data/preprocessing documentation; this cleanup does not invent those choices. Survival sections require the endpoint-specific event/time pair (`PFI`/`PFI.time` or `OS`/`OS.time`).

## Provenance and compatibility filenames

**The canonical names containing `15Oct19` and `rel_8.1` are compatibility names expected by the notebook, not verified release labels for the supplied data.** The supplied workbooks were named “Oct 27” and the annotation file was labeled v8.5. Renaming does not establish equivalence to a 2019 GDSC release or compound release 8.1. Release differences can affect scientific results; reproducing a specific analysis requires verifying the intended releases independently.

| Original supplied filename | Organized destination | Evidence / caveat |
| --- | --- | --- |
| `Cell Model Passports Dose Response Oct 27 (1).xlsx` | `data/raw/GDSC1_fitted_dose_response_15Oct19.xlsx` | First `DATASET` value is GDSC1; original release/date label retained here |
| `Cell Model Passports Dose Response Oct 27.xlsx` | `data/raw/GDSC2_fitted_dose_response_15Oct19.xlsx` | First `DATASET` value is GDSC2; not evidence of a 2019 release |
| `Screened Compounds v8.5.csv` | `data/raw/screened_compounds_rel_8.1.csv` | Supplied v8.5 label; not a conversion to v8.1 |
| `Single cell RNA-seq analysis of head and neck cancer.txt` | `data/raw/GSE103322_data.txt` | Five metadata rows match the notebook's expected layout |
| `HNSC Gene Level Data.txt` | `data/raw/HNSC_mc3_gene_level.txt` | Gene-level input, not the separate missing `HNSC_mc3.txt` |
| `Human MSigDB Hs Symbols v2026.1.gmt` | `data/raw/Human MSigDB Hs Symbols v2026.1.gmt` | Name retained |
| `phenotype.txt` | `data/raw/phenotype.txt` | Name retained |
| `TCGA HNSC HiSeqV2.txt` | `data/raw/TCGA HNSC HiSeqV2.txt` | Original retained, not renamed to a prepared CSV |
| `HNSC_survival.txt` | `data/raw/HNSC_survival.txt` | Original retained, no automatic survival preparation |
| `Gene Identifiers Dec 12 2024.csv` | `data/reference/Gene Identifiers Dec 12 2024.csv` | Reference annotation |
| `Cell Model Passports July 2026.csv` | `data/reference/Cell Model Passports July 2026.csv` | Reference annotation |
| `Human MSigDB Hs Symbols v2026.1.pdf` | `data/reference/Human MSigDB Hs Symbols v2026.1.pdf` | Reference document |
| `Cell Model Passports July 2026 (1).csv` | `data/reference/duplicates/Cell Model Passports July 2026 (1).csv` | Exact duplicate preserved, not deleted |
| `TCGA.HNSC.sampleMap-HNSC_clinicalMatrix.txt` | `data/reference/duplicates/TCGA.HNSC.sampleMap-HNSC_clinicalMatrix.txt` | Duplicate phenotype source preserved |
| `mc3.v0.2.8.PUBLIC.maf.csv` | `data/reference/mc3.v0.2.8.PUBLIC.maf.csv` | Tab-separated despite `.csv`; not the missing HNSC mutation input |

The supplied `mc3.v0.2.8.PUBLIC.maf.csv` was inspected across 49 rows, including a malformed row. Its only populated tumor barcode was `TCGA-02-0003-01A-01D-1490-08`, with no overlap with the HNSC phenotype data. It must not be represented as a valid replacement for `HNSC_mc3.txt`.

## Outputs and execution dependencies

Outputs are **generated artifacts, not missing external prerequisites**. Examples under `results/` include:

- Training targets/activity: `gdsc_full_pathway_z.csv`, `gdsc_full_dependency_scores.csv`, `gdsc_full_dependency_scores_rawLN.csv`, `gdsc_full_50pathway_scores.csv`, and `gdsc_full_dependency_scores_aligned.csv`.
- Models and validation: `full_gdsc_models_z.pkl`, `cv_r2_leakage_free.csv`, `nested_lopo_results.csv`, `oof_predictions.csv`, `calibration_summary.csv`, and pathway-specific bootstrap coefficient/statistic files.
- Cohort outputs: `tcga_dependency_predictions_fullGDSC.csv`, `gse65858_gsva_scores.csv`, `gse65858_predictions.csv`, and `gse65858_hpv.csv`.
- Single-cell outputs: `expr_malignant.pkl`, `meta_malignant.pkl`, `pred_sc_GSE103322.pkl`, `X_hallmark_GSE103322.pkl`, and pseudobulk/co-dependency summaries.
- Clinical, mediation, mutation, CNV, RPPA, survival, correlation/network, and sensitivity-analysis CSVs and PNGs.
- Six main figure assemblies (PNG and PDF) in `results/Figures_Main/` and the supplementary workbook `results/Supplementary_Tables_S1-S23.xlsx`. The notebook also creates `results/Figures_Supplementary/`, but currently does not export figures into that directory.

The supplementary-table consumers for S14/S15 use the actual producer filenames `phenotype_associations_all_pathways_raw_noalcohol.csv` and `treatment_effects_results_raw_noalcohol.csv`. Some analyses conditionally create outputs; an absent table can mean a prerequisite or conditional analysis was not satisfied, not that an external dataset must be downloaded. Do not publish model pickle files from untrusted sources or load untrusted pickles.

`gdsc_full_dependency_scores_rawLN.csv` is an optional backup name: the notebook moves an existing dependency-score file there before overwriting it. The backup operation does not recalculate raw-LN scores or verify the old file's normalization.

### Known blockers retained by this cleanup

The following refer to **1-based notebook cell positions**, including markdown cells, rather than execution counters:

- Cell 12 loads `full_gdsc_models_z.pkl`, but its producer is cell 14.
- TCGA predictions are first read in cell 28 and generated in cell 114.
- GSE65858 predictions are optionally read in cell 40 and required in cell 52, but generated in cell 106.
- Cell 44 shadows the `gsva` function with a dataframe, breaking later enrichment calls in the shared namespace.
- Cell 92 references undefined `bulk_dep`.
- Cell 120 uses `rankdata`, `norm`, and `fftconvolve` without the required names being imported.
- Cell 146 uses the undefined `mpl` alias.

These issues are **not fixed by path/layout cleanup**, and a simple “Run All” recommendation would be misleading. Resolve input provenance/preparation and these execution dependencies before claiming reproducibility; reordering alone does not fix namespace problems. No scientific method changes or automated raw-to-prepared TCGA conversions are implied.

The single-cell source is approximately **397 MB**. Pandas matrices, model fitting, resampling, nested validation, and long bootstrap analyses can require substantial memory and runtime beyond the on-disk input sizes. No hardware-specific runtime or memory guarantee is provided.

### Verified repository checks

The layout migration was checked by compiling all 74 code cells, comparing the analytical code structure after accounting for path/setup changes, and confirming SHA-256 preservation of all 15 relocated data/reference files. The project-root setup was executed in both valid and invalid working directories, and a local Jupyter server successfully served the renamed 148-cell notebook. Git ignore behavior and the code-only ZIP command below were exercised in temporary locations.

These are packaging and path-resolution checks, not an end-to-end analysis run, a fresh requirements installation, or validation of the statistical results. Temporary verification files and the smoke server were removed/stopped.

## GitHub and archive publication

The repository publishes the three main analysis files and `data/`, plus `.gitattributes` and `.gitignore` for large-file handling and safe exclusions. The `.gitignore` excludes generated results, environments (`jupyter_env/`, `.venv/`, etc.), checkpoints, Python caches, and macOS metadata; it does **not** exclude source data. Ignore rules do not remove files already tracked in another Git history. Notebook cell outputs can also contain data or sensitive identifiers: review them before sharing.

GitHub blocks ordinary Git files larger than **100 MiB**; the single-cell input exceeds that limit. This repository configures Git LFS for the single-cell and bulk TCGA expression matrices in `.gitattributes`. Contributors must install Git LFS before adding or changing those files; pushing through Git LFS uploads their actual contents separately from Git pointer records. Verify redistribution rights, dataset licenses, consent/access restrictions, and clinical-data privacy obligations before publishing additional data. A filename or local possession is not permission to redistribute.

### ZIP examples (macOS/Linux)

Run these from the repository root. Archives are written to the parent directory so they cannot include themselves. These examples preserve the local environment on disk while excluding it from archives. ZIP exclusion patterns must remain quoted.

**Code-only ZIP** (no datasets or results):

```bash
zip -r ../hnscc-code-only.zip . \
  -x 'jupyter_env/*' '.venv/*' 'venv/*' 'env/*' \
     '.git/*' '*/.git/*' '.ipynb_checkpoints/*' '*/.ipynb_checkpoints/*' \
     '.DS_Store' '*/.DS_Store' '__pycache__/*' '*/__pycache__/*' '*.pyc' \
     '.env' '.env.*' 'data/*' 'results/*'
```

**Data-inclusive ZIP** (only when sharing is permitted; still excludes environments and repository internals):

```bash
zip -r ../hnscc-with-data.zip . \
  -x 'jupyter_env/*' '.venv/*' 'venv/*' 'env/*' \
     '.git/*' '*/.git/*' '.ipynb_checkpoints/*' '*/.ipynb_checkpoints/*' \
     '.DS_Store' '*/.DS_Store' '__pycache__/*' '*/__pycache__/*' '*.pyc' \
     '.env' '.env.*'
```

The data-inclusive example includes whatever actually exists in `data/` and `results/`, including reference duplicates; it cannot supply missing inputs and may be large. Run `git lfs pull` before creating a data-inclusive ZIP from a fresh clone so that the two matrices contain actual data rather than LFS pointers. ZIP does not apply `.gitignore`, which is why exclusions are explicit. Review archive contents and any additional local secrets before distributing.

## Citation and licensing

No verified article citation, DOI, or software license is supplied here. Cite the verified publication associated with the analysis if one is provided by the project authors; otherwise identify the notebook/version or commit you used and describe the analysis accurately. Also cite the original GDSC, TCGA, GEO, MSigDB, and other data/tool sources according to their own citation requirements and the releases actually used. Do not infer publication metadata or redistribution permissions from this README. Obtain clarification from the authors before reuse that requires a software license.
