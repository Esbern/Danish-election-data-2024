# Data Schema Reference

## Database Structure

The reorganized election data follows a normalized database structure with one reference table and multiple topic-specific detail tables.

### Entity Relationship Diagram

```
valgsteds (Reference Table)
    ├── PK: ValgstedId
    └── Has many: [All Topic Files via ValgstedId]

Topic Files (1-33)
    ├── FK: ValgstedId → valgsteds.ValgstedId
    └── Contains: Topic-specific metrics
```

## Reference Table: `valgsteds.csv`

The master reference file for all voting areas.

| Column Name | Type | Description | Example |
|---|---|---|---|
| Gruppe | string | Group identifier | "" (empty in data) |
| ValgstedId | integer | **Primary Key** - Unique voting area ID | 101001 |
| KredsNr | integer | Electoral district number | 1-10 |
| StorKredsNr | integer | Large electoral district number | 1-3 |
| LandsdelsNr | integer | Regional number | 1-5 |
| KommuneKode | string | Municipality code derived from the first 3 digits of ValgstedId | 101 |
| KommuneNavn | string | Municipality name from Dataforsyningen | København |
| Regionskode | string | Region code from Dataforsyningen | 1084 |
| RegionNavn | string | Region name from Dataforsyningen | Region Hovedstaden |

**Total Records**: 1,298 voting areas  
**Primary Key**: `ValgstedId`

These administrative columns make it easier to aggregate or join election district data to standard Danish municipality and region datasets.

## Topic File Structure Pattern

All topic files follow this standard structure:

```
Location Columns (5 shared with valgsteds.csv):
├── Gruppe
├── ValgstedId [FOREIGN KEY]
├── KredsNr
├── StorKredsNr
└── LandsdelsNr

Topic-Specific Columns (N variables):
├── [Metric 1]
├── [Metric 2]
└── ... [Metric N]
```

### Example: Income Distribution File

```
Gruppe;ValgstedId;KredsNr;StorKredsNr;LandsdelsNr;
Under 100.000 kr.;100.000 - 149.999 kr;150.000 - 199.999 kr;...;
50%-percentil for husstandsindkomst;80%-percentil for husstandsindkomst
```

## Topic Classifications

### 1. Demographic Topics

| File | Focus | Temporal | Granularity |
|------|-------|----------|------------|
| Antal_personer_... | Population by citizenship/gender/age | Current | Age groups × Gender × Citizenship |
| Indvandrere_og_efterkommere... | Immigrants by origin | Current | Age × Gender × Origin country |

**Key Columns in Demographic Files**:
- Age groups: 0-4, 5-9, 10-14, 15-17, 18-19, 20-24, 25-29, ... 70+
- Gender: Kvinder (Female), Mænd (Male)
- Citizenship/Origin: 01. Danmark, 02. Nordiske lande, 03. Tyrkiet, ...

### 2. Housing Topics

| File | Focus | Measurement |
|------|-------|------------|
| Boliger_og_personer_efter_boligstørrelse | Size distribution | m² and unit counts |
| BoligtypeAntalbboliger | Housing types | Unit counts |
| BoligtypeAntalpersoner | Housing types | Population |
| EjerforholdAntalbboliger | Ownership types | Unit counts |
| EjerforholdAntalpersoner | Ownership types | Population |
| Opførelsesår_* | Construction period | Units and population |

**Housing Size Categories**:
- 0-39 m² | 40-79 m² | 80-119 m² | 120-159 m² | 160-199 m² | 200+ m²

**Ownership Types**:
1. Ejerbolig (Owner-occupied)
2. Andelsbolig (Cooperative)
3. Almennyttig bolig (Public housing)
4. Lejer i offentligt ejet (Public rental)
5. Privat lejerbolig (Private rental)
6. Øvrige beboede boliger (Other)

**Housing Types**:
- Stuehuse og parcelhuse (Single-family)
- Række- og kædehuse (Row houses)
- Flerfamiliehuse (Apartment buildings)
- Kollegier (Dormitories)
- Sommerhuse (Summer houses)
- Øvrige boliger (Other)

### 3. Socio-Economic Topics

| File | Focus | Breakdown |
|------|-------|-----------|
| Socio-økonomisk_status... | Employment & industry | Status × Industry |
| Højst_fuldførte_erhvervsuddannelse... | Education level | Age × Education level |
| Husstandsindkomster_fordelt... | Income distribution | Income brackets + statistics |

