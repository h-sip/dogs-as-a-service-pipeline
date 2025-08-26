# Deployment Guide

## Overview

This guide covers deploying the dogs-as-a-service-pipeline to Google Cloud Platform as a Cloud Function with scheduled execution.

## Prerequisites

### Required Tools
- **Google Cloud SDK** (`gcloud` CLI)
- **Python 3.11+** (specified in pyproject.toml)
- **UV package manager** (recommended) or pip

### Google Cloud Setup
- Google Cloud Project with billing enabled
- Required APIs enabled:
  - Cloud Functions API
  - BigQuery API
  - Cloud Storage API
  - Cloud Scheduler API (for automation)

### Service Account Permissions
Create a service account with these IAM roles:
- **BigQuery Data Editor** - For writing to BigQuery tables
- **Storage Admin** - For Cloud Storage operations
- **Cloud Functions Invoker** - For function execution

## Local Development Setup

### 1. Environment Preparation

```bash
# Clone repository
git clone https://github.com/hendrik-spl/dogs-as-a-service-pipeline.git
cd dogs-as-a-service-pipeline

# Install UV (if not already installed)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Install dependencies
uv sync

# Set up Python version
uv python install 3.11
```

### 2. Local Testing

```bash
# Test pipeline locally
python main.py

# Or test specific components
python -c "from src.dog_api_pipeline import fetch_dog_breeds; print(len(list(fetch_dog_breeds())))"
```

### 3. Configure Google Cloud Credentials

```bash
# Authenticate with Google Cloud
gcloud auth login
gcloud config set project YOUR_PROJECT_ID

# Create and download service account key (for local dev)
gcloud iam service-accounts create dogs-pipeline-service
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/bigquery.dataEditor"
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/storage.admin"

# Download key for local development
gcloud iam service-accounts keys create credentials.json \
    --iam-account=dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com

export GOOGLE_APPLICATION_CREDENTIALS="$(pwd)/credentials.json"
```

## Cloud Function Deployment

### 1. Basic Deployment

```bash
# Deploy Cloud Function
gcloud functions deploy dog-pipeline-handler \
    --runtime python311 \
    --source . \
    --entry-point dog_pipeline_handler \
    --trigger-http \
    --allow-unauthenticated \
    --timeout 540s \
    --memory 512MB \
    --service-account dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com
```

### 2. Advanced Deployment Configuration

Create `deployment/cloud-function.yaml`:

```yaml
name: dog-pipeline-handler
runtime: python311
entryPoint: dog_pipeline_handler
httpsTrigger: {}
timeout: 540s
availableMemoryMb: 512
serviceAccountEmail: dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com
environmentVariables:
  GOOGLE_CLOUD_PROJECT: YOUR_PROJECT_ID
  DLT_DESTINATION__BIGQUERY__LOCATION: US
labels:
  service: dogs-pipeline
  environment: production
```

Deploy using configuration:
```bash
gcloud functions deploy dog-pipeline-handler --source . --env-vars-file deployment/cloud-function.yaml
```

### 3. Verify Deployment

```bash
# Test the deployed function
FUNCTION_URL=$(gcloud functions describe dog-pipeline-handler --format="value(httpsTrigger.url)")
curl -X POST $FUNCTION_URL

# Check function logs
gcloud functions logs read dog-pipeline-handler --limit 50
```

## Cloud Storage Setup

### 1. Create Storage Bucket

```bash
# Create bucket for raw data storage
gsutil mb -p YOUR_PROJECT_ID gs://YOUR_PROJECT_ID-dogs-pipeline-raw-data

# Set lifecycle policy (optional - for data retention management)
cat > lifecycle.json << EOF
{
  "rule": [
    {
      "action": {"type": "Delete"},
      "condition": {"age": 365}
    }
  ]
}
EOF

gsutil lifecycle set lifecycle.json gs://YOUR_PROJECT_ID-dogs-pipeline-raw-data
```

### 2. Configure DLT for Storage

Create `.dlt/secrets.toml` (local development):
```toml
[destination.filesystem]
bucket_url = "gs://YOUR_PROJECT_ID-dogs-pipeline-raw-data"

[destination.bigquery] 
project_id = "YOUR_PROJECT_ID"
location = "US"
```

## BigQuery Setup

### 1. Create Dataset

```bash
# Create bronze dataset for raw data
bq mk --dataset \
    --description "Bronze layer - raw dog breed data" \
    YOUR_PROJECT_ID:bronze
```

### 2. Set Access Controls

```bash
# Grant service account access to dataset
bq add-iam-policy-binding \
    --member=serviceAccount:dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com \
    --role=roles/bigquery.dataEditor \
    YOUR_PROJECT_ID:bronze
```

## Automated Scheduling

### 1. Cloud Scheduler Setup

```bash
# Create daily schedule for pipeline execution
gcloud scheduler jobs create http dogs-pipeline-daily \
    --schedule="0 6 * * *" \
    --uri=$FUNCTION_URL \
    --http-method=POST \
    --time-zone="UTC" \
    --description="Daily dog breeds data pipeline execution"
```

### 2. Schedule Options

**Daily at 6 AM UTC:**
```bash
--schedule="0 6 * * *"
```

**Every 12 hours:**
```bash
--schedule="0 */12 * * *"
```

**Weekly on Sundays:**
```bash
--schedule="0 6 * * 0"
```

### 3. Manual Trigger

```bash
# Trigger scheduled job manually
gcloud scheduler jobs run dogs-pipeline-daily
```

## Monitoring and Alerting

### 1. Cloud Function Monitoring

