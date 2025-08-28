# BigQuery Dataset Architecture

## Overview

This document outlines the BigQuery dataset structure for the Dog Breed Explorer project, following a modern data warehouse architecture with clear separation of concerns.

## Dataset Structure

### 1. Bronze Layer (Raw Data)
- **Dataset**: `bronze`
- **Purpose**: Raw, unprocessed data from external sources
- **Data Quality**: Minimal validation, preserves original format
- **Retention**: Long-term storage of raw data
- **Tables**:
  - `dog_breeds` - Raw data from TheDogAPI via dlt pipeline

### 2. Staging Layer (Cleaned Data)
- **Dataset**: `dog_explorer_staging` (dev: `dog_explorer_dev_staging`)
- **Purpose**: Cleaned, validated, and standardized data
- **Data Quality**: Type casting, null handling, data validation
- **Retention**: Medium-term storage
- **Tables**:
  - `stg_dog_breeds` - Cleaned and parsed dog breed data

### 3. Marts Layer (Business-Ready Data)
- **Dataset**: `dog_explorer_marts` (dev: `dog_explorer_dev_marts`)
- **Purpose**: Business-ready dimensional models for analytics
- **Data Quality**: Business logic applied, optimized for querying
- **Retention**: Long-term storage of business data
- **Sub-datasets**:
  - `dog_explorer_marts_core` (dev: `dog_explorer_dev_marts_core`)
    - `dim_breeds` - Dimension table for breed information
    - `dim_temperament` - Dimension table for temperament traits
    - `fct_breed_metrics` - Fact table for breed measurements and metrics

## Data Flow

```
External API (TheDogAPI)
        ↓
    dlt Pipeline
        ↓
    bronze.dog_breeds (Raw)
        ↓
    stg_dog_breeds (Cleaned)
        ↓
    dim_breeds + dim_temperament + fct_breed_metrics (Business-ready)
        ↓
    Streamlit Frontend (metric-only)
        ├── frontend/filters.py → sidebar + SQL clauses (metric-only)
        ├── frontend/overview.py → charts (lifespan, size distribution, top temperaments)
        └── frontend/finder.py → dataset-grounded assistant (streams via OpenAI; heuristic fallback)
```

## Environment Separation

### Development Environment
- **Base Dataset**: `dog_explorer_dev`
- **Staging**: `dog_explorer_dev_staging`
- **Marts**: `dog_explorer_dev_marts`
- **Core Marts**: `dog_explorer_dev_marts_core`

### Production Environment
- **Base Dataset**: `dog_explorer`
- **Staging**: `dog_explorer_staging`
- **Marts**: `dog_explorer_marts`
- **Core Marts**: `dog_explorer_marts_core`

## Naming Conventions

### Datasets
- `bronze` - Raw data layer
- `{project}_{env}_staging` - Staging layer
- `{project}_{env}_marts` - Business layer
- `{project}_{env}_marts_core` - Core business layer

### Tables
- `stg_*` - Staging tables (cleaned raw data)
- `dim_*` - Dimension tables (descriptive attributes)
- `fct_*` - Fact tables (measurable events/metrics)

## Benefits of This Structure

1. **Clear Separation**: Each layer has a distinct purpose
2. **Data Lineage**: Easy to trace data from source to consumption
3. **Environment Isolation**: Development and production are completely separate
4. **Scalability**: Easy to add new data sources and business domains
5. **Performance**: Optimized materializations for each use case
6. **Maintainability**: Clear ownership and responsibility for each layer
7. **Modular UI**: Frontend split into pages and shared filters for reuse

## Migration from Current State

The current `dbt_hsip_staging` dataset was created due to incorrect schema configuration. After applying the updated `dbt_project.yml`, the correct structure will be created. You may want to:

1. **Backup existing data** if needed
2. **Run dbt run** to create the new structure
3. **Verify data quality** in the new datasets
4. **Drop old datasets** once confirmed working

## Best Practices

1. **Never query bronze directly** for business logic
2. **Use staging for data quality checks** and validation
3. **Build business logic in marts** layer only
4. **Test each layer independently**
5. **Document schema changes** and data lineage
6. **Monitor data freshness** and quality metrics
7. **Prefer metric units** in UI for consistency (kg, cm)
8. **Secrets management**: Streamlit requires `st.secrets["gcp_service_account"]`; optional `OPENAI_API_KEY` for assistant
9. **Dataset scoping**: Set `PROJECT_DATASET` in `streamlit_app.py` to the correct `..._marts_core` dataset