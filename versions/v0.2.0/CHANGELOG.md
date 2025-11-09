# OTS v0.2.0 Changelog

## New Features

### User-Defined Functions (UDFs) Support

**Added `source_functions` field to SQL transformation code structure**

- **Field**: `source_functions` (optional array of strings)
- **Purpose**: Track user-defined functions (UDFs) called in SQL transformations
- **Location**: `code.sql.source_functions`
- **Format**: Array of function names (preferably fully qualified: `schema.function_name`)

### Changes to SQL Transformation Structure

The `code.sql` object now includes:

```yaml
code:
  sql:
    original_sql: string
    resolved_sql: string
    source_tables: [string]      # Existing field
    source_functions: [string]   # NEW in v0.2.0
```

### Benefits

1. **Dependency Analysis**: Enables accurate dependency graph building including function dependencies
2. **Execution Order**: Ensures functions are created before transformations that use them
3. **Validation**: Allows tools to verify all required functions exist before execution
4. **Function Chains**: Supports function-to-function dependencies

### Backward Compatibility

- `source_functions` is **optional** - existing v0.1.0 modules remain valid
- If omitted, tools should assume an empty array `[]`
- All examples in v0.2.0 include `source_functions: []` for transformations without function dependencies

### Functions Array in OTS Modules

**Added `functions` array to OTS Module structure**

- **Field**: `functions` (optional array of function definitions)
- **Purpose**: Define user-defined functions (UDFs) that can be used in transformations
- **Location**: Top-level in OTS Module (same level as `transformations`)
- **Structure**: Array of function definitions with `function_id`, `function_type`, `language`, `code`, `parameters`, `return_type`, `deterministic`, `dependencies`, and `metadata`

**Function Execution Order:**
- Functions are created before transformations in dependency order
- Function-to-function dependencies are resolved automatically
- Execution order: Seeds → Functions → Transformations

**Function Overloading:**
- OTS 0.2.0 supports function overloading (multiple functions with same name, different signatures)
- Functions are identified by fully qualified name and parameter signature
- Each overloaded function is tracked separately in the `functions` array

### Test Library Structure Updates

**Added `ots_version` field to Test Library structure**

- **Field**: `ots_version` (required string)
- **Purpose**: Indicates which version of the OTS standard the test library follows
- **Location**: Top-level in Test Library file
- **Format**: OTS version string (e.g., "0.2.0")

### Documentation Updates

- Added new section: "User-Defined Functions (UDFs)" with examples
- Updated all code examples to include `source_functions` field
- Added example transformation using UDFs
- Added `functions` array definition to OTS Module structure
- Updated test library examples to include `ots_version` field

## Migration from v0.1.0

To migrate an OTS Module from v0.1.0 to v0.2.0:

1. Update `ots_version` from `"0.1.0"` to `"0.2.0"`
2. Add `source_functions: []` to all `code.sql` objects (if no functions are used)
3. If transformations use UDFs, populate `source_functions` with function names

Example migration:

```yaml
# v0.1.0
code:
  sql:
    original_sql: "..."
    resolved_sql: "..."
    source_tables: ["table1"]

# v0.2.0
code:
  sql:
    original_sql: "..."
    resolved_sql: "..."
    source_tables: ["table1"]
    source_functions: []  # Add this line
```

