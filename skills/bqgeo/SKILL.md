---
name: bqgeo
description: "Expert guidance for BigQuery geospatial analytic queries and best practices. Use this skill when the user wants to perform spatial analysis using SQL in Google Cloud BigQuery."
version: 1.0.0
---

# BigQuery Geospatial Skill

## Overview
This skill provides patterns and best practices for performing geospatial analysis in Google Cloud BigQuery using its built-in GEOGRAPHY functions.

## Key Geospatial Functions

### Constructors
- `ST_GEOGPOINT(longitude, latitude)`
- `ST_GEOGFROMTEXT(wkt_string)`
- `ST_GEOGFROMGEOJSON(geojson_string)`

### Accessors
- `ST_ASTEXT(geography)`
- `ST_X(point)`, `ST_Y(point)`
- `ST_AREA(geography)`
- `ST_LENGTH(geography)`

### Relationships
- `ST_INTERSECTS(geog1, geog2)`
- `ST_CONTAINS(geog1, geog2)`
- `ST_DWITHIN(geog1, geog2, distance_meters)`

### Transformations
- `ST_BUFFER(geog, distance_meters)`
- `ST_UNION(geog1, geog2)`
- `ST_INTERSECTION(geog1, geog2)`

## Best Practices
1. **SRID:** BigQuery Geography uses WGS84 (EPSG:4326) on a spherical surface. All coordinates MUST be longitude, latitude order.
2. **Clustering:** Always cluster your tables by a GEOGRAPHY column for optimal performance in spatial joins.
3. **Joins:** Prefer `ST_DWithin` or `ST_Intersects` in the `JOIN` condition rather than a `WHERE` clause for better performance.
4. **Data Types:** Use the `GEOGRAPHY` data type for spatial data, not `STRING` or `BYTES`.

## Procedures

### Creating a Spatially Indexed Table
1.  Define the schema with a `GEOGRAPHY` column.
2.  Use `CLUSTER BY <geography_column>` in the `CREATE TABLE` statement.
    ```sql
    CREATE TABLE `project.dataset.table`
    (
      id INT64,
      geom GEOGRAPHY
    )
    CLUSTER BY geom;
    ```

### Performing a Spatial Join
1.  Identify two tables with `GEOGRAPHY` columns.
2.  Use `ST_INTERSECTS` or `ST_DWITHIN` in the `JOIN ON` clause.
    ```sql
    SELECT a.id, b.attribute
    FROM `project.dataset.table_a` AS a
    INNER JOIN `project.dataset.table_b` AS b
    ON ST_INTERSECTS(a.geom, b.geom);
    ```
