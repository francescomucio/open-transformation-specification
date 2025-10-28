# Open Transformation Specification v0.1.0

## Table of Contents

1. [Introduction](#introduction)
2. [Specification Structure](#specification-structure)
3. [Core Concepts](#core-concepts)
4. [Open Transformation Definition (OTD) Structure](#open-transformation-definition-otd-structure)
5. [Materialization Types](#materialization-types)
6. [Examples](#examples)

## Introduction

The Open Transformation Specification (OTS) defines a standard, programming language-agnostic interface description for data transformations. This specification allows both humans and computers to discover and understand how transformations behave, what outputs they produce, and how those outputs are materialized (as tables, views, incremental updates, SCD2, etc.) without requiring additional documentation or configuration.

An OTS-based transformation must include both the code that transforms the data and metadata about the transformation. A tool implementing OTS should be able to execute an OTS transformation with no additional code or information beyond what's specified in the OTS document.

## Specification Structure

An Open Transformation Specification document describes a data transformation and is represented in YAML or JSON formats. These documents may be produced and served statically or generated dynamically from an application.

The Open Transformation Specification does not require rewriting existing data transformations. It does not require binding any software to a transformation – the described transformation may not even be owned by the creator of its description. It does, however, require that the transformation's capabilities be described in the structure of the Open Transformation Specification.

## Core Concepts

### Open Transformation Definition (OTD)

An **Open Transformation Definition (OTD)** is a concrete instance of the Open Transformation Specification - a single file or document that describes a specific data transformation using the OTS format. 

A transformation is a unit of data processing that takes one or more data sources as input and produces one data output. Transformations can be SQL queries, Python functions resulting in a sqlglot expression.

### Open Transformation Specification Module

An **Open Transformation Specification Module (OTS Module)** is a collection of related transformations that target the same database and schema. An OTS Module can contain one or more transformations, much like how an OpenAPI specification can contain multiple endpoints.

Key characteristics of an OTS Module:
- **Single target**: All transformations in a module target the same database and schema
- **Logical grouping**: Related transformations are organized together
- **Deployment unit**: The entire module can be deployed as a single unit
- **Dependency management**: Transformations within a module can reference each other

#### OTS vs OTD vs OTS Module

- **Open Transformation Specification (OTS)**: The standard that defines the structure and rules
- **Open Transformation Definition (OTD)**: A specific transformation within a module
- **Open Transformation Specification Module (OTS Module)**: A collection of related transformations targeting the same database and schema

Think of it this way: OTS is like the blueprint, an OTS Module is the house (a complete set of transformations), and each OTD is a room within that house (an individual transformation).

## Components of an OTD

An Open Transformation Definition consists of several key components that work together to define an executable transformation:

1. **Transformation Code**: The transformation logic as a sqlglot expression (for SQL transformations)
2. **Schema Definition**: The structure of the output data including column definitions, types, and validation rules
3. **Materialization Strategy**: How the output is stored and updated (table, view, incremental, SCD2)
4. **Tests**: Validation rules that ensure data quality at table level
5. **Metadata**: Additional information about the transformation (owner, tags, creation date, etc.)

### Transformation Code

The transformation code is stored as a sqlglot expression. sqlglot provides a unified representation of SQL that can be parsed from various dialects (PostgreSQL, MySQL, BigQuery, Spark, etc.) and enables automatic dependency analysis, validation, and cross-database execution.

A sqlglot expression contains:
- `sql_content`: The original SQL query
- `qualified_sql`: SQL with fully qualified table names (schema.table)
- `sql_type`: Type of SQL operation (select, union, insert, etc.)
- `tables`: List of source tables referenced in the query
- `columns`: List of output columns
- `functions`: SQL functions used in the query
- `aliases`: Column aliases defined in the query

### Schema Definition

Schema defines the structure of the output data, including column names, data types, descriptions, partitioning, indexes, and other properties of the physical table. The schema is essential for understanding what the transformation produces without executing it. For example, it enables generating DDL statements for creating the output table.

### Materialization Strategy
Materialization defines how the transformation output is stored and updated. Common types include:
- **table**: Full table replacement on each run
- **view**: Virtual table that queries underlying data
- **incremental**: Partial updates using strategies like delete+insert or merge
- **scd2**: Slowly Changing Dimension type 2 for tracking historical changes

### Tests
Tests are validation rules that ensure data quality. They can be defined at two levels:
- **Column-level tests**: Applied to individual columns (e.g., `not_null`, `unique`)
- **Table-level tests**: Applied to the entire output (e.g., `row_count_gt_0`, `no_duplicates`)

Tests enable automated data quality validation without manual inspection.

### Metadata
Metadata provides additional information about the transformation including:
- **file_path**: Location of the source transformation file
- **owner**: Person or team responsible for the transformation
- **tags**: Keywords for categorization and discovery

## OTS Module Structure

An OTS Module is a YAML or JSON document that can contain one or more transformations. Below is the complete structure:

### Complete OTS Module Structure

```yaml
# OTS version
ots_version: string             # OTS specification version (e.g., "0.1.0")

# Module metadata
module_name: string              # Module name (e.g., "ecommerce_analytics")
module_description: string       # Description of the module (optional)
version: string                  # Optional: Module version (e.g., "1.0.0")
tags: [string]                   # Optional: Tags for categorization (e.g., ["analytics", "fct"])
target:
  database: string               # Target database name
  schema: string                 # Target schema name
  sql_dialect: string            # Optional: SQL dialect (e.g., "postgres", "bigquery", "snowflake", "spark", etc.)
  connection_profile: string     # Optional: connection profile reference

# Transformations
transformations:                 # Array of transformation definitions
  - transformation_id: string    # Fully qualified identifier (e.g., "analytics.my_first_table")
    description: string          # Optional: Description of what the transformation does (optional)
    sql_dialect: string          # Optional: SQL dialect of the transformation code (for translation to target dialect)
    
    # Transformation code as sqlglot expression
    sqlglot:
      sql_content: string        # The original SQL query
      parsed_ast: string         # Parsed AST representation (optional)
      qualified_sql: string      # SQL with fully qualified table names (schema.table)
      sql_type: string           # Type of SQL operation ("select", "union", "insert", etc.)
      tables: [string]           # List of input tables referenced
      columns: [string]          # List of output columns
      functions: [string]        # List of SQL functions used
      aliases: [string]          # List of column aliases
    
    # Schema definition
    schema:
      columns:                   # Array of column definitions
        - name: string           # Column name
          datatype: string       # Data type ("number", "string", "date", etc.)
          description: string    # Column description
      partitioning: [string]     # Optional: Partition keys
      indexes:                   # Optional: Array of index definitions
        - name: string           # Index name (optional, auto-generated if not provided)
          columns: [string]      # Columns to index
    
    # Materialization strategy
    materialization:
      type: string               # "table", "view", "incremental", "scd2"
      incremental_details:       # Required if type is "incremental"
        strategy: string         # "delete_insert", "append", "merge"
        delete_condition: string # SQL condition for delete (delete_insert only)
        filter_condition: string # SQL condition for filtering data
        merge_key: [string]      # Primary key columns for matching records (merge only)
        update_columns: [string] # (Optional) List of columns to be updated in merge strategy
      scd2_details:              # Optional if type is "scd2"
        start_column: string     # Name of the start column (default: "valid_from")
        end_column: string       # Name of the end column (default: "valid_to")
        unique_key: [string]     # Array of columns that uniquely identify a record in SCD2 modeling (optional)
    
    # Tests: both column-level and table-level
    tests:
      columns:                   # Optional: Column-level tests
        column_name: [string]    # Tests for specific columns (e.g., "id": ["not_null", "unique"])
      table: [string]            # Optional: Table-level tests (e.g., ["row_count_gt_0", "no_duplicates"])
    
    # Metadata
    metadata:
      file_path: string          # Path to the source transformation file
      owner: string              # Optional: Person or team responsible (optional)
      tags: [string]             # Optional: Keywords for categorization (optional)
```

## Simple Table Transformation

<details>
<summary><strong>JSON Format</strong></summary>

```json
{
  "ots_version": "0.1.0",
  "module_name": "analytics_customers",
  "module_description": "Customer analytics transformations",
  "target": {
    "database": "warehouse",
    "schema": "analytics",
    "sql_dialect": "postgres"
  },
  "transformations": [
    {
      "transformation_id": "analytics.customers",
      "description": "Customer data table",
      "sqlglot": {
        "sql_content": "SELECT id, name, email, created_at FROM source.customers WHERE active = true",
        "parsed_ast": "SELECT id, name, email, created_at FROM source.customers WHERE active = true",
        "qualified_sql": "SELECT id, name, email, created_at FROM warehouse.source.customers WHERE active = true",
        "sql_type": "select",
        "tables": ["source.customers"],
        "columns": ["id", "name", "email", "created_at", "active"],
        "functions": [],
        "aliases": []
      },
      "schema": {
        "columns": [
          {
            "name": "id",
            "datatype": "number",
            "description": "Unique customer identifier"
          },
          {
            "name": "name",
            "datatype": "string",
            "description": "Customer name"
          },
          {
            "name": "email",
            "datatype": "string",
            "description": "Customer email address"
          },
          {
            "name": "created_at",
            "datatype": "date",
            "description": "Customer creation date"
          }
        ],
        "partitioning": [],
        "indexes": [
          {
            "name": "idx_customers_id",
            "columns": ["id"]
          },
          {
            "name": "idx_customers_email",
            "columns": ["email"]
          }
        ]
      },
      "materialization": {
        "type": "table"
      },
      "tests": {
        "columns": {
          "id": ["not_null", "unique"],
          "email": ["not_null", "unique"],
          "created_at": ["not_null"]
        },
        "table": ["row_count_gt_0"]
      },
      "metadata": {
        "file_path": "/models/analytics/customers.sql",
        "owner": "data-team",
        "tags": ["customer", "core"]
      }
    }
  ]
}
```

</details>

<details>
<summary><strong>YAML Format</strong></summary>

```yaml
ots_version: "0.1.0"
module_name: "analytics_customers"
module_description: "Customer analytics transformations"
target:
  database: "warehouse"
  schema: "analytics"
  sql_dialect: "postgres"

transformations:
  - transformation_id: "analytics.customers"
    description: "Customer data table"
    
    sqlglot:
      sql_content: "SELECT id, name, email, created_at FROM source.customers WHERE active = true"
      parsed_ast: "SELECT id, name, email, created_at FROM source.customers WHERE active = true"
      qualified_sql: "SELECT id, name, email, created_at FROM warehouse.source.customers WHERE active = true"
      sql_type: "select"
      tables: ["source.customers"]
      columns: ["id", "name", "email", "created_at", "active"]
      functions: []
      aliases: []
    
    schema:
      columns:
        - name: "id"
          datatype: "number"
          description: "Unique customer identifier"
        - name: "name"
          datatype: "string"
          description: "Customer name"
        - name: "email"
          datatype: "string"
          description: "Customer email address"
        - name: "created_at"
          datatype: "date"
          description: "Customer creation date"
      partitioning: []
      indexes:
        - name: "idx_customers_id"
          columns: ["id"]
        - name: "idx_customers_email"
          columns: ["email"]
    
    materialization:
      type: "table"
    
    tests:
      columns:
        id: ["not_null", "unique"]
        email: ["not_null", "unique"]
        created_at: ["not_null"]
      table: ["row_count_gt_0"]
    
  metadata:
      file_path: "/models/analytics/customers.sql"
      owner: "data-team"
      tags: ["customer", "core"]
```

</details>

## Materialization Types

### Incremental Materialization

Incremental materialization updates only changed data using one of three strategies:
- **delete_insert**: Deletes rows matching a condition and inserts new data
- **append**: Simply appends new data without removing existing rows
- **merge**: Performs an upsert operation using a merge statement

#### Delete-Insert Strategy

```yaml
materialization:
  type: "incremental"
  incremental_details:
    strategy: "delete_insert"
    delete_condition: "to_date(updated_ts) = '@start_date'" 
    filter_condition: "to_date(updated_ts) = '@start_date'"
```

#### Append Strategy

```yaml
materialization:
  type: "incremental"
  incremental_details:
    strategy: "append"
    filter_condition: "created_at >= '@start_date'"
```

#### Merge Strategy

```yaml
materialization:
  type: "incremental"
  incremental_details:
    strategy: "merge"
    merge_key: "customer_id"      # Primary key or unique identifier for matching
    filter_condition: "updated_at >= '@start_date'"
    update_columns: ["name", "email"]  # Optional: specific columns to update
```

### SCD2 Materialization

SCD2 (Slowly Changing Dimension Type 2) materialization tracks historical changes with valid date ranges. Requires a unique key to identify records.

```yaml
materialization:
  type: "scd2"
  scd2_details:
    unique_key: ["product_id"]    # Primary key or unique identifier
    start_column: "valid_from"    # Optional, defaults to "valid_from"
    end_column: "valid_to"        # Optional, defaults to "valid_to"
```

### Schema Column Definition

A schema column in an OTD defines the structure and properties of a single column in the output table:

```yaml
columns:
  - name: string              # Column name
    datatype: string          # Data type ("number", "string", "date", etc.)
    description: string        # Column description
```

**Common Data Types:**
- `number`: Numeric values
- `string`: Text values  
- `date`: Date and timestamp values
- `boolean`: True/false values
- `array`: Array of values
- `object`: Complex nested objects

### Test Types

**Column-level tests** are applied to specific columns:
- `not_null`: Field cannot be null
- `unique`: Field values must be unique

**Table-level tests** are applied to the entire output:
- `row_count_gt_0`: Table must have at least one row
- `no_duplicates`: No duplicate rows allowed

## Complete Examples: Incremental Strategies

### Delete-Insert Example

<details>
<summary><strong>YAML Format</strong></summary>

```yaml
ots_version: "0.1.0"
transformation_id: "analytics.recent_orders"
description: "Orders updated in the last 7 days"

sqlglot:
  sql_content: "SELECT order_id, customer_id, order_date, amount, status FROM source.orders WHERE updated_at >= '@start_date'"
  sql_type: "select"
  tables: ["source.orders"]
  columns: ["order_id", "customer_id", "order_date", "amount", "status"]

schema:
  columns:
    - name: "order_id"
      datatype: "number"
      description: "Unique order identifier"
    - name: "customer_id"
      datatype: "number"
      description: "Customer ID"
    - name: "order_date"
      datatype: "date"
      description: "Order date"
    - name: "amount"
      datatype: "number"
      description: "Order amount"
    - name: "status"
      datatype: "string"
      description: "Order status"
  partitioning: ["order_date"]
  indexes:
    - name: "idx_order_id"
      columns: ["order_id"]

materialization:
  type: "incremental"
  incremental_details:
    strategy: "delete_insert"
    delete_condition: "to_date(updated_at) = '@start_date'"
    filter_condition: "to_date(updated_at) = '@start_date'"
    
tests:
  columns:
    order_id: ["not_null", "unique"]
    order_date: ["not_null"]
  table: ["row_count_gt_0"]

  metadata:
  file_path: "/models/analytics/recent_orders.sql"
  owner: "analytics-team"
  tags: ["orders", "incremental"]
```

</details>

<details>
<summary><strong>JSON Format</strong></summary>

```json
{
  "ots_version": "0.1.0",
  "transformation_id": "analytics.recent_orders",
  "description": "Orders updated in the last 7 days",
  
  "sqlglot": {
    "sql_content": "SELECT order_id, customer_id, order_date, amount, status FROM source.orders WHERE updated_at >= '@start_date'",
    "sql_type": "select",
    "tables": ["source.orders"],
    "columns": ["order_id", "customer_id", "order_date", "amount", "status"],
    "functions": [],
    "aliases": []
  },
  
  "schema": {
    "columns": [
      {
        "name": "order_id",
        "datatype": "number",
        "description": "Unique order identifier"
      },
      {
        "name": "customer_id",
        "datatype": "number",
        "description": "Customer ID"
      },
      {
        "name": "order_date",
        "datatype": "date",
        "description": "Order date"
      },
      {
        "name": "amount",
        "datatype": "number",
        "description": "Order amount"
      },
      {
        "name": "status",
        "datatype": "string",
        "description": "Order status"
      }
    ],
    "partitioning": ["order_date"],
    "indexes": [
      {
        "name": "idx_order_id",
        "columns": ["order_id"]
      }
    ]
  },
  
  "materialization": {
    "type": "incremental",
    "incremental_details": {
      "strategy": "delete_insert",
      "delete_condition": "to_date(updated_at) = '@start_date'",
      "filter_condition": "to_date(updated_at) = '@start_date'"
    }
  },
  
  "tests": {
    "columns": {
      "order_id": ["not_null", "unique"],
      "order_date": ["not_null"]
    },
    "table": ["row_count_gt_0"]
  },
  
  "metadata": {
    "file_path": "/models/analytics/recent_orders.sql",
    "owner": "analytics-team",
    "tags": ["orders", "incremental"]
  }
}
```

</details>

### Append Example

<details>
<summary><strong>YAML Format</strong></summary>

```yaml
ots_version: "0.1.0"
transformation_id: "logs.event_stream"
description: "Append-only event log"

sqlglot:
  sql_content: "SELECT event_id, timestamp, user_id, event_type, payload FROM source.events WHERE timestamp >= '@start_date'"
  sql_type: "select"
  tables: ["source.events"]
  columns: ["event_id", "timestamp", "user_id", "event_type", "payload"]

schema:
  columns:
    - name: "event_id"
      datatype: "string"
      description: "Unique event identifier"
    - name: "timestamp"
      datatype: "date"
      description: "Event timestamp"
    - name: "user_id"
      datatype: "string"
      description: "User who triggered the event"
    - name: "event_type"
      datatype: "string"
      description: "Type of event"
    - name: "payload"
      datatype: "object"
      description: "Event payload data"
  partitioning: ["timestamp"]
  indexes:
    - name: "idx_timestamp"
      columns: ["timestamp"]
    - name: "idx_user_id"
      columns: ["user_id"]

materialization:
  type: "incremental"
  incremental_details:
    strategy: "append"
    filter_condition: "timestamp >= '@start_date'"

tests:
  columns:
    event_id: ["not_null", "unique"]
    timestamp: ["not_null"]
  table: ["row_count_gt_0"]

metadata:
  file_path: "/models/logs/event_stream.sql"
  owner: "data-engineering"
  tags: ["events", "append-only"]
```

</details>

<details>
<summary><strong>JSON Format</strong></summary>

```json
{
  "ots_version": "0.1.0",
  "transformation_id": "logs.event_stream",
  "description": "Append-only event log",
  
  "sqlglot": {
    "sql_content": "SELECT event_id, timestamp, user_id, event_type, payload FROM source.events WHERE timestamp >= '@start_date'",
    "sql_type": "select",
    "tables": ["source.events"],
    "columns": ["event_id", "timestamp", "user_id", "event_type", "payload"],
    "functions": [],
    "aliases": []
  },
  
  "schema": {
    "columns": [
      {
        "name": "event_id",
        "datatype": "string",
        "description": "Unique event identifier"
      },
      {
        "name": "timestamp",
        "datatype": "date",
        "description": "Event timestamp"
      },
      {
        "name": "user_id",
        "datatype": "string",
        "description": "User who triggered the event"
      },
      {
        "name": "event_type",
        "datatype": "string",
        "description": "Type of event"
      },
      {
        "name": "payload",
        "datatype": "object",
        "description": "Event payload data"
      }
    ],
    "partitioning": ["timestamp"],
    "indexes": [
      {
        "name": "idx_timestamp",
        "columns": ["timestamp"]
      },
      {
        "name": "idx_user_id",
        "columns": ["user_id"]
      }
    ]
  },
  
  "materialization": {
    "type": "incremental",
    "incremental_details": {
      "strategy": "append",
      "filter_condition": "timestamp >= '@start_date'"
    }
  },
  
  "tests": {
    "columns": {
      "event_id": ["not_null", "unique"],
      "timestamp": ["not_null"]
    },
    "table": ["row_count_gt_0"]
  },
  
  "metadata": {
    "file_path": "/models/logs/event_stream.sql",
    "owner": "data-engineering",
    "tags": ["events", "append-only"]
  }
}
```

</details>

### Merge Example

<details>
<summary><strong>YAML Format</strong></summary>

```yaml
ots_version: "0.1.0"
transformation_id: "product.master_data"
description: "Customer master data with upsert logic"

sqlglot:
  sql_content: "SELECT customer_id, name, email, phone, updated_at FROM source.customers WHERE updated_at >= '@start_date'"
  sql_type: "select"
  tables: ["source.customers"]
  columns: ["customer_id", "name", "email", "phone", "updated_at"]

schema:
  columns:
    - name: "customer_id"
      datatype: "number"
      description: "Unique customer identifier"
    - name: "name"
      datatype: "string"
      description: "Customer name"
    - name: "email"
      datatype: "string"
      description: "Customer email"
    - name: "phone"
      datatype: "string"
      description: "Customer phone number"
    - name: "updated_at"
      datatype: "date"
      description: "Last update timestamp"
  partitioning: []
  indexes:
    - name: "idx_customer_id"
      columns: ["customer_id"]
    - name: "idx_email"
      columns: ["email"]

materialization:
  type: "incremental"
  incremental_details:
    strategy: "merge"
    filter_condition: "updated_at >= '@start_date'"
    merge_key: ["customer_id"]
    update_columns: ["name", "email", "phone", "updated_at"]

tests:
  columns:
    customer_id: ["not_null", "unique"]
    email: ["not_null"]
  table: ["row_count_gt_0", "no_duplicates"]

  metadata:
  file_path: "/models/product/master_data.sql"
  owner: "product-team"
  tags: ["customers", "master-data"]
```

</details>

<details>
<summary><strong>JSON Format</strong></summary>

```json
{
  "ots_version": "0.1.0",
  "transformation_id": "product.master_data",
  "description": "Customer master data with upsert logic",
  
  "sqlglot": {
    "sql_content": "SELECT customer_id, name, email, phone, updated_at FROM source.customers WHERE updated_at >= '@start_date'",
    "sql_type": "select",
    "tables": ["source.customers"],
    "columns": ["customer_id", "name", "email", "phone", "updated_at"],
    "functions": [],
    "aliases": []
  },
  
  "schema": {
    "columns": [
      {
        "name": "customer_id",
        "datatype": "number",
        "description": "Unique customer identifier"
      },
      {
        "name": "name",
        "datatype": "string",
        "description": "Customer name"
      },
      {
        "name": "email",
        "datatype": "string",
        "description": "Customer email"
      },
      {
        "name": "phone",
        "datatype": "string",
        "description": "Customer phone number"
      },
      {
        "name": "updated_at",
        "datatype": "date",
        "description": "Last update timestamp"
      }
    ],
    "partitioning": [],
    "indexes": [
      {
        "name": "idx_customer_id",
        "columns": ["customer_id"]
      },
      {
        "name": "idx_email",
        "columns": ["email"]
      }
    ]
  },
  
  "materialization": {
    "type": "incremental",
    "incremental_details": {
      "strategy": "merge",
      "filter_condition": "updated_at >= '@start_date'",
      "merge_key": ["customer_id"],
      "update_columns": ["name", "email", "phone", "updated_at"]
    }
  },
  
  "tests": {
    "columns": {
      "customer_id": ["not_null", "unique"],
      "email": ["not_null"]
    },
    "table": ["row_count_gt_0", "no_duplicates"]
  },
  
  "metadata": {
    "file_path": "/models/product/master_data.sql",
    "owner": "product-team",
    "tags": ["customers", "master-data"]
  }
}
```

</details>

### SCD2 Example

<details>
<summary><strong>YAML Format</strong></summary>

```yaml
ots_version: "0.1.0"
transformation_id: "dim.products_scd2"
description: "Product dimension with full history tracking"

sqlglot:
  sql_content: "SELECT product_id, product_name, price, category, updated_at FROM source.products WHERE updated_at >= '@start_date'"
  sql_type: "select"
  tables: ["source.products"]
  columns: ["product_id", "product_name", "price", "category", "updated_at"]

schema:
  columns:
    - name: "product_id"
      datatype: "number"
      description: "Unique product identifier"
    - name: "product_name"
      datatype: "string"
      description: "Product name"
    - name: "price"
      datatype: "number"
      description: "Product price"
    - name: "category"
      datatype: "string"
      description: "Product category"
    - name: "updated_at"
      datatype: "date"
      description: "Last update timestamp"
    - name: "valid_from"
      datatype: "date"
      description: "Record validity start date"
    - name: "valid_to"
      datatype: "date"
      description: "Record validity end date"
  partitioning: []
  indexes:
    - name: "idx_product_id"
      columns: ["product_id"]
    - name: "idx_valid_from"
      columns: ["valid_from"]

materialization:
  type: "scd2"
  scd2_details:
    unique_key: ["product_id"]
    start_column: "valid_from"
    end_column: "valid_to"

tests:
  columns:
    product_id: ["not_null", "unique"]
    valid_from: ["not_null"]
  table: ["row_count_gt_0"]

  metadata:
  file_path: "/models/dim/products_scd2.sql"
  owner: "data-engineering"
  tags: ["products", "scd2", "dimension"]
```

</details>

<details>
<summary><strong>JSON Format</strong></summary>

```json
{
  "ots_version": "0.1.0",
  "transformation_id": "dim.products_scd2",
  "description": "Product dimension with full history tracking",
  
  "sqlglot": {
    "sql_content": "SELECT product_id, product_name, price, category, updated_at FROM source.products WHERE updated_at >= '@start_date'",
    "sql_type": "select",
    "tables": ["source.products"],
    "columns": ["product_id", "product_name", "price", "category", "updated_at"],
    "functions": [],
    "aliases": []
  },
  
  "schema": {
    "columns": [
      {
        "name": "product_id",
        "datatype": "number",
        "description": "Unique product identifier"
      },
      {
        "name": "product_name",
        "datatype": "string",
        "description": "Product name"
      },
      {
        "name": "price",
        "datatype": "number",
        "description": "Product price"
      },
      {
        "name": "category",
        "datatype": "string",
        "description": "Product category"
      },
      {
        "name": "updated_at",
        "datatype": "date",
        "description": "Last update timestamp"
      },
      {
        "name": "valid_from",
        "datatype": "date",
        "description": "Record validity start date"
      },
      {
        "name": "valid_to",
        "datatype": "date",
        "description": "Record validity end date"
      }
    ],
    "partitioning": [],
    "indexes": [
      {
        "name": "idx_product_id",
        "columns": ["product_id"]
      },
      {
        "name": "idx_valid_from",
        "columns": ["valid_from"]
      }
    ]
  },
  
  "materialization": {
    "type": "scd2",
    "scd2_details": {
      "unique_key": ["product_id"],
      "start_column": "valid_from",
      "end_column": "valid_to"
    }
  },
  
  "tests": {
    "columns": {
      "product_id": ["not_null", "unique"],
      "valid_from": ["not_null"]
    },
    "table": ["row_count_gt_0"]
  },
  
  "metadata": {
    "file_path": "/models/dim/products_scd2.sql",
    "owner": "data-engineering",
    "tags": ["products", "scd2", "dimension"]
  }
}
```

</details>
