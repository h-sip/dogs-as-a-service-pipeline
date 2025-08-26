# Architecture Documentation

## Current Architecture Implementation

**Status**: Functional ETL pipeline implemented using modern Python data engineering tools.

## System Architecture Overview

```
TheDogAPI (REST) → Data Pipeline (Cloud Function) → Dual Storage (GCS + BigQuery)
```

### High-Level Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────┐
│   TheDogAPI     │    │   Cloud Function │    │    Storage Layer    │
│   (External)    │───▶│   ETL Pipeline   │───▶│  ┌─────────────────┐│
│                 │    │                  │    │  │   BigQuery      ││
│ • REST API      │    │ • Data Extract   │    │  │   (bronze)      ││
│ • 172 breeds    │    │ • Transform      │    │  └─────────────────┘│
│ • JSON format   │    │ • Dual Load      │    │  ┌─────────────────┐│
└─────────────────┘    └──────────────────┘    │  │ Cloud Storage   ││
                                                │  │ (raw/partitioned│
                                                │  └─────────────────┘│
                                                └─────────────────────┘
```

## Core Components

### 1. Data Extraction Layer
**Location**: `src/dog_api_pipeline.py:fetch_dog_breeds()`

- **Source**: TheDogAPI (https://api.thedogapi.com/v1/breeds)
- **Method**: HTTP GET request using `requests` library
- **Format**: JSON array of dog breed objects
- **Volume**: 172 dog breed records per extraction
- **Error Handling**: Request exceptions with proper logging

### 2. Data Transformation Layer
**Location**: `src/dog_api_pipeline.py:fetch_dog_breeds()`

- **Metadata Enrichment**: Adds `extracted_at` timestamp
- **Date Partitioning**: Adds `extraction_date` for data partitioning
- **Schema Consistency**: Preserves original API structure
- **Data Types**: Maintains string/integer types from source

### 3. Storage Architecture

#### Raw Data Storage (Data Lake)
**Location**: `src/dog_api_pipeline.py:save_to_cloud_storage()`

- **Destination**: Google Cloud Storage
- **Format**: JSON files (raw API responses)
- **Partitioning**: Date-based (YYYY_MM_DD)
- **Purpose**: Historical data preservation, data lineage

#### Processed Data Storage (Data Warehouse)
**Location**: `src/dog_api_pipeline.py:load_to_bigquery()`

- **Destination**: Google BigQuery
- **Dataset**: `bronze` (raw/landing layer)
- **Table**: `dog_breeds` (auto-created by DLT)
- **Schema**: Dynamic schema inference by DLT
- **Write Mode**: Replace (full refresh per run)

### 4. Pipeline Orchestration
**Framework**: DLT (Data Load Tool)

#### DLT Configuration
- **Pipeline Name**: `dog_breeds_pipeline`
- **Resource Name**: `dog_breeds`
- **Write Disposition**: `replace` (full refresh)
- **Column Definitions**: Timestamp column explicitly typed

#### Dual Pipeline Pattern
1. **Filesystem Pipeline**: Raw data to Cloud Storage
2. **BigQuery Pipeline**: Processed data to warehouse

### 5. Cloud Function Integration
**Entry Points**: 
- `main.py:dog_pipeline_handler()` - HTTP handler
- `src/dog_api_pipeline.py:main()` - Direct execution

#### Function Characteristics
- **Runtime**: Python 3.11+
- **Trigger**: HTTP request (Cloud Scheduler compatible)
- **Response**: JSON status with load information
- **Dependencies**: Managed via `pyproject.toml`

## Data Flow Architecture

### 1. Extraction Phase
```python
# HTTP Request → JSON Response → Python Dict List
api_url = "https://api.thedogapi.com/v1/breeds"
response = requests.get(api_url)
breeds_data = response.json()  # List[Dict[str, Any]]
```

### 2. Transformation Phase  
```python
# Add Metadata → Enhanced Records
for breed in breeds_data:
    breed["extracted_at"] = datetime.utcnow().isoformat()
    breed["extraction_date"] = datetime.utcnow().date().isoformat()
```

### 3. Dual Load Phase
```python
# Parallel loading to two destinations
save_to_cloud_storage(data, date_partition)  # Raw data
pipeline.run(fetch_dog_breeds())             # Processed data
```

## Technology Stack Details

### Core Dependencies
```toml
[project.dependencies]
dlt[bigquery,gcs] = ">=1.15.0"     # Pipeline orchestration
functions-framework = ">=3.9.2"    # Cloud Function runtime
requests = "*"                      # HTTP client (implicit)
```

### DLT Architecture Benefits
- **Schema Evolution**: Automatic schema detection and evolution
- **Data Lineage**: Built-in metadata tracking
- **Error Handling**: Robust error handling and retries
- **Incremental Loading**: Support for incremental patterns (not used currently)
- **Multiple Destinations**: Native support for multiple storage targets

## Deployment Architecture

### Current Deployment Pattern
- **Local Development**: Direct Python execution via `python main.py`
- **Cloud Deployment**: Google Cloud Function (HTTP triggered)
- **Scheduling**: Cloud Scheduler → HTTP trigger → Cloud Function

### Infrastructure Requirements
- **Google Cloud Project** with enabled APIs:
  - Cloud Functions API
  - BigQuery API  
  - Cloud Storage API
- **Service Account** with appropriate permissions:
  - BigQuery Data Editor
  - Storage Admin
- **Cloud Storage Bucket** for raw data storage

## Error Handling & Monitoring

### Current Error Handling
- **API Failures**: Request exceptions caught and logged
- **Pipeline Failures**: DLT handles load errors with detailed logging
- **Function Failures**: JSON error responses returned to caller

### Logging Strategy
- **Info Level**: Successful operations, record counts
- **Error Level**: API failures, pipeline exceptions
- **Return Values**: Structured JSON responses with status

## Data Quality & Governance

### Current Implementation
- **Data Freshness**: Timestamps on every record
- **Data Lineage**: DLT metadata tracking
- **Schema Consistency**: DLT schema validation
- **Error Tracking**: Exception logging and status returns

### Missing Elements (Improvement Areas)
- **Data Validation Rules**: No explicit data quality checks
- **Deduplication Logic**: No handling of duplicate breeds
- **Monitoring/Alerting**: No automated failure notifications
- **Data Catalog**: No metadata catalog integration

## Performance Considerations

### Current Performance Characteristics
- **Extraction**: Single API call (fast)
- **Volume**: 172 records (~small dataset)
- **Network**: Minimal latency for API calls
- **Storage**: Dual writes (parallel operations)

### Scalability Considerations
- **API Limits**: TheDogAPI rate limiting (not currently handled)
- **BigQuery Limits**: Well within quotas for current volume
- **Cloud Function Limits**: 540 seconds timeout (adequate)
- **Memory Usage**: Minimal for current data volume