# Documentation Overview

This directory contains comprehensive documentation for the `dogs-as-a-service-pipeline` project - a functional ETL pipeline that extracts dog breed data and loads it into Google Cloud Platform.

## Documentation Files

### 📋 [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md)
Complete project summary including current functionality, technology stack, and development roadmap.

### 🏗️ [ARCHITECTURE.md](./ARCHITECTURE.md) 
Detailed system architecture, data flow, component design, and technical implementation details.

### 📁 [FILE_STRUCTURE.md](./FILE_STRUCTURE.md)
Complete directory structure analysis, source code organization, and file descriptions.

### 🔌 [API_REFERENCE.md](./API_REFERENCE.md)
API documentation for Cloud Function endpoints, pipeline functions, data schemas, and error handling.

### 🚀 [DEPLOYMENT.md](./DEPLOYMENT.md)
Comprehensive deployment guide for Google Cloud Platform including setup, configuration, and monitoring.

## Purpose

These documents provide code agents with:
- **Context**: Understanding of project state and goals
- **Structure**: Clear view of current and recommended file organization  
- **Guidance**: Architecture patterns and technology decisions
- **Roadmap**: Next steps for development

## Project Status

**Current State**: Functional ETL pipeline with dual storage (BigQuery + Cloud Storage) and a modular Streamlit dashboard backed by dbt marts
**Last Updated**: August 26, 2025
**Target Audience**: AI code agents, developers, and DevOps engineers

## Recent Updates

- Added a modular Streamlit frontend (`streamlit_app.py`) with tabs:
  - `frontend/overview.py`: Overview charts
  - `frontend/finder.py`: Placeholder “Find your own dog” page
  - `frontend/filters.py`: Sidebar filters and SQL clause builder
- UI set to metric-only (kg/cm). Removed imperial units from the app.
- How to run the app:
  ```bash
  streamlit run /Users/hendrik/Documents/Repositories2/dogs-as-a-service-pipeline/streamlit_app.py
  ```

## Quick Navigation

- New to the project? Start with [PROJECT_OVERVIEW.md](./PROJECT_OVERVIEW.md)
- Planning implementation? Review [ARCHITECTURE.md](./ARCHITECTURE.md)
- Need file locations? Check [FILE_STRUCTURE.md](./FILE_STRUCTURE.md)

This documentation will be updated as the project evolves and implementation progresses.