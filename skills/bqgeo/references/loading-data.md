# Loading Geospatial Data into BigQuery

BigQuery supports loading several geospatial data formats using the `bq load` command. The primary formats are **WKT (Well-Known Text)**, **WKB (Well-Known Binary)**, **GeoJSON**, and **GeoParquet**.

## Core Requirement: Schema Definition
When loading geospatial data (except for GeoParquet), **you must explicitly define the schema** and specify the `GEOGRAPHY` data type for the spatial column. Schema auto-detection will default spatial strings (like WKT) to `STRING`.

## 1. Loading GeoJSON (JSONL Format)
BigQuery can load newline-delimited JSON (JSONL) where each line is a GeoJSON Feature or a JSON object containing a GeoJSON geometry.

**Format Requirements**:
- Must be `NEWLINE_DELIMITED_JSON`.
- Use the `--json_extension=GEOJSON` flag if the file strictly contains GeoJSON features.
- Ensure the schema maps the geometry field to `GEOGRAPHY`.

**Example Command**:
```bash
bq load \
  --source_format=NEWLINE_DELIMITED_JSON \
  --json_extension=GEOJSON \
  my_project:my_dataset.my_table \
  gs://my_bucket/my_data.jsonl \
  schema.json
```
*(Where `schema.json` defines the geometry field as `"type": "GEOGRAPHY"`).*

## 2. Loading WKT or WKB from CSV
If you have a CSV file where one column contains WKT strings or hex-encoded WKB, you can load it directly by specifying the `GEOGRAPHY` type in the inline schema.

**Example CSV (`data.csv`)**:
```csv
id,geom
1,"POINT(-122.3491 47.6205)"
2,"LINESTRING(-122.3491 47.6205, -122.3501 47.6215)"
```

**Example Command**:
```bash
bq load \
  --source_format=CSV \
  --skip_leading_rows=1 \
  my_project:my_dataset.my_table \
  ./data.csv \
  id:INTEGER,geom:GEOGRAPHY
```

## 3. Post-Load Transformation (Lat/Lon Columns)
If your data only contains separate `longitude` and `latitude` columns (e.g., in a standard CSV), load them as `FLOAT64` first. Then, use a SQL query with `ST_GEOGPOINT` to create the spatial column.

**Example SQL**:
```sql
CREATE TABLE `my_dataset.new_table` AS
SELECT 
  *, 
  ST_GEOGPOINT(longitude, latitude) AS geom
FROM `my_dataset.loaded_table`;
```

## 4. Loading GeoParquet
GeoParquet is supported natively by BigQuery. When loading GeoParquet files, schema detection works automatically, and you can optionally use the `--geography_metadata` flag to handle specific Coordinate Reference Systems (CRS) or metadata embedded in the file.
