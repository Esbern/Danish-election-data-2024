# Reorganized Danish Election Data Documentation

## Overview

Your election data from the Danish Elections Database (dst.dk) has been reorganized from 5 messy CSV files into a clean, normalized structure with:

- **1 Reference File**: `valgsteds.csv` - Master list of voting areas
- **33 Topic-Specific Files**: One CSV file per data topic, with EV2024 prefixes removed
- **All files linked by ValgstedId**: Proper database-like relationships

## Data Structure

## Election District Geometry

The spatial files represent **election districts (afstemningsomrader)** intended to link with the tabular election attributes.

### Geometry files in this folder

- `valgsteds_2024.geojson`: Full geometry export
- `valgsteds_2024.topo.json`: Full-resolution TopoJSON
- `valgsteds_2024.simplified.topo.json`: Simplified TopoJSON for web maps
- `valgsteds_2024_matched.geojson`: Geometry filtered to keys found in `valgsteds.csv`
- `valgsteds_2024_matched.topo.json`: TopoJSON version of matched geometry
- `valgsteds_2024_matched.simplified.topo.json`: Simplified matched TopoJSON for web maps

### Recommended default for users

If your goal is easiest use with the attribute tables, start with:

- `valgsteds_2024_matched.geojson` for GIS workflows and reproducible joins
- `valgsteds_2024_matched.simplified.topo.json` for web maps and lightweight sharing

Why this is easiest:

- These files are already filtered to election districts that match tabular IDs
- They reduce join errors caused by extra geometry records
- They are much smaller than full exports

Projection note:

- GeoJSON is WGS84 longitude/latitude (EPSG:4326)
- TopoJSON may prompt for CRS in GIS software; if prompted, choose EPSG:4326 for these exports

### 30-second quick start

1. Open `valgsteds_2024_matched.geojson` in QGIS.
2. Import any attribute CSV (for example `Sociokonomisk_status_og_brancher_fordelt_p_afst___.csv`).
3. Join the CSV to geometry:
   - Geometry key: `kms_stat_id`
   - CSV key: `ValgstedId`
4. Use `valgsteds_2024_matched.simplified.topo.json` when you need a small web map file.

If QGIS asks for CRS on TopoJSON, choose EPSG:4326.

### Join key for attributes

Use the following key mapping when joining geometry to attributes:

- Geometry field: `kms_stat_id`
- Attribute field: `ValgstedId` (in `valgsteds.csv` and all topic files)

Current overlap check:

- 1,298 unique `ValgstedId` values in attributes
- 1,314 unique `kms_stat_id` values in full geometry
- 1,272 keys overlap directly

This means some districts are present only in one source. Use the `valgsteds_2024_matched.*` files when you need geometry that is guaranteed to match attribute keys.

### Node.js workflow (already configured)

This folder now contains a local Node setup (`package.json`) with conversion scripts.

Run from this folder:

```bash
npm run topo
npm run topo:simplified
npm run topo:matched
npm run topo:matched:simplified
```

### Geometry attribution

Geometry source is Klimadatastyrelsen (CC-BY-4.0). Keep attribution and avoid use of official logos.

### Reference File: `valgsteds.csv`

Master file containing location information for all 1,298 voting areas (afstemningsområder).

**Primary Key**: `ValgstedId`

**Columns**:
- `Gruppe`: Group identifier
- `ValgstedId`: Unique voting area identifier (PRIMARY KEY)
- `KredsNr`: Electoral district number
- `StorKredsNr`: Large electoral district number  
- `LandsdelsNr`: Regional number

### Topic-Specific Files

Each file contains data for a specific topic, organized by voting area.

**Foreign Key**: All files use `ValgstedId` to link to `valgsteds.csv`

**All files follow this structure**:
```
Gruppe;ValgstedId;KredsNr;StorKredsNr;LandsdelsNr;[topic-specific columns]
```

**All EV2024 prefixes have been removed** from column names for readability.

## File Listing

### 1. **Antal_personer_opgjort_efter_statsborgerskab_kn___.csv**
   - Population by citizenship, gender, and age groups
   - 389 columns (384 demographic breakdowns)
   - Focus: Citizenship distribution by gender and age

### 2. **Boliger_og_personer_efter_boligstørrelse.csv**
   - Housing and population by dwelling size
   - 19 columns
   - Size categories: 0-39m², 40-79m², 80-119m², 120-159m², 160-199m², 200+m²

### 3-4. **BoligtypeAntalbbol
iger.csv** & **BoligtypeAntalpersoner.csv**
   - Housing types: Single-family houses, row houses, apartment buildings, dormitories, summer houses
   - 11 columns each (both units and persons)

