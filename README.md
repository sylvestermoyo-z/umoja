## UMOJA
**Unified Microbial Omics for Joint AMR Analysis — Bloodstream Infection (BSI) pipeline**

Umoja is a standards-driven, offline-capable Snakemake workflow that harmonizes best-in-class bacterial WGS tools
to produce reproducible BSI genomics outputs across labs and countries.

## Why UMOJA?
- **BSI-focused:** routes common BSI pathogens through organism-aware modules.
- **Joint AMR analysis:** harmonizes QC, databases, and reporting **across labs, pathogens, tools, and time**.
- **Offline-first:** per-rule conda envs you can pre-fetch and reuse without internet.
- **Reproducible:** each release ships tool/DB manifests and checksums.

## Quick start
```bash
# 1) one-time: create envs while online (caches packages locally)
for y in workflow/envs/*.yml; do conda env create -f "$y" || conda env update -f "$y"; done

# 2) edit DB paths & samples
nano workflow/configs/config.yaml

# 3) preflight check (tools + DBs)
bash bin/umoja-preflight.sh

# 4) dry run, then execute
snakemake -n
snakemake --use-conda --cores 8
```
### Reproducible, offline-ready setup
```bash
# 1) Freeze exact versions into lock files (commit them)
make lock

# 2) Build envs from locks to warm the local conda cache (online once)
make envs

# 3) (Optional) Pre-create Snakemake's hashed rule envs
make prebuild

# 4) Verify tools & DB paths
make preflight

# 5) Dry-run and then execute the smoke test
make smoke
```
_Offline tip:_ after step 2, you can disconnect and run: "snakemake --use-conda" offline. 

---
## Process flow (High Level)
0.  Pre-run checks Core)- Verify tools + DB versions; confirm paths (Kraken2 DB, ConFindr DB, etc.)
1.	Inputs & config (Core) — Organize paired FASTQs and set DB/tool paths in "workflow/configs/config.yaml"
2.	Read QC (Core) — Run quality checks and trimming ([FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/); [fastp](https://github.com/OpenGene/fastp))
3.	Contamination check (Core) — Detect within-species contamination ([ConFindr](https://github.com/OLC-Bioinformatics/ConFindr)). Ths step flags intra/inter-species contamination before assembly. 
4.	Taxonomic ID (Core) — Classify reads using ([Kraken2](https://ccb.jhu.edu/software/kraken2/index.shtml) and offline local database [MiniKraken2_v2_8GB](https://ccb.jhu.edu/software/kraken2/index.shtml?t=downloads)
5.	Assembly (Core)— Assemble with Shovill (using SKESA backend by default) and produce contigs. Anaconda+3GitHub+3bioconda.github.io+3
6.	Assembly QC — Evaluate contigs with QUAST. quast.sourceforge.net+2quast.sourceforge.net+2
7.	Typing — Assign sequence type with MLST. GitHub
8.	AMR calling — Detect AMR genes/point mutations with AMRFinderPlus and/or ResFinder/PointFinder (if you plan to include them). GitHub+4NCBI+4bioconda.github.io+4
9.	(Optional) Virulence — Run virulence panels (e.g., VFDB) when you’re ready; keep this step disabled for now.
10.	Reporting — Aggregate results with MultiQC and export a clinician-friendly Excel (pandas/openpyxl) plus TSV/JSON. docs.seqera.io+2docs.seqera.io+2

_Tip:_ keep each step runnable on its own (Snakemake rules), so you can test/debug “bit-by-bit”

---
## Inputs / Outputs

_Inputs:_ Paired-end FASTQs; (optional) phenotype/AST CSV. to help flag genotype–phenotype discrepancies (e.g., carbapenemase gene present but Meropenem (MEM) reported S).

_Outputs:_ Per-step artifacts under results/<sample>/… and a consolidated Excel + HTML in results/summary/

---
## Reproducibility & offline
- Per-rule conda envs (workflow/envs/…) + local conda cache = internet-free re-runs
- DB paths set in "workflow/configs/config.yaml"
- Ship DB manifests in db_manifests/

---
## Citation

Moyo S.Z. et al. (2025) UMOJA: Unified Microbial Omics for Joint AMR Analysis in BSIs. Github https://github.com/sylvestermoyo-z/umoja
