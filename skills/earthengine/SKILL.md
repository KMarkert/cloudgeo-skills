---
name: earthengine
description: "Expert guidance for Google Earth Engine (GEE) CLI. Use this skill when the user wants to manage assets, tasks, or interact with GEE from the command line."
version: 1.0.0
---

# Google Earth Engine (GEE) Skill

## Overview
This skill provides guidance and best practices for interacting with Google Earth Engine using the `earthengine` command-line tool.

## Core Commands

### Asset Management
- **List Assets:** `earthengine ls [parent_asset_id]`
- **Create Folder:** `earthengine create folder [asset_id]`
- **Delete Asset:** `earthengine rm -r [asset_id]` (recursive)
- **Set Metadata:** `earthengine asset set --property name=value [asset_id]`

### Task Management
- **List Tasks:** `earthengine task list`
- **Cancel Task:** `earthengine task cancel [task_id]`
- **Task Status:** `earthengine task wait [task_id]`

### Data Ingestion
- **Upload Image:** `earthengine upload image --asset_id [id] [source_url]`
- **Upload Table:** `earthengine upload table --asset_id [id] [source_url]`

## Best Practices
1. **Asset IDs:** Always use full asset IDs (e.g., `projects/my-project/assets/my-asset`).
2. **Batching:** When uploading multiple assets, use scripts to loop through files rather than manual commands.
3. **Task Monitoring:** Always check the status of long-running tasks using `earthengine task list`.
4. **Recursive Deletion:** Be cautious with `earthengine rm -r` as it deletes all children within a folder or image collection.

## Procedures

### Uploading a COG (Cloud Optimized GeoTIFF)
1. Ensure the file is accessible in a Google Cloud Storage bucket.
2. Use `earthengine upload image --asset_id <id> gs://<bucket>/<file>.tif`.
3. Monitor the task with `earthengine task list`.

### Cleaning up a Project
1. List all assets: `earthengine ls projects/<project-id>/assets`.
2. Identify unused assets.
3. Remove them using `earthengine rm`.
