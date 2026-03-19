# Usage Guide: Working with Reorganized Election Data

## Quick Start

All organized data files are in the `organized_data/` folder:

```
valgsteds.csv                                    ← Master reference file
Antal_personer_...csv                           ← Demographics
Boliger_og_personer_...csv                      ← Housing
... 17 more topic files ...
README.md                                        ← This documentation
SCHEMA.md                                        ← Database schema details
```

**Key Point**: Use `ValgstedId` as the foreign key to link all files together.

---

## Python Examples

### 1. Load and explore data

```python
import pandas as pd

# Load master reference
valgsteds = pd.read_csv('organized_data/valgsteds.csv', sep=',')
print(f"Total voting areas: {len(valgsteds)}")
# Output: Total voting areas: 1298

# Load a topic file
demographics = pd.read_csv('organized_data/Antal_personer_opgjort_efter_statsborgerskab_kn___.csv', sep=',')
print(demographics.columns[:10])
```

### 2. Get data for a specific voting area

```python
# Get all demographic data for voting area 101001
area_id = 101001
area_data = demographics[demographics['ValgstedId'] == area_id]

# Get location information
area_info = valgsteds[valgsteds['ValgstedId'] == area_id]
print(f"Electoral district: {area_info['KredsNr'].values[0]}")
```

### 3. Regional aggregation

```python
# Join to reference for regional fields
demographics_with_geo = demographics.merge(
    valgsteds[['ValgstedId', 'StorKredsNr', 'LandsdelsNr', 'KommuneKode', 'RegionNavn']],
    on='ValgstedId',
    how='left'
)

# Sum all demographic data by electoral district
by_district = demographics_with_geo.groupby('StorKredsNr').sum(numeric_only=True)
print(by_district)

# Or by region
by_region = demographics_with_geo.groupby('LandsdelsNr').sum(numeric_only=True)
```

### 4. Combine multiple topics

```python
# Get income and education data for analysis
income = pd.read_csv('organized_data/Husstandsindkomster_fordelt_p_afstemningsomrder.csv', sep=',')
education = pd.read_csv('organized_data/Hjst_fuldfrte_erhvervsuddannelse_og_aldersgrupper.csv', sep=',')

# Merge by voting area
combined = income.merge(education, on='ValgstedId', suffixes=('_income', '_edu'))

# Analyze correlation
print(combined[['Antal husstande i alt', 'Lang videregående']].corr())
```

### 5. Filter and explore

```python
# Find voting areas with highest income (50th percentile)
high_income = demographics[demographics['50%-percentil for husstandsindkomst'].astype(float) > 400000]
print(f"High-income areas: {len(high_income)}")

# Get voting areas by region
region_1 = demographics[demographics['LandsdelsNr'] == 1]
print(f"Voting areas in region 1: {len(region_1)}")
```

### 6. Data cleaning

```python
# Missing values are already standardized to 0
demographics_clean = demographics.copy()

# Convert to numeric
numeric_cols = demographics.columns[1:]  # Skip ValgstedId
demographics[numeric_cols] = demographics[numeric_cols].apply(pd.to_numeric, errors='coerce')
```

---

## SQL Examples (SQLite)

### 1. Import CSV files

```sql
-- Create and import reference table
CREATE TABLE valgsteds (
    gruppe TEXT,
    valgsted_id INTEGER PRIMARY KEY,
    kreds_nr INTEGER,
    stor_kreds_nr INTEGER,
    landsdels_nr INTEGER
);

.mode csv
.separator ,
.import 'organized_data/valgsteds.csv' valgsteds

-- Create and import demographic table
CREATE TABLE demografis AS
SELECT * FROM (
    SELECT * FROM valgsteds
);

.import 'organized_data/Antal_personer_...csv' demographic
```

### 2. Query by region

```sql
SELECT 
    stor_kreds_nr,
    COUNT(*) as num_areas,
    AVG(CAST(befolkningstal AS FLOAT)) as avg_population
FROM demographic
GROUP BY stor_kreds_nr
ORDER BY stor_kreds_nr;
```

### 3. Join tables

```sql
SELECT 
    v.valgsted_id,
    v.KredsNr,
    d.befolkningstal,
    i.indkomst_gns
FROM valgsteds v
JOIN demographic d ON v.valgsted_id = d.valgsted_id
JOIN income i ON v.valgsted_id = i.valgsted_id
WHERE v.stor_kreds_nr = 2;
```

### 4. Aggregation

```sql
SELECT 
    stor_kreds_nr,
    SUM(CAST(antal_boliger AS INTEGER)) as total_housing,
    AVG(CAST(antal_personer AS FLOAT)) as avg_population
FROM housing_data
GROUP BY stor_kreds_nr
HAVING SUM(antal_boliger) > 50000;
```

---

## R Examples

