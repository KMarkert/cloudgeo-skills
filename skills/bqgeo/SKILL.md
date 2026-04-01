---
name: bqgeo
description: Expert guidance for BigQuery geospatial analytic queries and best practices. Use this skill when the user wants to perform spatial analysis using SQL in Google Cloud BigQuery, optimize spatial queries, load geographic data, or analyze raster data.
---

# BigQuery Geospatial Skill

This skill provides patterns, functions, and best practices for performing geospatial analysis in Google Cloud BigQuery using its built-in `GEOGRAPHY` data type and `ST_` functions.

## 📚 References & Workflows

**CRITICAL INSTRUCTION**: If the user's request involves any of the following specific workflows, you MUST read the corresponding reference file before proceeding.

*   **Loading Geospatial Data**: If the user asks to import, load, or ingest WKT, GeoJSON, CSV, or GeoParquet data into BigQuery, read `references/loading-data.md`.
*   **Raster Data Analysis**: If the user asks about raster data, zonal statistics, Earth Engine integration, or functions like `ST_RegionStats`, read `references/raster-data.md`.

---

## Core Concepts

- **`GEOGRAPHY` Data Type:** Represents a point, linestring, polygon, or a collection of these on the Earth's surface. 
- **Spherical Geometry:** BigQuery uses the **WGS84** reference system (EPSG:4326), treating the Earth as a sphere. Distances and areas are calculated using geodesic paths. All coordinates MUST be in **longitude, latitude** order.

## Key Geospatial Functions

BigQuery supports a wide array of `ST_` (Spatial Type) functions.

### Constructors
- `ST_GEOGPOINT(longitude, latitude)`: Creates a point.
- `ST_GEOGFROMTEXT(wkt_string)`: Creates a geography from Well-Known Text (WKT).
- `ST_GEOGFROMGEOJSON(geojson_string)`: Creates a geography from GeoJSON.

### Accessors
- `ST_ASTEXT(geography)` / `ST_ASGEOJSON(geography)`: Convert `GEOGRAPHY` to string formats.
- `ST_X(point)` / `ST_Y(point)`: Extract longitude/latitude.

### Measurements
- `ST_AREA(geography)`: Area in square meters.
- `ST_LENGTH(geography)`: Length in meters.
- `ST_DISTANCE(geog1, geog2)`: Shortest distance in meters between two geographies.

### Relationships (Predicates)
- `ST_INTERSECTS(geog1, geog2)`: True if the two geographies intersect.
- `ST_CONTAINS(geog1, geog2)`: True if `geog1` completely contains `geog2`.
- `ST_DWITHIN(geog1, geog2, distance_meters)`: True if the distance between them is `<= distance_meters`.

### Transformations
- `ST_BUFFER(geog, distance_meters)`: Expands a geography by a given radius.
- `ST_SIMPLIFY(geog, tolerance_meters)`: Simplifies a complex geometry, reducing vertices.

## Optimization & Best Practices

1. **Native Spatial Clustering (Critical):** Always cluster your tables by a `GEOGRAPHY` column for optimal performance in spatial joins and filters. BigQuery uses S2 indexing under the hood. *Note: You cannot partition on a `GEOGRAPHY` column.*
2. **Optimize Query Predicates:** To leverage spatial clustering, use specific predicate functions in your `WHERE` or `JOIN ON` clause (e.g., `ST_INTERSECTS`, `ST_DWITHIN`, `ST_CONTAINS`, `ST_COVERS`, `ST_WITHIN`). 
   - *Warning:* `ST_DISJOINT` does **not** benefit from spatial clustering.
3. **Data Types:** Always store spatial data using the native `GEOGRAPHY` data type rather than `STRING` or `BYTES` to enable clustering and fast processing.
4. **Join Order:** For spatial joins, prefer `ST_INTERSECTS` or `ST_DWITHIN` in the `JOIN ON` clause rather than filtering in a `WHERE` clause after a cross join. Ensure both tables are clustered for maximum performance.

## Core Procedures

### 1. Creating a Spatially Clustered Table
```sql
CREATE TABLE `my_project.my_dataset.my_table`
(
  id INT64,
  name STRING,
  geom GEOGRAPHY
)
CLUSTER BY geom;
```

### 2. Performing an Efficient Spatial Join
Joining a table of points to a table of polygons to find which polygon contains each point.
```sql
SELECT 
  p.id AS point_id, 
  poly.name AS polygon_name
FROM `my_dataset.points_table` AS p
INNER JOIN `my_dataset.polygons_table` AS poly
  ON ST_INTERSECTS(p.geom, poly.geom); -- Uses clustering effectively
```
