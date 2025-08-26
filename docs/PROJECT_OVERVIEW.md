# Project Overview

## dogs-as-a-service-pipeline

### Current State
This is a **functional data pipeline project** that fetches dog breed information from TheDogAPI and loads it into Google BigQuery via Google Cloud Storage. The project is built using modern Python data engineering tools and is designed to run as a Google Cloud Function.

### Repository Information
- **Owner**: hendrik-spl
- **Repository**: https://github.com/hendrik-spl/dogs-as-a-service-pipeline.git
- **Primary Language**: Python
- **Purpose**: ETL pipeline for dog breed data
- **Deployment**: Google Cloud Functions
- **Storage**: Google BigQuery + Google Cloud Storage

### Key Features
- **Data Extraction**: Fetches dog breed data from TheDogAPI REST API
- **Data Processing**: Adds extraction timestamps and metadata
- **Dual Storage**: Saves raw data to GCS and processed data to BigQuery
- **Cloud Native**: Designed for Google Cloud Platform deployment
- **Serverless**: Runs as a Cloud Function triggered by Cloud Scheduler
- **Modern Tooling**: Uses DLT (data load tool) for pipeline orchestration

### Technology Stack
- **Pipeline Framework**: DLT (Data Load Tool)
- **HTTP Client**: requests
- **Cloud Platform**: Google Cloud Platform
- **Data Warehouse**: BigQuery
- **Data Lake**: Google Cloud Storage
- **Runtime**: Google Cloud Functions
- **Package Management**: UV (ultraviolet)
- **Development**: Jupyter notebooks for EDA

### Current Implementation Status
- ✅ Core pipeline functionality implemented
- ✅ Data extraction from TheDogAPI
- ✅ Data transformation and enrichment
- ✅ BigQuery integration (bronze layer)
- ✅ Cloud Storage integration (raw data)
- ✅ Cloud Function entry point
- ✅ Error handling and logging
- ✅ Package dependencies defined
- ❌ Testing framework not implemented
- ❌ CI/CD pipeline not configured
- ❌ Deployment scripts not included

### Data Pipeline Architecture
1. **Extraction**: Fetch dog breeds from TheDogAPI (172 breeds)
2. **Transformation**: Add extraction timestamps and metadata
3. **Load (Raw)**: Save original JSON to Cloud Storage (partitioned by date)
4. **Load (Processed)**: Load structured data to BigQuery bronze table
5. **Orchestration**: Triggered by Cloud Scheduler or HTTP request

### Data Schema
Each dog breed record includes:
- Basic info: `id`, `name`, `breed_group`, `origin`
- Physical traits: `weight`, `height`, `life_span`
- Characteristics: `temperament`, `bred_for`
- Metadata: `extracted_at`, `extraction_date`, `reference_image_id`

### Development Roadmap
1. **Testing Infrastructure**
   - Add pytest framework
   - Unit tests for data extraction
   - Integration tests for BigQuery/GCS
   - Mock API responses for testing

2. **Deployment Automation**
   - Cloud Function deployment scripts
   - Terraform/Cloud Deployment Manager configs
   - Environment variable management

3. **Data Quality & Monitoring**
   - Data validation rules
   - Pipeline monitoring and alerting
   - Error handling improvements

4. **Pipeline Enhancements**
   - Silver/Gold layer transformations
   - Data deduplication logic
   - Incremental loading strategies

5. **CI/CD Pipeline**
   - GitHub Actions workflow
   - Automated testing
   - Deployment automation