```r
# Install if needed: install.packages(c("tidyverse", "readr"))
library(tidyverse)

# Load data
valgsteds <- read_delim("organized_data/valgsteds.csv", delim = ",")
demographics <- read_delim("organized_data/Antal_personer_...csv", delim = ",")

# Regional summary
demographic_summary <- demographics %>%
  group_by(StorKredsNr) %>%
  summarise(across(starts_with("Kvinder") | starts_with("Mænd"), sum, na.rm=TRUE))

# Join and analyze
combined <- inner_join(demographics, income, by = "ValgstedId") %>%
  mutate(income_numeric = as.numeric(`50%-percentil for husstandsindkomst`))

# Visualization
combined %>%
  ggplot(aes(x = StorKredsNr, y = income_numeric)) +
  geom_boxplot() +
  theme_minimal()
```

---

## Excel/LibreOffice

### 1. Open files
- Use "Open" and select any CSV file
- Choose "," as delimiter when prompted
- UTF-8 encoding should be auto-detected

### 2. Use VLOOKUP for references
```excel
=VLOOKUP(A2, valgsteds!$A:$E, 2, FALSE)
```

### 3. Pivot Tables
1. Select data range
2. Insert → Pivot Table
3. Drag `ValgstedId` to Rows
4. Drag numeric columns to Values
5. Add filters as needed

---

## Common Data Transformations

### 1. From text to numeric

```python
# Remove thousand separators if present
demographics['total'] = (demographics['antal']
    .str.replace('.', '', regex=False)
    .str.replace(',', '.', regex=False)
    .astype(float))

# Optional: treat 0 as missing for selected analyses
demographics = demographics.replace(0, np.nan)
```

### 2. Calculate percentages

```python
# Calculate % of voting areas with specific characteristic
housing = pd.read_csv('organized_data/BoligtypeAntalbboliger.csv', sep=',')
total = housing[['Stuehuse og parcelhuse']].sum()[0]
housing['pct_singlefamily'] = (housing['Stuehuse og parcelhuse'].astype(float) / total * 100)
```

### 3. Create derived metrics

```python
# Income distribution analysis
income = pd.read_csv('income_file.csv', sep=',')
income['income_range'] = pd.cut(
    income['50%-percentil for husstandsindkomst'].astype(float),
    bins=[0, 300000, 400000, 500000, float('inf')],
    labels=['Low', 'Medium', 'High', 'Very High']
)
```

---

## Troubleshooting

### Issue: Character encoding errors
**Solution**: Always specify UTF-8 encoding
```python
df = pd.read_csv('file.csv', sep=',', encoding='utf-8')
```

### Issue: standardized 0 values should be treated as missing in analysis
**Solution**: Replace selected zero-coded fields with NaN before modeling
```python
df = df.replace(0, pd.NA)
```

### Issue: ValgstedId doesn't match between files
**Solution**: Check for leading zeros or type mismatches
```python
demographics['ValgstedId'] = demographics['ValgstedId'].astype(str)
valgsteds['ValgstedId'] = valgsteds['ValgstedId'].astype(str)
```

### Issue: Some voting areas missing from a file
**Solution**: Perform LEFT JOIN instead of INNER JOIN
```python
result = demographics.merge(housing, on='ValgstedId', how='left')
```

---

## Performance Tips

### For large datasets:
1. **Filter first**: Select specific voting areas before analysis
2. **Use chunking**: Process files in chunks if memory-limited
3. **Lazy loading**: Only load columns you need
4. **Indexing**: Set ValgstedId as index for faster lookups

```python
# Example: Filter before loading
dtype_dict = {'ValgstedId': int}
demographics = pd.read_csv('file.csv', sep=',', dtype=dtype_dict)
geo = pd.read_csv('organized_data/valgsteds.csv', sep=',', dtype=dtype_dict)
filtered = demographics.merge(geo[['ValgstedId', 'StorKredsNr']], on='ValgstedId', how='left')
filtered = filtered[filtered['StorKredsNr'].isin([1, 2])]
```

---

## Integration with GIS

The `afstemnings_områder.gpkg` file contains geographic boundaries for voting areas:

```python
import geopandas as gpd
import pandas as pd

# Load geographic data
geom = gpd.read_file('afstemnings_områder.gpkg')

# Load demographic data
demographics = pd.read_csv('demographics.csv', sep=',')

# Merge by voting area ID
merged = geom.merge(demographics, on='ValgstedId')

# Create choropleth map
merged.plot(column='befolkningstal', cmap='OrRd')
```

---

## Next Steps

1. **Data validation**: Run quality checks on imported data
2. **Create views**: Set up database views for common analyses
3. **Build dashboards**: Connect to BI tools (Power BI, Tableau, Metabase)
4. **Archive**: Backup organized data with version numbers
5. **Document**: Document any custom transformations or derived metrics

---

**For more details**: See README.md and SCHEMA.md in the organized_data folder
