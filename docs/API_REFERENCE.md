# API Reference

## Overview

This document describes the API interface for the dogs-as-a-service-pipeline, including the Cloud Function endpoints and the underlying data pipeline functions.

## Cloud Function API

### Endpoint
```
POST/GET https://{region}-{project-id}.cloudfunctions.net/dog-pipeline-handler
```

### HTTP Handler Function

#### `dog_pipeline_handler(request)`
**Location**: `main.py:3`

Entry point for Google Cloud Function HTTP requests.

**Parameters:**
- `request`: Flask Request object (can be None for local execution)

**Returns:**
```json
{
  "status": "success|error",
  "message": "Description of result",
  "load_info": "DLT load information (string representation)"
}
```

**Success Response Example:**
```json
{
  "status": "success",
  "message": "Dog breeds data loaded successfully",
  "load_info": "LoadInfo(pipeline_name='dog_breeds_pipeline', destination_name='bigquery', ..."
}
```

**Error Response Example:**
```json
{
  "status": "error", 
  "message": "Pipeline failed: API request timeout"
}
```

## Core Pipeline API

### Main Pipeline Function

#### `main(request=None)`
**Location**: `src/dog_api_pipeline.py:90`

Main entry point that orchestrates the complete ETL pipeline.

**Parameters:**
- `request` (optional): HTTP request object (ignored in current implementation)

**Workflow:**
1. Calls `load_to_bigquery()` to execute pipeline
2. Handles exceptions and returns status
3. Logs all operations

**Returns:** Same JSON structure as HTTP handler

### Data Processing Functions

#### `fetch_dog_breeds()`
**Location**: `src/dog_api_pipeline.py:13`

DLT resource function that extracts dog breed data from TheDogAPI.

**Decorator:** `@dlt.resource`
**Configuration:**
- `name`: "dog_breeds"
- `write_disposition`: "replace" 
- `columns`: {"extracted_at": {"data_type": "timestamp"}}

**External API:** `https://api.thedogapi.com/v1/breeds`

**Returns:** `List[Dict[str, Any]]` - List of enhanced dog breed records

**Data Enhancement:**
- Adds `extracted_at`: ISO format timestamp
- Adds `extraction_date`: Date string (YYYY-MM-DD)

**Error Handling:**
- Catches `requests.exceptions.RequestException`
- Logs errors and re-raises exceptions

#### `save_to_cloud_storage(data, date_partition)`
**Location**: `src/dog_api_pipeline.py:40`

Saves raw JSON data to Google Cloud Storage with date partitioning.

**Parameters:**
- `data`: `List[Dict[str, Any]]` - Raw dog breed data
- `date_partition`: `str` - Date string for partitioning (YYYY-MM-DD)

**Pipeline Configuration:**
- `pipeline_name`: "dog_breeds_raw_storage"
- `destination`: "filesystem" (GCS)
- `dataset_name`: "raw_data_{YYYY}_{MM}_{DD}"

**Storage Pattern:**
```
gs://{bucket}/raw_data_{YYYY}_{MM}_{DD}/raw_dog_api_data/
```

**Returns:** None (logs completion status)

#### `load_to_bigquery()`
**Location**: `src/dog_api_pipeline.py:61`

Main pipeline orchestrator that coordinates extraction and loading.

**Pipeline Configuration:**
- `pipeline_name`: "dog_breeds_pipeline"  
- `destination`: "bigquery"
- `dataset_name`: "bronze"

**Execution Flow:**
1. Fetches data using `fetch_dog_breeds()`
2. Saves raw data to GCS via `save_to_cloud_storage()`  
3. Loads processed data to BigQuery via DLT pipeline
4. Returns DLT LoadInfo object

**Returns:** `LoadInfo` - DLT pipeline execution results

## Data Schema

### API Source Schema (TheDogAPI)

The external API returns dog breed objects with the following structure:

```json
{
  "id": 1,
  "name": "Affenpinscher", 
  "bred_for": "Small rodent hunting, lapdog",
  "breed_group": "Toy",
  "life_span": "10 - 12 years",
  "temperament": "Stubborn, Curious, Playful, Adventurous, Active, Fun-loving",
  "origin": "Germany, France",
  "reference_image_id": "BJa4kxc4X",
  "weight": {
    "imperial": "6 - 13",
    "metric": "3 - 6"
  },
  "height": {
    "imperial": "9 - 11.5", 
    "metric": "23 - 29"
  }
}
```

