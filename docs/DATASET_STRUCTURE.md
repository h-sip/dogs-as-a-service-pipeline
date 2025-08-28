# BigQuery Dataset Structure Reference

## Quick Reference

| Layer | Dataset | Purpose | Materialization | Tables |
|-------|---------|---------|----------------|---------|
| **Bronze** | `bronze` | Raw data | Table | `dog_breeds` |
| **Staging** | `dog_explorer_dev_staging` | Cleaned data | View | `stg_dog_breeds` |
| **Marts** | `dog_explorer_dev_marts_core` | Business data | Table | `dim_breeds`, `dim_temperament`, `fct_breed_metrics` |

## Environment Mapping

### Development (`--target dev`)
- Base: `dog_explorer_dev`
- Staging: `dog_explorer_dev_staging`
- Marts: `dog_explorer_dev_marts_core`

### Production (`--target prod`)
- Base: `dog_explorer`
- Staging: `dog_explorer_staging`
- Marts: `dog_explorer_marts_core`

## Data Flow Summary

```
bronze.dog_breeds (Raw API data)
    ↓
stg_dog_breeds (Cleaned & parsed)
    ↓
dim_breeds + dim_temperament + fct_breed_metrics (Business-ready)
```

## Key Changes Made

1. **Fixed Schema Naming**: Updated `dbt_project.yml` to use `{{ target.dataset }}_staging` instead of just `staging`
2. **Proper Environment Separation**: Each environment gets its own datasets
3. **Clear Layer Separation**: Bronze → Staging → Marts with distinct purposes

## Next Steps

1. Run `dbt run --target dev` to create the new structure
2. Verify data quality with `dbt test`
3. Generate documentation with `dbt docs generate`
4. Drop old `dbt_hsip_staging` dataset once confirmed working
