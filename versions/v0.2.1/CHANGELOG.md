# OTS v0.2.1 Changelog

## New Features

### Schema Change Handling for Incremental Materialization

**Added `on_schema_change` field to incremental materialization configuration**

- **Field**: `on_schema_change` (optional string)
- **Purpose**: Control how schema differences between transformation output and existing target table are handled
- **Location**: `materialization.incremental_details.on_schema_change`
- **Default**: `"fail"` (fail transformation if schema changes detected)
- **Options**:
  - `"fail"`: Fail the transformation if any schema differences are detected (default)
  - `"ignore"`: Ignore schema differences and proceed (may cause errors if columns don't match)
  - `"append_new_columns"`: Automatically add new columns, keep existing columns
  - `"sync_all_columns"`: Add new columns and remove missing columns (may cause data loss)
  - `"full_refresh"`: Drop and recreate table with full transformation output
  - `"full_incremental_refresh"`: Drop, recreate, then run incremental strategy in chunks
  - `"recreate_empty"`: Drop and recreate as empty table (for external backfilling)

### Full Incremental Refresh Configuration

**Added `full_incremental_refresh` field for chunked incremental execution**

- **Field**: `full_incremental_refresh` (optional object)
- **Purpose**: Configure parameter-based incremental chunking after table recreation
- **Location**: Top-level in transformation definition (same level as `materialization`)
- **Required when**: `on_schema_change` is set to `"full_incremental_refresh"`

**Structure:**
```yaml
full_incremental_refresh:
  parameters:
    - name: string         # Parameter name (matches placeholder, e.g., "@start_date")
      start_value: string  # Initial value for the parameter
      end_value: string    # End condition: hardcoded value or expression evaluated against source table (e.g., "max(event_date)" from source.events)
      step: string         # Increment step (SQL interval or numeric value)
```

**Use Cases:**
- Single parameter: One parameter in the array (e.g., `@start_date`)
- Multiple parameters: Multiple parameters for boundary-based queries (e.g., `@start_date` and `@end_date`)
- Both parameters use the same `step` value (ideally), but can differ if needed
- Parameter names must match placeholders in the query (e.g., `"@start_date"` matches `'@start_date'` in SQL)

### Schema Change Behavior Details

**Type Mismatches:**
- Columns with same name but different data type are treated as schema changes
- With `on_schema_change="fail"`, type mismatches cause immediate failure
- Different data type = different column (fail immediately)

**Column Order:**
- Column order differences are detected and logged as warnings
- Tools should rely on explicit column lists in INSERT/MERGE statements
- No automatic reordering is performed

**Schema Comparison:**
- Schema comparison happens after time-based filtering but before execution
- Uses database-specific `DESCRIBE` or equivalent methods for precision
- If table doesn't exist, schema comparison is skipped and table is created normally

## Changes to Incremental Materialization Structure

The `incremental_details` object now includes:

```yaml
incremental_details:
  strategy: string
  delete_condition: string
  filter_condition: string
  merge_key: [string]
  update_columns: [string]
  on_schema_change: string  # NEW in v0.2.1
```

The transformation definition now supports:

```yaml
materialization:
  type: "incremental"
  incremental_details: {...}

full_incremental_refresh:  # NEW in v0.2.1 (optional)
  parameters: [...]
```

## Backward Compatibility

- `on_schema_change` is **optional** - existing v0.2.0 modules remain valid
- If omitted, default behavior is `"fail"` (matches previous implicit behavior)
- `full_incremental_refresh` is **optional** - only required when `on_schema_change="full_incremental_refresh"`
- All existing v0.2.0 examples remain valid in v0.2.1

## Migration from v0.2.0

To migrate an OTS Module from v0.2.0 to v0.2.1:

1. Update `ots_version` from `"0.2.0"` to `"0.2.1"`
2. Optionally add `on_schema_change` to `incremental_details` if you want explicit schema change handling
3. If using `full_incremental_refresh`, add the `full_incremental_refresh` configuration

Example migration:

```yaml
# v0.2.0
materialization:
  type: "incremental"
  incremental_details:
    strategy: "append"
    filter_condition: "created_at >= '@start_date'"

# v0.2.1 (explicit default)
materialization:
  type: "incremental"
  incremental_details:
    strategy: "append"
    filter_condition: "created_at >= '@start_date'"
    on_schema_change: "fail"  # Optional: explicit default

# v0.2.1 (with schema change handling)
materialization:
  type: "incremental"
  incremental_details:
    strategy: "append"
    filter_condition: "created_at >= '@start_date'"
    on_schema_change: "append_new_columns"  # Auto-add new columns

# v0.2.1 (with full incremental refresh)
materialization:
  type: "incremental"
  incremental_details:
    strategy: "append"
    filter_condition: "event_date >= '@start_date'"
    on_schema_change: "full_incremental_refresh"

full_incremental_refresh:
  parameters:
    - name: "@start_date"
      start_value: "2024-01-01"
      end_value: "max(event_date)"  # Evaluated against source table
      step: "INTERVAL 1 DAY"
```

## Documentation Updates

- Added "Schema Change Handling" section to Incremental Materialization documentation
- Added examples for all `on_schema_change` options
- Added `full_incremental_refresh` configuration examples
- Updated append strategy example to show `on_schema_change` usage
- Added documentation for type mismatch and column order handling