### Enhanced Schema (Pipeline Output)

The pipeline adds metadata fields to each record:

```json
{
  "id": 1,
  "name": "Affenpinscher",
  "bred_for": "Small rodent hunting, lapdog",
  "breed_group": "Toy", 
  "life_span": "10 - 12 years",
  "temperament": "Stubborn, Curious, Playful, Adventurous, Active, Fun-loving",
  "origin": "Germany, France",
  "reference_image_id": "BJa4kxc4X",
  "weight": {"imperial": "6 - 13", "metric": "3 - 6"},
  "height": {"imperial": "9 - 11.5", "metric": "23 - 29"},
  "extracted_at": "2025-08-26T14:17:18.109571",
  "extraction_date": "2025-08-26"
}
```

### BigQuery Schema

DLT automatically infers and creates the BigQuery schema. Key characteristics:

**Table:** `{project}.bronze.dog_breeds`

**Column Types:**
- `id`: INTEGER
- `name`: STRING
- `breed_group`: STRING (nullable)
- `life_span`: STRING
- `temperament`: STRING
- `origin`: STRING (nullable)
- `reference_image_id`: STRING
- `weight`: RECORD (nested: imperial STRING, metric STRING)
- `height`: RECORD (nested: imperial STRING, metric STRING) 
- `extracted_at`: TIMESTAMP
- `extraction_date`: DATE

**Write Mode:** REPLACE (full table refresh each run)

## Error Handling

### HTTP Function Errors

The Cloud Function returns appropriate HTTP status codes:

- **200 OK**: Successful pipeline execution
- **500 Internal Server Error**: Pipeline failure

### Pipeline Errors

Common error scenarios and handling:

#### API Request Failures
```python
except requests.exceptions.RequestException as e:
    print(f"Error fetching data from Dog API: {e}")
    raise
```

**Causes:**
- Network connectivity issues
- TheDogAPI service downtime
- Request timeout
- Invalid API endpoint

#### DLT Pipeline Failures
```python
except Exception as e:
    print(f"Pipeline failed: {str(e)}")
    return {"status": "error", "message": f"Pipeline failed: {str(e)}"}
```

**Causes:**
- BigQuery authentication issues
- BigQuery quota exceeded
- Schema validation failures
- Network issues during data loading

### Monitoring and Observability

#### Logging Strategy

**Info Level Logs:**
- Successful data extraction: `"Successfully fetched {count} dog breeds"`
- Raw data storage: `"Raw data saved to Cloud Storage for date: {date}"`
- Pipeline completion: `"Pipeline completed successfully!"`

**Error Level Logs:**
- API failures: `"Error fetching data from Dog API: {error}"`
- Pipeline failures: `"Pipeline failed: {error}"`

#### Health Check

To verify pipeline functionality:

```bash
# Local execution test
python main.py

# Cloud Function test
curl -X POST https://{region}-{project}.cloudfunctions.net/dog-pipeline-handler
```

Expected successful response includes:
- `status: "success"`
- `message`: Contains "loaded successfully"
- `load_info`: Contains DLT execution details

## Usage Examples

### Local Development

```python
from dog_api_pipeline import main

# Execute pipeline locally
result = main()
print(result)
```

### Cloud Function Deployment

```yaml
# cloudfunctions/function.yaml
name: dog-pipeline-handler
runtime: python311
entry_point: dog_pipeline_handler
source: .
environment_variables:
  GOOGLE_CLOUD_PROJECT: your-project-id
```

### Programmatic Usage

```python
from dog_api_pipeline import fetch_dog_breeds, load_to_bigquery

# Extract data only
breeds = list(fetch_dog_breeds())
print(f"Extracted {len(breeds)} breeds")

# Full pipeline execution  
load_info = load_to_bigquery()
print(f"Pipeline result: {load_info}")
```

## Rate Limits and Quotas

### TheDogAPI Limits
- **Requests**: Not explicitly documented (appears unlimited for public API)
- **Data Volume**: 172 breeds per request (small dataset)
- **Recommendation**: Implement exponential backoff for production use

### Google Cloud Limits
- **Cloud Functions**: 540 second timeout (adequate for current volume)
- **BigQuery**: Well within quotas for 172 records
- **Cloud Storage**: No practical limits for current volume

### Recommended Production Enhancements
- Implement request retry logic with exponential backoff
- Add rate limiting to respect API quotas
- Monitor BigQuery slot usage for larger datasets
- Implement circuit breaker pattern for API failures