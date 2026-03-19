# Time In GIS: Understanding Current, Temporal, And Bitemporal Data (DAGI)

This note explains how to work with time in GIS using Danish DAGI election district data.

## Why Time Matters In GIS
A map object (for example an election district) can change over time. In many registers we need to answer at least two different time questions:

1. Valid time (virkningstid): When was the object valid in the real world?
2. Transaction time (registreringstid): When was this version recorded in the system?

DAGI supports both ideas.

## The Four Core Time Fields

### 1) virkningFra
- Meaning: Start of real-world validity.
- Question answered: From when did this district legally/administratively apply?

### 2) virkningTil
- Meaning: End of real-world validity.
- If NULL: Still valid in real-world terms (open-ended period).

### 3) registreringFra
- Meaning: When this record version was entered in the register.

### 4) registreringTil
- Meaning: When this record version stopped being the newest recorded version.
- If NULL: It is the currently active registration version.

## The Three Dataset Types

### Current
- Contains only the latest known version of currently active objects.
- Typical pattern: virkningTil = NULL and registreringTil = NULL.
- Use when you need "what exists now".

### Temporal
- Contains historical object validity over time (valid-time history).
- Typical pattern: registreringTil = NULL, while virkningFra/virkningTil describe historical periods.
- Use when you need "how the world looked at time T" based on current knowledge.

### Bitemporal
- Contains both valid-time history and registration history.
- Both virkningTil and registreringTil may have values.
- Use when you need:
  - what was valid at time T, and
  - what was known/registered at time R.

This is the classical "two times" model in GIS.

## Scale Filter In DAGI
For election districts, scale/generalization level is controlled by field skala.
Possible values include:
- 1:10.000
- 1:250.000
- 1:500.000
- 1:2.000.000

For detailed analysis of election districts, use:
- skala = '1:10.000'

## Practical Filter Examples (QGIS Layer Filter)

In QGIS, right-click layer -> Filter..., then paste one of the expressions below.

### A) Election districts valid on 2024-06-09 using Temporal
Use this when you want how we today represent districts for EU election day 2024.

```sql
"virkningfra" < '2024-06-09T00:00:00Z'
AND (
  "virkningtil" > '2024-06-09T00:00:00Z'
  OR "virkningtil" IS NULL
)
AND "skala" = '1:10.000'
```

### B) Same from Bitemporal, but only latest registration versions
Use this when you want current registration view of those historically valid districts.

```sql
"virkningfra" < '2024-06-09T00:00:00Z'
AND (
  "virkningtil" > '2024-06-09T00:00:00Z'
  OR "virkningtil" IS NULL
)
AND "registreringtil" IS NULL
AND "skala" = '1:10.000'
```

### C) What districts for 2024-06-09 looked like as known on 2024-06-09 (true bitemporal snapshot)
Use this when both valid time and registration time are fixed to the same date.

```sql
"virkningfra" < '2024-06-09T00:00:00Z'
AND (
  "virkningtil" > '2024-06-09T00:00:00Z'
  OR "virkningtil" IS NULL
)
AND "registreringfra" < '2024-06-09T00:00:00Z'
AND (
  "registreringtil" > '2024-06-09T00:00:00Z'
  OR "registreringtil" IS NULL
)
AND "skala" = '1:10.000'
```

## Interpretation Guide For Students
- Temporal filter answers: "Which districts were valid then (according to current register state)?"
- Bitemporal with registreringTil IS NULL answers: "Which historical districts are selected by today’s latest registrations?"
- Full bitemporal snapshot answers: "What did the register know at that time about what was valid at that time?"

## Teaching Summary
When using bitemporal GIS data, always ask two time questions:
1. Valid time question: When should the object be valid in the real world?
2. Registration time question: At what point in the database history do we observe knowledge about that object?

If you do not explicitly choose both time axes in bitemporal data, you may mix "what was valid" with "what we learned later".

## Suggested Classroom Exercise
1. Run filter A and count features.
2. Run filter B and compare count.
3. Run filter C and compare count.
4. Explain why counts may differ and what each result means conceptually.
5. Export each result and map differences.

This exercise demonstrates why time semantics matter in GIS analysis, legal interpretation, and reproducibility.
