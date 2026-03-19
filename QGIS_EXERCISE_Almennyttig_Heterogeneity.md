# QGIS Exercise: Almennyttig Heterogeneity Within Municipalities

**Objective:** Calculate and visualize how Almennyttig (non-profit) housing shares vary within vs. between municipalities using only QGIS.

**Data files needed:**
- `afstemningsomraader_2024.geojson` (voting areas with boundaries and area metadata)
- `ejerforhold_shares_by_valgomraade.csv` (Almennyttig data by area)

---

## Part 1: Load and Join Data

### Step 1: Open QGIS and load the GeoJSON
1. **Layer** → **Add Layer** → **Add Vector Layer**
2. Select `afstemningsomraader_2024.geojson`
3. Click **Add**

You now have 1,298 voting area polygons.

### Step 2: Join the CSV data
1. Right-click the layer → **Properties** → **Joins**
2. Click the `+` icon to add a join
3. **Join layer:** `ejerforhold_shares_by_valgomraade` (import CSV first via **Add Layer** → **Add Delimited Text Layer**)
4. **Join field:** `ValgstedId` (from CSV)
5. **Target field:** `valgstedid` (from GeoJSON)
6. Click **OK** twice

Now the layer has columns including `ShareAlmennyttig`, `KommuneKode`, `KommuneNavn`, `muni_mean`, and `muni_std`.

---

## Part 2: Calculate Deviation and Group Statistics

### Step 3: Add a "Deviation" field
1. Open **Attribute Table** (right-click layer → **Open Attribute Table**)
2. Click **Field Calculator** (icon with `ƒ` at top)
3. **Output field name:** `Deviation`
4. **Expression:** `"ShareAlmennyttig" - "muni_mean"`
5. Click **OK**

Each voting area now shows how far its Almennyttig share is from its municipality average.

### Step 4: Style the layer by deviation
1. Right-click layer → **Properties** → **Symbology**
2. **Symbology type:** **Graduated**
3. **Column:** `Deviation`
4. **Color ramp:** Choose a **Diverging** ramp (e.g., RdBu or coolwarm, reversed so red = above average)
5. **Classes:** 7 or 10
6. Click **Classify** → **OK**

**Map interpretation:** Red areas have above-municipality-average shares; blue areas have below-average shares. Visually, you see where municipalities are heterogeneous.

---

## Part 3: Calculate Within vs. Between Variation

### Step 5: Dissolve to municipality level
1. **Vector** → **Geoprocessing Tools** → **Dissolve**
2. **Input layer:** Your voting areas layer
3. **Dissolve field:** `KommuneKode`
4. **Aggregation:** For column `ShareAlmennyttig`, choose `mean()`
5. Click **Run**

Result: One polygon per municipality, showing the mean Almennyttig share across its voting areas.

### Step 6: Create a summary table (municipality-level statistics)
1. **Vector** → **Analysis Tools** → **Statistics by Categories**
2. **Input layer:** Original voting areas layer
3. **Category field:** `KommuneKode`
4. **Value field:** `ShareAlmennyttig`
5. Click **Run**

This opens a dialog showing `count`, `sum`, `mean`, `stddev`, `min`, `max` **per municipality**.

**Export this as a CSV:**
- In the results window, right-click → **Save as** CSV.

### Step 7: Calculate manual variance decomposition in a spreadsheet
Use the municipality summary table you just exported:

1. **Overall mean:** Average of all municipality means.
2. **Between-municipality SS:** Sum of (municipality_mean - overall_mean)² × count_in_municipality
3. **Within-municipality SS:** Sum of (standard_deviation²) × (count - 1) for each municipality
4. **Total SS:** Between SS + Within SS
5. **Between variance %:** Between SS / Total SS
6. **Within variance %:** Within SS / Total SS

**Example calculation in a spreadsheet:**
```
Column A: KommuneNavn
Column B: mean_Almennyttig (from Statistics by Categories)
Column C: stddev_Almennyttig
Column D: n_areas (count)

Overall Mean = AVERAGE(B:B)

Column E: (B - Overall Mean)² * D  [Between component]
Column F: C² * (D - 1)             [Within component]

Total Between SS = SUM(E:E)
Total Within SS = SUM(F:F)
Total SS = Total Between SS + Total Within SS

Between % = Total Between SS / Total SS * 100
Within % = Total Within SS / Total SS * 100
```

---

## Part 4: Visualize Your Findings

### Step 8: Create two maps

**Map 1: Municipality Means (External Variation)**
- Use the dissolved layer (Step 5)
- Style by `mean_ShareAlmennyttig` with a sequential color ramp (light to dark)
- This shows which municipalities have the highest baseline Almennyttig share.
- **Question for students:** Which regions have consistently high/low shares?

**Map 2: Voting Area Deviations (Internal Variation)**
- Use the original layer with the `Deviation` field styled (Step 4)
- This shows where within each municipality, areas differ from the municipality average.
- **Question for students:** Where is heterogeneity greatest?

---

## Expected Results

For Almennyttig across your dataset:
- **Between-municipality variation:** ~35% of total variance
- **Within-municipality variation:** ~65% of total variance

**Interpretation:** Most differences in non-profit housing shares occur within municipalities (local areas differ from their own municipality average), not between municipalities.

---

## Challenge Questions for Students

1. Compare the municipality-mean map with the deviation map. Do regions with high average shares also have high internal heterogeneity?
2. Which municipality has the most internal variation (highest stddev)?
3. Which voting areas are most unusual compared to their municipality?
4. What might explain high internal variation in a municipality? (e.g., central districts vs. suburbs)

---

## Files for Student Distribution

Package these in a folder:
```
student_exercise/
├── afstemningsomraader_2024.geojson
├── ejerforhold_shares_by_valgomraade.csv
└── QGIS_EXERCISE_Almennyttig_Heterogeneity.md
```

Students open QGIS, follow the steps, and produce:
- Two styled maps (municipality means + area deviations)
- One summary table with variance decomposition
- Written reflection on findings
