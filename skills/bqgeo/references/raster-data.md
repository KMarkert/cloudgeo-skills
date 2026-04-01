# BigQuery Raster Data Reference

BigQuery supports raster data analysis primarily through its integration with **Google Earth Engine (GEE)**, allowing users to perform zonal statistics and access analysis-ready datasets.

## Native BigQuery Raster Functions
The primary native function for analyzing raster data within BigQuery is `ST_RegionStats`.

*   **`ST_RegionStats(geography, raster_dataset_id)`**
    *   **Description**: Derives zonal statistics (e.g., mean, min, max, sum) from a specified raster dataset within a given `GEOGRAPHY` boundary (polygon).
    *   **Use Case**: Calculating average elevation, total precipitation, or land cover summaries within a specific region.
    *   **Data Access**: Raster datasets are accessed via **BigQuery Sharing** (formerly Analytics Hub), linking directly to Earth Engine assets.
    *   **Note**: This function does not return raw pixel values; it returns aggregated statistics for the region defined by the `GEOGRAPHY` polygon.

## Third-Party Ecosystem (CARTO & RaQuet)
For more advanced pixel-level operations (like band math, point-in-pixel extraction, or custom tiling), the ecosystem relies on extensions:

1.  **RaQuet Specification**: An open standard for storing raster data in Apache Parquet files, treating each tile as a row and bands as columns, enabling querying by standard SQL engines.
2.  **CARTO Analytics Toolbox**: Provides extended spatial SQL functions not native to BigQuery:
    *   `ST_RasterValue(block, band, point, metadata)`: Extracts a single pixel value at a given `POINT`.
    *   `ST_RasterSummaryStats(band, metadata)`: Provides per-tile summary statistics.
    *   `ST_Clip(band, block, geometry, metadata)`: Extracts the pixels falling within a specified geometry.

## Visualization
BigQuery Studio (via the Google Cloud Console) includes native geospatial visualization capabilities. When querying raster data or using `ST_RegionStats`, you can view the resulting geographies and associated raster layers directly on the built-in map, and inspect layer metadata via the **Image Details** tab.
