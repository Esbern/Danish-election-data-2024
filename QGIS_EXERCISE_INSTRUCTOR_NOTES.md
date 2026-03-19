# QGIS Student Exercise — Data Package Summary

## What to Give Students

1. **afstemningsomraader_2024.geojson** — Voting area boundaries (1,298 areas, ~95 MB)
2. **ejerforhold_shares_by_valgomraade.csv** — Housing data with:
   - `ValgstedId` (join key)
   - `KommuneNavn`, `KommuneKode` (municipality ID & name)
   - `RegionNavn` (region)
   - `ShareAlmennyttig` (non-profit housing %)
   - `muni_mean` (your municipality's average share)
   - `muni_std` (your municipality's standard deviation)
3. **QGIS_EXERCISE_Almennyttig_Heterogeneity.md** — Step-by-step instructions

## What Students Will Produce

### Deliverable 1: Two Maps
- **Map A**: Municipality means (external variation view)
- **Map B**: Voting area deviations (internal variation view)

### Deliverable 2: Variance Summary Table
| Metric | Value |
|--------|-------|
| Between-municipality variance (%) | __ |
| Within-municipality variance (%) | __ |

### Deliverable 3: Reflection (1–2 pages)
- Describe what the maps show
- Interpret the variance split
- Answer the challenge questions

---

## Instructor Notes

**Duration:** 2–3 hours (one full lab session)

**Skills taught:**
- Vector layer operations (join, dissolve)
- Symbolic styling (graduated, diverging ramps)
- Field calculations
- Grouped statistics
- Variance decomposition concept

**No Python needed.** All calculations are done via QGIS GUI + spreadsheet.

**Data size:** GeoJSON is ~95 MB; CSV is <200 KB. Total ~95 MB download.

**Assumptions:** Students have QGIS 3.20+ and basic familiarity with layers/attribute tables.

---

## Quick Teacher Walkthrough (if you help them)

1. Students load GeoJSON: 1,298 areas appear
2. Join CSV: Now they see Almennyttig data on each area
3. Calculate `Deviation` field: Instantly shows local variation
4. Style by deviation: Visual story emerges (red/blue map)
5. Dissolve to municipalities: 98 municipality polygons
6. Run Statistics by Categories: Summary table
7. Spreadsheet calculation: Between/within split
8. Maps + table + reflection = complete exercise

The beauty is that they *see* the spatial heterogeneity immediately (the red/blue map) before they compute the variance numbers, so the numbers mean something.

---

## Extensions (if time allows)

- Compare this with other housing types (Andelsbolig, single-family, etc.)
- Add a choropleth of population density to see if heterogeneity correlates with urban/rural differences
- Interactive web map creation (QGIS to QGIS Cloud export)
