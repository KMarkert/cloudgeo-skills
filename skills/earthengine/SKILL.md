---
name: earthengine
description: Manage Google Earth Engine assets, tasks, and data ingestion using the 'earthengine' CLI. Use when the user asks to upload data, view tasks, list GEE assets, or manage Earth Engine storage.
---

# Earth Engine CLI Skill

This skill provides guidance for using the `earthengine` Command Line Interface to manage Google Earth Engine (GEE) assets, upload data, and monitor tasks.

## Quick Start / Core Operations

When executing operations, prefer silent flags where applicable to keep the output manageable, and paginate if necessary.

### 1. Authentication
Ensure the user is authenticated before attempting operations:
```bash
earthengine authenticate
```

### 2. Project Context
For any operation that fails due to missing project context, you may need to specify the Cloud project using `--project` flag or by setting it as default:
```bash
earthengine set_project <project_id>
```

### 3. Listing Assets (`ls`)
List the contents of a GEE folder or ImageCollection. Be careful with large collections—append a prefix if searching for specific assets.
```bash
earthengine ls users/username/my_folder
earthengine ls projects/my-project/assets/my_collection
```

### 4. Asset Management
- **Create a folder or collection:** `earthengine create folder users/username/new_folder`
- **Copy an asset:** `earthengine cp <src_asset> <dst_asset>`
- **Move/Rename an asset:** `earthengine mv <src_asset> <dst_asset>`
- **Delete an asset:** `earthengine rm <asset_path>` (for recursive deletion, use `-r` flag)
- **Check size:** `earthengine du <folder_or_collection>`

### 5. Access Control (ACL)
Prints or updates the access control list (ACL) of an asset.
- **View ACL:** `earthengine acl get <asset>`
- **Set ACL:** `earthengine acl set <public|private> <asset>`

### 6. Data Ingestion (Upload)
Use the CLI to upload images (raster) or tables (vector like Shapefiles). 

- **Upload Image (GeoTIFF):**
  ```bash
  earthengine upload image --asset_id=users/username/my_image path/to/image.tif
  ```
- **Upload Table (Shapefile):**
  *Note: Shapefiles require the `.shp`, `.shx`, `.dbf`, and `.prj` files to be passed together or zipped.*
  ```bash
  earthengine upload table --asset_id=users/username/my_table path/to/shapefile.shp
  ```

### 7. Manifest-Based Uploads
Manifests are JSON files used for complex uploads (e.g., mosaicking tiles, stacking bands, or defining specific table parsing rules).

#### Workflow for Manifest Uploads
1. **Identify Source Files:** List all files (local or GCS URIs) intended for the asset.
2. **Generate JSON Manifest:** Create a temporary `.json` file with the required structure.
3. **Execute Upload:** Run the manifest upload command and capture the Task ID.

#### Image Manifest Template
Use this for mosaicking multiple Cloud Storage files into one asset:
```json
{
  "name": "projects/<project-id>/assets/<asset-id>",
  "tilesets": [
    {
      "sources": [
        { "uris": ["gs://bucket/tile1.tif", "gs://bucket/tile2.tif"] }
      ]
    }
  ],
  "bands": [{ "id": "B1" }]
}
```
**Command:** `earthengine upload image --manifest /path/to/manifest.json`

#### Table Manifest Template
Use this for specific CSV parsing (e.g., delimiters, lat/long columns):
```json
{
  "name": "projects/<project-id>/assets/<asset-id>",
  "sources": [
    {
      "uris": ["gs://bucket/data.csv"],
      "csvDelimiter": ",",
      "xColumn": "longitude",
      "yColumn": "latitude"
    }
  ]
}
```
**Command:** `earthengine upload table --manifest /path/to/manifest.json`

### 8. Task Monitoring
Uploading data to Earth Engine creates an asynchronous ingestion task. Use the `task` command to monitor its progress.
- **List tasks:** `earthengine task list`
- **Task details:** `earthengine task info <task_id>`
- **Cancel task:** `earthengine task cancel <task_id>`

## Best Practices
- **Token Efficiency:** If `earthengine ls` or `earthengine task list` returns hundreds of lines, pipe the output through `head` or `grep` to avoid overwhelming the context window (e.g., `earthengine task list | head -n 20`).
- **Asset IDs:** Always verify the correct absolute asset ID (e.g., `projects/my-project/assets/my_image` or `users/username/my_asset`). Do not guess asset paths without using `ls` to verify the parent directory first.
- **Handling Errors:** If an upload fails, check the task error output using `earthengine task info <task_id>`. Common errors include missing projections or invalid data types.