**Employment Status Codes**:
1. Selvstændig og medhj. (Self-employed/helper)
2. Topledere (Top managers)
3. Lønmodtagere på højt niveau (High-level employees)
4. Lønmodtagere på mellem niveau (Mid-level employees)
5. Lønmodtagere på grundniveau (Basic-level employees)
6. Øvrige lønmodtagere (Other employees)
7. Arbejdsløse (Unemployed)
8. Uddannelsessøgende (Students)
9. Folke- og tjenestemandspension (Pensioners)
10. Efterløn (Early retirement)
11. Børn (Children)
12. Øvrige (offentligt forsørgede) (Other public support)

**Education Levels**:
1. Grundskole (Primary school)
2. Almengymnasial (General secondary)
3. Erhvervsgymnasial (Vocational secondary)
4. Erhvervsfaglig (Vocational)
5. Kort videregående (Short tertiary)
6. Mellemlang videregående (Medium tertiary)
7. Lang videregående (Long tertiary)
8. Uoplyst (Unknown)

### 4. Other Topics

| File | Focus | Values |
|------|-------|--------|
| Husstande_efter_bilrådighed | Vehicle access | 0, 1, 2+ cars |
| Medlemsskab_af_folkekirken | Church membership | Members / Non-members |
| Mennesker_fordelt_efter_samlet_ydelsesvarighed | Benefit duration | Age × Duration periods |
| Kriminalitet_*.csv | Crime statistics | Arrests, verdicts, origin |

## Data Types

All data values are stored as strings (CSV format):

| Value Format | Meaning | Example |
|---|---|---|
| Numeric | Count or quantity | `1298`, `45` |
| Zero `0` | Standardized missing/unavailable marker (from original `-`) | `0` |
| Empty | Not applicable | `` |
| Country names | Geographic origin | `Danmark`, `Tyrkiet` |

## Common Queries

### Query 1: Get all data for a voting area
```python
import pandas as pd

valgsteds = pd.read_csv('valgsteds.csv', sep=',')
demographics = pd.read_csv('Antal_personer_...csv', sep=',')

area_data = demographics[demographics['ValgstedId'] == 101001]
```

### Query 2: Sum by region
```python
data_by_region = demographics.groupby('StorKredsNr')[metric_columns].sum()
```

### Query 3: Join multiple topics
```python
demographics = pd.read_csv('Antal_personer_...csv', sep=',')
housing = pd.read_csv('Boliger_og_personer_...csv', sep=',')
income = pd.read_csv('Husstandsindkomster_...csv', sep=',')

combined = demographics.merge(housing, on='ValgstedId')\
                      .merge(income, on='ValgstedId')
```

## Data Completeness

- **Voting Areas Covered**: All 1,298 Danish voting areas
- **Redundancy**: Each topic file contains location columns for easy reference
- **Missing Data**: Original `-` values standardized to `0`
- **Duplicates**: None - each ValgstedId appears once per file

## Integration Notes for Databases

### PostgreSQL Import Example

```sql
-- Create reference table
CREATE TABLE valgsteds (
    gruppe VARCHAR(255),
    valgsted_id INTEGER PRIMARY KEY,
    kreds_nr INTEGER,
    stor_kreds_nr INTEGER,
    landsdels_nr INTEGER
);

-- Create topic table example
CREATE TABLE befolkning_demographics (
    gruppe VARCHAR(255),
    valgsted_id INTEGER PRIMARY KEY REFERENCES valgsteds(valgsted_id),
    kreds_nr INTEGER,
    stor_kreds_nr INTEGER,  
    landsdels_nr INTEGER,
    -- Topic-specific columns
    kvinder_0_4_danmark INTEGER,
    kvinder_0_4_nordiske_lande INTEGER,
    -- ... more columns
);

-- Import data
COPY valgsteds FROM 'valgsteds.csv' WITH (FORMAT csv, DELIMITER ',', HEADER);
COPY befolkning_demographics FROM 'Antal_personer_...csv' WITH (FORMAT csv, DELIMITER ',', HEADER);
```

## File Version History

- **Current Version**: 1.0
- **Created**: March 19, 2026
- **Source Data**: EV 2024 Election Database (dst.dk)
- **Organization Method**: Topic-based normalization with foreign key relationships
- **Original Files**: 5 Befolkning*.csv files consolidated into 33 topic files + 1 reference

---

**Last Updated**: March 19, 2026
**Reorganization Script**: `reorganize_election_data.py`
