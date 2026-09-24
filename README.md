[README v2.md](https://github.com/user-attachments/files/32587242/README.v2.md)
# GC Column RI Coverage Tool

Paste in a list of volatile compound names and this tool checks NIST WebBook for retention index (RI) data on polar and non-polar GC stationary phases, then ranks which exact phases *and* phase **families** (Wax, FFAP, DB-5-type, DB-1-type, mid-polarity, etc.) give you the best literature coverage — so you can pick a GC column with actual evidence behind the choice, not just convention.

Runs entirely in the browser via Google Colab. No local install required.

## Before you run it: verify the code yourself

This tool makes live requests to external services, so it's worth checking before you run it — especially if you're pasting it into a shared or institutional environment.

- **View the notebook directly on GitHub**: (https://github.com/aspiringutilitybot/NIST-Searcher-for-gas-chromatography-volatile-retention-indices/blob/main/NIST%20searcher.ipynb) renders in your browser, no download needed.
- **Paste it into an LLM of your choice** (Claude, ChatGPT, or similar) and ask it to review the code for anything unsafe or unexpected.
- **What it actually contacts**, for full transparency:
  - **NIST WebBook** (`webbook.nist.gov`) — the actual retention index data
  - **PubChem** (`pubchem.ncbi.nlm.nih.gov`) — resolves compound names to CAS numbers, for more reliable NIST lookups
  - **A published Google Sheet** (only if `KNOWN_ISSUES_SHEET_URL` is set) — a small, human-curated list of compounds known to fail automated lookup, and where to check them manually
- It does not touch your files, credentials, or anything else on your system.

## Run it in Google Colab (no download required)

**[Open in Colab](https://colab.research.google.com/github/aspiringutilitybot/NIST-Searcher-for-gas-chromatography-volatile-retention-indices/blob/main/gc_column_ri_coverage_tool.ipynb)**

This link opens the exact file in this repository directly in Colab — the same code you just reviewed, not a separate copy.

First-time setup, once per session:
1. **Runtime → Change runtime type → R**, if it doesn't launch in R automatically.
2. Run the install cell (`install.packages(c("webchem", "dplyr"))`) — takes 1-2 minutes. Colab doesn't save packages between sessions, so this repeats each time you return.
3. Edit the compound list and RI type selection (see below), then run the rest of the notebook top to bottom.

## How to use it

- **Paste your compound names** into `MY_COMPOUNDS_RAW`, one per line. No commas between compounds — the script splits on newlines only (commas *within* a compound name, like `(E,E)-2,4-decadienal`, are fine).
- **Choose your RI type(s)** via `RI_TYPES`. NIST reports RI under up to four calculation methods, which are *not* interchangeable:
  - `kovats` — isothermal runs
  - `linear` — Van den Dool & Kratz, for temperature-programmed runs
  - `alkane` — Normal Alkane RI
  - `lee` — PAH-specific
  
  The default fetches all four (safest, ~4x slower). Narrow this to just what your own method uses for a faster run.
- **Optionally set `KNOWN_ISSUES_SHEET_URL`** to cross-check failed compounds against the known-issues sheet (see below).

## Outputs

Written into Colab's temporary storage (download via the folder icon in the sidebar before your session ends):

**`gc_ri_coverage_results.xlsx`** - every table below as a separate tab in one workbook. Probably all you need to download.

The same data is also written as individual CSVs, if you need just one table on its own:

| File | Contents |
|---|---|
| `nist_ri_nonpolar_raw.csv` / `nist_ri_polar_raw.csv` | Every literature RI record retrieved, tagged by RI type and phase family |
| `coverage_by_phase_nonpolar.csv` / `_polar.csv` | Ranking of exact GC phases by how many of your compounds they cover |
| `coverage_by_family_nonpolar.csv` / `_polar.csv` | Same ranking, rolled up to phase family |
| `ri_stats_by_phase_nonpolar.csv` / `_polar.csv` | Count / mean / SD / min / max RI per compound + phase + RI type |
| `ri_stats_by_family_nonpolar.csv` / `_polar.csv` | Same, rolled up to phase family |
| `failed_compounds_lookup.csv` | Any compound with no retrievable RI data, cross-checked against the known-issues sheet |

## Known-issues pointer sheet

A small number of compounds can fail automated NIST lookup — for example, if a compound has duplicate entries in NIST's database under the same name, an automated search can land on a disambiguation page instead of the actual data. (Limonene and alpha-pinene were early cases of this, but resolving compound names to CAS numbers via PubChem first — see Known limitations below — fixed lookup for those two, so they're no longer expected to fail.) The sheet is currently empty; it exists so that if and when a compound does fail, it gets flagged here rather than silently returning no data. Since this is a moderated, community-maintained list:

- **Found a failing compound?** [Submit it here](https://docs.google.com/forms/d/e/1FAIpQLSdfW17fDu-OfuK3k1Lv7HNXF2C8Ee-mxKUV0XRF9ByCyIb1XA/viewform?usp=publish-editor) after manually confirming it on NIST WebBook.
- The sheet stores **pointers only — compound name, CAS number, and a direct NIST URL — never actual RI data**. NIST's Standard Reference Data has its own copyright terms restricting reproduction of the underlying data itself, so nothing from NIST's database is copied or redistributed through this mechanism.

## Known limitations

- Name-based NIST lookup is the least reliable match type. This tool resolves compound names to CAS numbers via PubChem first (far more reliable), falling back to name-based search only if that fails.
- The phase-family lookup table is curated from manufacturer documentation but isn't exhaustive. Unrecognized phases are labeled `UNCLASSIFIED` rather than guessed. Currently unresolved: `LM-120`, `CBP-1`, `CBP-5`, `MS5`, `SBP-5` — if you can find reliable documentation for these, contributions welcome.
- RI coverage reflects where **literature data exists**, not which column will separate your specific sample matrix best. Use this to narrow candidates, not to skip method development entirely.
