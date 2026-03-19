# Attribution & Data Source

## Original Data Source

This dataset is derived from **Statistics Denmark (Danmarks Statistik)** election data available at:
- **URL**: https://valgdatabase.dst.dk/data
- **Organization**: Statistics Denmark (dst.dk)
- **Datasets**: Danish 2024 Election Statistics organized by geographic voting areas (afstemningsområder)

## Geometry Source

Administrative geometry is provided by **Klimadatastyrelsen**.

- **Organization**: Klimadatastyrelsen
- **Format (original)**: GeoPackage
- **License**: CC-BY-4.0 (attribution required)

If you publish or redistribute derived geospatial files (for example GeoJSON or TopoJSON), include source accreditation to Klimadatastyrelsen and avoid use of official logos.

## Citation

When using this dataset, please cite as follows:

**BibTeX**:
```bibtex
@dataset{danish_election_2024,
  author = {{Statistics Denmark}},
  title = {Danish 2024 Election Data - Reorganized by Topic and Voting Area},
  year = {2024},
  url = {https://valgdatabase.dst.dk/data},
  organization = {Statistics Denmark (Danmarks Statistik)}
}
```

**APA**:
> Statistics Denmark. (2024). Danish 2024 election data - Reorganized by topic and voting area. Retrieved from https://valgdatabase.dst.dk/data

**Chicago**:
> Statistics Denmark. "Danish 2024 Election Data - Reorganized by Topic and Voting Area." Accessed 2024. https://valgdatabase.dst.dk/data

## Usage Rights

According to Statistics Denmark's [Source Attribution Policy](https://www.dst.dk/en/presse/kildeangivelse):

✓ **Allowed**:
- Re-use and reproduction of contents (including commercial use)
- Translation and adaptation
- Integration into analyses and research

✗ **Not Allowed**:
- Use of Statistics Denmark logo
- Misrepresentation of data origin

**Required**:
- Include source accreditation to Statistics Denmark
- Include source accreditation to Klimadatastyrelsen for geometry
- Maintain attribution in any derivative works

## License

This reorganized dataset is licensed under **Creative Commons Attribution 4.0 International (CC-BY-4.0)**.

You are free to:
- Share, adapt, and use this data for any purpose (including commercial)
- Modify and remix the data

**Provided that you**:
- Give appropriate credit to Statistics Denmark (tabular data) and Klimadatastyrelsen (geometry)
- Indicate if changes were made
- Provide a link to the license

See [LICENSE](LICENSE) for full legal text.

## Data Organization

The original data from Statistics Denmark was reorganized to:
1. Consolidate 5 large CSV files into topic-specific files
2. Remove redundant geographic columns (Gruppe, ValgstedId, KredsNr, StorKredsNr, LandsdelsNr) and place them in a single reference table (valgsteds.csv)
3. Standardize column naming by removing "EV2024" prefixes
4. Create a normalized data structure with foreign key relationships via ValgstedId

This reorganization makes the data more accessible for analysis and GIS applications while maintaining full data integrity.

Spatial joins are performed against geometry data distributed under CC-BY-4.0 by Klimadatastyrelsen.

## Questions?

For questions about:
- **Original data**: Visit https://valgdatabase.dst.dk/ or contact Statistics Denmark
- **Geometry source**: Contact Klimadatastyrelsen
- **Reorganization**: Check the documentation files (README.md, SCHEMA.md, USAGE_GUIDE.md)
- **Terms of use**: See https://www.dst.dk/en/presse/kildeangivelse
