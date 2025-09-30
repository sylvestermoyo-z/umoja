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

