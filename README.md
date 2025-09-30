## UMOJA
**Unified Microbial Omics for Joint AMR Analysis — Bloodstream Infection (BSI) pipeline**

Umoja is a standards-driven, offline-capable Snakemake workflow that harmonizes best-in-class bacterial WGS tools
to produce reproducible BSI genomics outputs across labs and countries. 

The design of the pipeline has tried to consider low resource settings where computational resources may be minimal

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
2.	Read QC (Core) — Run quality checks and trimming ([FastQC](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/) and [fastp](https://github.com/OpenGene/fastp)
3.	Contamination check (Core) — Detect within-species contamination before assembly with [ConFindr](https://github.com/OLC-Bioinformatics/ConFindr).  
4.	Taxonomic ID (Core) — Classify reads using [Kraken2](https://ccb.jhu.edu/software/kraken2/index.shtml) and offline local database [MiniKraken2_v2_8GB](https://ccb.jhu.edu/software/kraken2/index.shtml?t=downloads)
5.  Species Consensus Check (optional addon, recommended) - Use [BactInspector](https://gitlab.com/antunderwood/bactinspector) to confirm species; optionally require concordance with Kraken2 before proceeding to organism-specific modules.
6.	De novo Assembly (Core)— Assemble with [Shovill](https://github.com/tseemann/shovill) (using [SKESA](https://github.com/ncbi/SKESA) as backend by default) and produce contigs. 
7.	Assembly QC (Core)— Evaluate contigs with [QUAST](https://quast.sourceforge.net/quast). If QUAST fails for some samples then re-run De novo assembly using [SPAdes](https://ablab.github.io/spades/index.html) to compare, and then redo QUAST.
8.	Typing and Virulence — Assign sequence type with [MLST](https://github.com/tseemann/mlst) and detect virulence genes (Virulome) using [AMRFinderPlus](https://github.com/ncbi/amr/wiki/Running-AMRFinderPlus) - optionally use [VirulenceFinder](https://bitbucket.org/genomicepidemiology/virulencefinder/src/master/) for in-depth virulence gene detection for specific pathogens such as _E. coli, E. faecalis, E. faecium, S aureus and L. monocytogenes_.
9.	AMR profiling — Detect AMR genes/point mutations (Resistome) with [AMRFinderPlus](https://github.com/ncbi/amr/wiki/Running-AMRFinderPlus) (default) and perform (optional) Targeted cross-check for _E. coli/Salmonella_ with ResFinder, PlasmidFinder, and PointFinder (using [staramr](https://github.com/phac-nml/staramr)). Additionally, known and novel variants in anti-microbial resistance genes, can be predicted from clean reads using the Comprehensive Antibiotic Resistance Database [CARD](https://card.mcmaster.ca/). Note that CARD is also optional due to being computationaly heavy/requiring more resources.
10.	Species-specific modules - if species are known,
    - Run [Kleborate](https://github.com/klebgenomics/Kleborate) and also consider also running [Kaptive](https://github.com/klebgenomics/Kaptive) for additional vaccine-development focused insights (for _K. pneumoniae_)
    - Run [SISTR](https://github.com/phac-nml/sistr_cmd) (for _Salmonella enterica_)
    - Run [ECTyper](https://github.com/phac-nml/ecoli_serotyping) (for _Escherichia coli_)
    - Run [SeroBA](https://github.com/sanger-pathogens/seroba) (for _Streptococcus pneumoniae_)
    - Run both [SCCmec typing](https://github.com/rpetit3/sccmec) (infection-control and therapy decisions) and [spaTyper](https://github.com/HCGB-IGTP/spaTyper) (subtyping and cluster confirmation) - for MRSA
12.	Reporting (Core)— Aggregate results with MultiQC and export a clinician-friendly Excel (pandas/openpyxl) plus TSV/JSON. 

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
