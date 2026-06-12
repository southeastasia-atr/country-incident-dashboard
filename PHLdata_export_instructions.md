# PHLdata.csv Export Instructions

Use these instructions whenever you need to regenerate `PHLdata.csv` from `PHL_MasterDataset.xlsx` for the dashboard. Upload both files to Claude and ask it to run the export.

---

## What the export script does

- Reads the `Incidents` sheet from `PHL_MasterDataset.xlsx` using `openpyxl` in `read_only=True, data_only=True` mode
- Outputs exactly the 44 columns the dashboard expects, in the correct order
- Derives `Actor Categories` by parsing the `Actors` field
- Takes `Arms Score` directly from column AP (the cached formula value) — **do not recompute in Python**
- Formats dates as `DD Month YYYY` (e.g. `07 February 2026`) via `.strftime('%d %B %Y')`
- Normalises Tags delimiters to comma (handles `\n` and `,` in source)
- Writes UTF-8 with BOM (`utf-8-sig`) for safe handling in both Excel and the dashboard
- Excludes all trailing empty columns (43–61) from the XLSX

---

## Output column order

```
Report No., Code, Incident Type, Title, Date, Location, Barangay, Municipality,
Province, Latitude, Longitude, Description, Weapons, Actors, Actor Categories,
Unspecified, Pistol, Revolver, Assault Rifle, Rifle, Shotgun, SMG, LMG, HMG,
GL, AGL, RPG/ATW, Mortar, ATGM, MANPADS, Hand Grenade, Projected Gren, Rifle Gren,
Munition, Ammo, Craft handgun, Craft Rifle, Craft Shotgun, Craft SMG, Craft RPG,
Tags, Convergence, Arms Score, Source
```

---

## Pre-export checklist (do in Excel before uploading)

- [ ] Verify the Arms Score formula in column AP is correct for all rows — especially after any column insertions, which can shift formula references
- [ ] Resave the file in Excel (not openpyxl) so formula cached values are up to date; `data_only=True` reads cached values only
- [ ] Resolve any duplicate `Code` values (each code must be unique)
- [ ] Review any rows with `Arms Score = 0` to confirm they are intentional (e.g. manufacturing site disruptions with no recovered weapons)

---

## Known data characteristics (not errors)

- **`Report No.`** is a source reference (1–15), not a unique row ID. The dashboard uses `Code` as the incident identifier.
- **24 rows have no `Municipality`** — these are multi-location incidents where a single municipality cannot be assigned. This is expected and does not break the dashboard.
- **Two zero-score rows** (PHL.26.02.28.004, PHL.26.03.05.002) — intentional, both are illicit manufacture incidents with no recovered weapons counted.
- **`Actor Categories`** does not exist as a column in the XLSX; it is derived during export by parsing `Actors`.

---

## Critical warning: openpyxl and formula columns

If `Arms Score` values come back as `None` after export, it means the XLSX was last saved by openpyxl (which strips cached formula values on write). The fix is to open the file in Excel and resave it before re-running the export. Never use openpyxl's write mode on this file as part of the export pipeline.

---

## After export

1. Replace `PHLdata.csv` in the repo with the new file
2. Push to `origin main`
3. Verify the live dashboard at https://southeastasia-atr.github.io/phl-incident-dashboard/

```bash
cd C:\path\to\phl-incident-dashboard
git add PHLdata.csv
git commit -m "Update PHLdata.csv from master dataset"
git push origin main
```
