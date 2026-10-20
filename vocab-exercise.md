# **GIS Vocabulary and ETL Exercise**

## **Description**

**Research Context:** Investigate whether neighborhood-level prevalence of single-parent households is associated with EHR-documented suicide attempts among adolescents aged 12-17 in 2022.

**Scenario:** A 16-year-old patient (`person_id = 1001`) lived at `location_id = 501` from March 15 to November 30, 2022. Their residence falls within a U.S. Census tract where 8.4% of households are single-parent households with children under 18, according to CDC/ATSDR SVI 2022 data.

For this exercise, treat the indicator as applicable from January 1 through December 31, 2022.

This is a standalone ETL design exercise. Assume that the patient, location, and residence-history record described above already exist in the corresponding tables.

**Your ETL Challenge:** Write an SQL
`INSERT INTO omopgis.external_exposure` statement:
- Find the Standard Concepts for the **exposure, unit, source type, and spatial relationship**, using [Custom Vocabulary Browser (CVB)](https://ohdsi.github.io/CustomVocabularyBrowser/?repository=TuftsCTSI%2FCVB&ref=main&vocabulary=GIS) and [OHDSI Athena](https://athena.ohdsi.org/search-terms/terms)
- Ensure that the location history record refers to a patient (Person) and that the relationship represents their residence (Patient Residence).
- Derive the exposure dates from `LOCATION_HISTORY` and the indicator's applicability period.
- Populate as many relevant fields as possible, preserving the numeric value and source provenance.

## **SQL template**

Replace every placeholder in square brackets with the appropriate concept ID, value, or SQL expression.
Use numeric concept IDs without quotation marks, single quotes for text values, and `DATE 'YYYY-MM-DD'` for date literals.
The template is incomplete and becomes executable only after all placeholders have been replaced.

```sql
INSERT INTO omopgis.external_exposure (
    external_exposure_id,
    person_id,
    location_id,
    exposure_concept_id,
    exposure_start_date,
    exposure_end_date,
    exposure_type_concept_id,
    exposure_relationship_concept_id,
    exposure_source_concept_id,
    exposure_source_value,
    exposure_relationship_source_value,
    dose_unit_source_value,
    value_as_number,
    unit_concept_id
)
SELECT
    900001 AS external_exposure_id, -- Unique exposure record ID
    lh.entity_id AS person_id,         -- Person from LOCATION_HISTORY
    lh.location_id AS location_id,     -- Residence location
    [FIND AND PASTE TARGET CONCEPT_ID HERE] AS exposure_concept_id, -- Standard concept representing the exposure indicator (use concept class ID 'SVI Item' for the search in CVB)
    [EXPOSURE_START_DATE_EXPRESSION] AS exposure_start_date, -- Start of the overlap between residence and indicator applicability
    [EXPOSURE_END_DATE_EXPRESSION] AS exposure_end_date, -- End of the overlap between residence and indicator applicability
    [FIND AND PASTE TARGET CONCEPT_ID HERE] AS exposure_type_concept_id, -- Type of data source from which the exposure was derived
    [FIND AND PASTE TARGET CONCEPT_ID HERE] AS exposure_relationship_concept_id, -- standard concept for "ST_Within" (use CVB)
    [SOURCE_SVI_CONCEPT_ID] AS exposure_source_concept_id, -- SVI concept concept ID (use CVB)
    [CDC_SVI_VARIABLE_CODE] AS exposure_source_value,  -- Original CDC SVI variable code (concept code in CVB)
    'ST_Within' AS exposure_relationship_source_value, -- Source spatial predicate
    [ORIGINAL_UNIT] AS unit_source_value, -- Original unit text
    [NUMERIC_EXPOSURE_VALUE] AS value_as_number,  -- Numeric value reported for the tract
    [FIND AND PASTE TARGET CONCEPT_ID HERE] AS unit_concept_id -- UCUM Standard Concept for the unit (use Athena)
FROM omopgis.location_history lh
WHERE lh.entity_id = [PERSON_ID]
  AND lh.location_id = [LOCATION_ID]
  AND lh.domain_id = [PERSON_DOMAIN_CONCEPT_ID]
  AND lh.relationship_type_concept_id =
      [FIND AND PASTE TARGET CONCEPT_ID HERE] -- Patient Residence (use CVB)
  AND lh.start_date = [RESIDENCE_START_DATE]
  AND lh.end_date = [RESIDENCE_END_DATE];
```