```bash
# View function metrics
gcloud functions describe dog-pipeline-handler --format="table(
    name,
    status,
    updateTime,
    httpsTrigger.url
)"

# Monitor executions
gcloud functions logs read dog-pipeline-handler --limit 100
```

### 2. BigQuery Monitoring

```sql
-- Check latest data loads
SELECT 
  extraction_date,
  COUNT(*) as record_count,
  MAX(extracted_at) as latest_extraction
FROM `YOUR_PROJECT_ID.bronze.dog_breeds` 
GROUP BY extraction_date
ORDER BY extraction_date DESC;
```

### 3. Error Alerting

Create alerting policy for function failures:

```bash
# Create notification channel (email)
gcloud alpha monitoring channels create \
    --display-name="Pipeline Alerts" \
    --type=email \
    --channel-labels=email_address=YOUR_EMAIL@example.com

# Create alerting policy for function errors
gcloud alpha monitoring policies create \
    --policy-from-file=deployment/alerting-policy.yaml
```

## Environment Management

### 1. Development Environment

```yaml
# deployment/environments/dev.yaml
environment: development
function_name: dog-pipeline-handler-dev
memory: 256MB
timeout: 300s
schedule: "0 */6 * * *"  # Every 6 hours for testing
```

### 2. Production Environment

```yaml
# deployment/environments/prod.yaml
environment: production
function_name: dog-pipeline-handler
memory: 512MB
timeout: 540s
schedule: "0 6 * * *"    # Daily at 6 AM
```

### 3. Deployment Script

Create `deploy.sh`:

```bash
#!/bin/bash
ENVIRONMENT=${1:-dev}
CONFIG_FILE="deployment/environments/${ENVIRONMENT}.yaml"

if [ ! -f "$CONFIG_FILE" ]; then
    echo "Environment configuration not found: $CONFIG_FILE"
    exit 1
fi

# Load configuration
FUNCTION_NAME=$(yq eval '.function_name' $CONFIG_FILE)
MEMORY=$(yq eval '.memory' $CONFIG_FILE)
TIMEOUT=$(yq eval '.timeout' $CONFIG_FILE)

# Deploy function
gcloud functions deploy $FUNCTION_NAME \
    --runtime python311 \
    --source . \
    --entry-point dog_pipeline_handler \
    --trigger-http \
    --allow-unauthenticated \
    --timeout $TIMEOUT \
    --memory $MEMORY \
    --service-account dogs-pipeline-service@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com

echo "Deployed $FUNCTION_NAME to $ENVIRONMENT environment"
```

Usage:
```bash
chmod +x deploy.sh
./deploy.sh dev   # Deploy to development
./deploy.sh prod  # Deploy to production
```

## Security Best Practices

### 1. Service Account Security

```bash
# Use principle of least privilege
gcloud projects add-iam-policy-binding YOUR_PROJECT_ID \
    --member="serviceAccount:dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com" \
    --role="roles/bigquery.dataEditor"

# Avoid using default service accounts
# Never commit service account keys to version control
```

### 2. Network Security

```bash
# Deploy function with VPC connector (optional)
gcloud functions deploy dog-pipeline-handler \
    --vpc-connector YOUR_VPC_CONNECTOR \
    --egress-settings vpc-connector

# Use private Google access for enhanced security
```

### 3. Secrets Management

```bash
# Store sensitive configuration in Secret Manager
gcloud secrets create api-key --data-file=api-key.txt

# Access secrets in function
gcloud functions deploy dog-pipeline-handler \
    --set-secrets API_KEY=api-key:latest
```

## Troubleshooting

### Common Issues

#### 1. Authentication Errors
```bash
# Check service account permissions
gcloud projects get-iam-policy YOUR_PROJECT_ID \
    --flatten="bindings[].members" \
    --format="table(bindings.role)" \
    --filter="bindings.members:dogs-pipeline-service@YOUR_PROJECT_ID.iam.gserviceaccount.com"
```

#### 2. Function Timeout
```bash
# Increase timeout (max 540s for HTTP functions)
gcloud functions deploy dog-pipeline-handler \
    --timeout 540s
```

#### 3. Memory Issues
```bash
# Increase memory allocation
gcloud functions deploy dog-pipeline-handler \
    --memory 1024MB
```

#### 4. BigQuery Quota Exceeded
```bash
# Check BigQuery quotas
bq ls --project_id=YOUR_PROJECT_ID --max_results=1000
```

### Log Analysis

```bash
# Function execution logs
gcloud functions logs read dog-pipeline-handler \
    --format="table(timestamp,severity,textPayload)"

# Filter error logs only
gcloud functions logs read dog-pipeline-handler \
    --severity=ERROR \
    --limit=50
```

### Performance Optimization

#### 1. Cold Start Reduction
```bash
# Set minimum instances to reduce cold starts
gcloud functions deploy dog-pipeline-handler \
    --min-instances 1
```

#### 2. Memory Optimization
- Start with 256MB and increase if needed
- Monitor memory usage in Cloud Console
- Consider function splitting for large operations

#### 3. Network Optimization
- Use VPC connector for consistent networking
- Implement connection pooling for database connections
- Cache frequently accessed data when appropriate

## Rollback Procedures

### 1. Function Rollback
```bash
# List function versions
gcloud functions versions list --function dog-pipeline-handler

# Rollback to previous version
gcloud functions deploy dog-pipeline-handler \
    --source gs://YOUR_BUCKET/previous-version.zip
```

### 2. Data Rollback
```bash
# Restore BigQuery table from backup
bq cp YOUR_PROJECT_ID:bronze.dog_breeds@TIMESTAMP \
    YOUR_PROJECT_ID:bronze.dog_breeds
```

This deployment guide provides a comprehensive approach to deploying and managing the dogs-as-a-service-pipeline in production environments.