### 5-6. **EjerforholdAntalbboliger.csv** & **EjerforholdAntalpersoner.csv**
   - Housing ownership: Owner-occupied, cooperative, public housing, rental, etc.
   - 11 columns each

### 7. **Husstande_efter_bilrådighed.csv**
   - Household vehicle access: 0 cars, 1 car, 2+ cars
   - 8 columns (including location data)

### 8. **Husstandsindkomster_fordelt_p_afstemningsomrder.csv**
   - Household income distribution and statistics
   - 22 columns including percentiles, income brackets, totals

### 9. **Hjst_fuldfrte_erhvervsuddannelse_og_aldersgrupper.csv**
   - Highest completed education level by age groups
   - 101 columns (education levels × age groups)
   - Education categories: Primary school through higher education

### 10. **Indvandrere_og_efterkommere_fordelt_efter_oprin___.csv**
   - Immigrants and descendants by origin country
   - 357 columns (broken down by gender, age, and origin)
   - Countries: Nordic, Turkey, Former Yugoslavia, EU, Africa, Americas, etc.

### 11-14. **Kriminalitet_*.csv** (4 files)
   - Criminal records data by voting area
   - `B_Anholdelser.csv`: Arrests
   - `C_Retsafgrelser1_Skyldig.csv`: Guilty verdicts
   - `C_Retsafgrelser2_Ikke_skyldig.csv`: Not guilty verdicts
   - `D_Strafferetslige_afgrelser_efter___.csv`: Court decisions by origin

### 15. **Medlemsskab_af_folkekirken_fordelt_p_afstemning___.csv**
   - Church membership (Danish People's Church)
   - 7 columns (members and non-members)

### 16-31. **Opførelsesår_*.csv** (16 files)
   - Housing construction year statistics
   - Broken into periods: before 1940, 1940-1959, 1960-1969, 1970-1979, 1980-1989, 1990-1999, 2000-2009, 2010+
   - Both dwelling units and persons counted

### 32. **Personer_fordelt_efter_samlet_ydelsesvarighed_a___.csv**
   - Social benefits duration by age and voting area
   - 25 columns with age groups and time periods

### 33. **Sociokonomisk_status_og_brancher_fordelt_p_afst___.csv**
   - Socio-economic status and industries
   - 141 columns (employment status × industry sectors)
   - Sectors: Agriculture, manufacturing, construction, retail, IT, finance, real estate, services, etc.

## How to Use This Data

### 1. **Query by Voting Area**
   ```
   Join any topic file with valgsteds.csv using ValgstedId
   Example: Get all demographic data for a specific voting area
   ```

### 2. **Regional Analysis**
   ```
   All location data (KredsNr, StorKredsNr, LandsdelsNr) is available 
   in every file for regional grouping and analysis
   ```

### 3. **Cross-Topic Queries**
   ```
   Combine data from multiple topic files using common ValgstedId
   Example: Correlate income (topic 8) with education (topic 9)
   ```

## Data Quality Notes

- **All 1,298 voting areas** have been deduplicated and consolidated
- **Missing values** are represented as empty strings or "-"
- **Column names** have been cleaned by removing "EV2024 - " prefixes
- **Encoding**: UTF-8 throughout
- **Delimiter**: Semicolon (;) as in original files
- **Data scope**: 2024 Danish Parliamentary Election (EV 2024)

## Recommended Next Steps

1. **Database Import**: Import `valgsteds.csv` as the primary table, then import topic files with foreign key constraints
2. **Data Validation**: Check for data types and NULL handling
3. **Aggregation**: Group data by `StorKredsNr` or `LandsdelsNr` for regional summaries
4. **Visualization**: Use geographic data with the included GIS file (`afstemnings_områder.gpkg`)

## Column Naming Convention

After the 5 location columns, all column names follow this pattern:

```
[Metric Description]_[Category 1]_[Category 2]_...
```

Examples:
- `Antal personer_Kvinder_0-4 år_01. Danmark`
- `Antal boliger_Ejerbolig`
- `Branche_Landbrug`

## File Statistics

| Metric | Value |
|--------|-------|
| Total Files | 34 |
| Total Rows (per file) | 1,298 |
| Voting Areas | 1,298 |
| Total Data Points | ~500,000+ |
| Largest File | 1.1 MB (Demographics) |
| Total Size | ~6.5 MB |

## Contact & Source

- **Source**: Danmarks Statistik (Statistics Denmark)
- **Database**: Valgdatabase (Election Database)
- **URL**: https://valgdatabase.dst.dk/data
- **Election**: EV 2024 (2024 Parliamentary Election)
- **Data Units**: Election voting areas by location

---

Generated: March 19, 2026
Last Updated: By reorganize_election_data.py script
