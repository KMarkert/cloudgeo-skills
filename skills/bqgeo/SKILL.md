---
name: bqgeo
description: Expert guidance for BigQuery geospatial analytic queries and best practices. Use this skill when the user wants to perform spatial analysis using SQL in Google Cloud BigQuery, optimize spatial queries, or load geographic data (like GeoJSON).
---

# BigQuery Geospatial Skill

This skill provides patterns, functions, and best practices for performing geospatial analysis in Google Cloud BigQuery using its built-in `GEOGRAPHY` data type and `ST_` functions.

## Core Concepts

- **`GEOGRAPHY` Data Type:** Represents a point, linestring, polygon, or a collection of these on the Earth's surface. 
- **Spherical Geometry:** BigQuery uses the **WGS84** reference system (EPSG:4326), treating the Earth as a sphere. Distances and areas are calculated using geodesic paths, not planar geometry. All coordinates MUST be in **longitude, latitude** order.

## Key Geospatial Functions

BigQuery supports a wide array of `ST_` (Spatial Type) functions.

### Constructors
- `ST_GEOGPOINT(longitude, latitude)`: Creates a point.
- `ST_GEOGFROMTEXT(wkt_string)`: Creates a geography from Well-Known Text (WKT).
- `ST_GEOGFROMGEOJSON(geojson_string)`: Creates a geography from GeoJSON.

### Accessors
- `ST_ASTEXT(geography)` / `ST_ASGEOJSON(geography)`: Convert `GEOGRAPHY` to string formats.
- `ST_X(point)` / `ST_Y(point)`: Extract longitude/latitude.
- `ST_NUMPOINTS(geography)`: Count vertices in a geography.

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
- `ST_UNION(geog1, geog2)`: Combines two geographies.
- `ST_INTERSECTION(geog1, geog2)`: Returns the overlapping geography.
- `ST_CENTROID(geog)`: Returns the geographic center.
- `ST_SIMPLIFY(geog, tolerance_meters)`: Simplifies a complex geometry, reducing vertices.

## Optimization & Best Practices

1. **Native Spatial Clustering (Critical):** Always cluster your tables by a `GEOGRAPHY` column for optimal performance in spatial joins and filters. BigQuery uses S2 indexing under the hood. *Note: You cannot partition on a `GEOGRAPHY` column.*
2. **Optimize Query Predicates:** To leverage spatial clustering, use specific predicate functions in your `WHERE` or `JOIN ON` clause (e.g., `ST_INTERSECTS`, `ST_DWITHIN`, `ST_CONTAINS`, `ST_COVERS`, `ST_WITHIN`). 
   - *Warning:* `ST_DISJOINT` does **not** benefit from spatial clustering.
3. **Data Types:** Always store spatial data using the native `GEOGRAPHY` data type rather than `STRING` or `BYTES` to enable clustering and fast processing.
4. **Simplify Geometries:** If high precision isn't required (e.g., for broad heatmaps), use `ST_SIMPLIFY` to reduce storage and speed up calculations.
5. **Join Order:** For spatial joins, prefer `ST_INTERSECTS` or `ST_DWITHIN` in the `JOIN ON` clause rather than filtering in a `WHERE` clause after a cross join. Ensure both tables are clustered for maximum performance.

## Data Ingestion (JSONL / GeoJSON)

BigQuery allows loading geospatial data directly from **newline-delimited JSON (JSONL)** files where each line is a valid GeoJSON Feature or geometry.

### Loading GeoJSON (JSONL Format) via `bq` CLI
When loading GeoJSON data, BigQuery can automatically parse the geometry if you specify the schema correctly or let it autodetect (though explicit schemas are safer). The column holding the GeoJSON geometry should be loaded as a `GEOGRAPHY` type.

1.  **Format:** Ensure your file is JSONL (newline-delimited JSON). Each line must be a flat JSON object where one of the keys contains a GeoJSON geometry.
2.  **Command:** Use `bq load` with the `--json_extension=GEOJSON` flag if the file contains GeoJSON features, or simply define the schema with a `GEOGRAPHY` column if each line is a standard JSON object containing a GeoJSON geometry string.

```bash
bq load \
  --source_format=NEWLINE_DELIMITED_JSON \
  --json_extension=GEOJSON \
  --autodetect \
  my_project:my_dataset.my_table \
  gs://my_bucket/my_data.jsonl
```

## Procedures

### 1. Creating a Spatially Clustered Table
When creating a table that contains geographic data, use the `CLUSTER BY` clause on the geography column.
```sql
CREATE TABLE `my_project.my_dataset.my_table`
(
  id INT64,
  name STRING,
  geom GEOGRAPHY
)
CLUSTER BY geom;
```

### 2. Updating Points from Lat/Lon
If loading data from a CSV where lat/lon are separate columns, you can create the geography post-load:
```sql
UPDATE `my_dataset.my_table`
SET geom = ST_GEOGPOINT(longitude, latitude)
WHERE geom IS NULL;
```

### 3. Performing an Efficient Spatial Join
Joining a table of points to a table of polygons to find which polygon contains each point.
```sql
SELECT 
  p.id AS point_id, 
  poly.name AS polygon_name
FROM `my_dataset.points_table` AS p
INNER JOIN `my_dataset.polygons_table` AS poly
  ON ST_INTERSECTS(p.geom, poly.geom); -- Uses clustering effectively
```
