# 🐕 Dogs-as-a-Service Data Platform

## Overview

A comprehensive **end-to-end data engineering platform** that demonstrates modern data architecture patterns through dog breed analytics. This project combines a robust ETL pipeline with advanced analytics capabilities, showcasing both technical depth and practical business value.

### 🎯 Project Highlights

- **Complete Data Platform**: ETL pipeline + dbt analytics warehouse
- **Production-Ready**: 20+ automated tests, comprehensive documentation
- **Advanced Analytics**: Temperament scoring, family matching algorithms
- **Modern Stack**: DLT, dbt, BigQuery, Cloud Functions
- **Business Value**: Real-world insights for pet industry applications

## 🏗️ Architecture

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────────┐    ┌─────────────────────┐
│   TheDogAPI     │    │   Cloud Function │    │   Bronze Layer      │    │   dbt Analytics     │
│   (External)    │───▶│   ETL Pipeline   │───▶│                     │───▶│                     │
│                 │    │                  │    │ ┌─────────────────┐ │    │ ┌─────────────────┐ │
│ • REST API      │    │ • Data Extract   │    │ │   BigQuery      │ │    │ │   Staging       │ │
│ • 172 breeds    │    │ • Transform      │    │ │   bronze        │ │    │ │ stg_dog_breeds  │ │
│ • JSON format   │    │ • Dual Load      │    │ │   schema        │ │    │ └─────────────────┘ │
└─────────────────┘    └──────────────────┘    │ └─────────────────┘ │    │ ┌─────────────────┐ │
                                                │ ┌─────────────────┐ │    │ │      Marts      │ │
                                                │ │ Cloud Storage   │ │    │ │ • dim_breeds    │ │
                                                │ │(raw/partitioned)│ │    │ │ • fct_metrics   │ │
                                                │ └─────────────────┘ │    │ │ • dim_temper    │ │
                                                └─────────────────────┘    │ └─────────────────┘ │
                                                                           └─────────────────────┘
```

## 🚀 Quick Start

### Prerequisites
- Python 3.11+
- Google Cloud Platform account
- dbt-core with BigQuery adapter

### 1. ETL Pipeline Setup
```bash
# Clone and install
git clone <repository>
cd dogs-as-a-service-pipeline
uv sync  # or pip install -r requirements.txt

# Test locally
python main.py
```

### 2. dbt Analytics Setup
```bash
# Install dbt dependencies
dbt deps

# Configure BigQuery connection (see profiles.yml template)
cp profiles.yml ~/.dbt/profiles.yml
# Edit with your BigQuery project details

# Run analytics pipeline
dbt run
dbt test
```

### dbt Project Details

#### Data Models
- `stg_dog_breeds`: Cleaned and normalized raw data with parsed measurement ranges
- `dim_breeds`: Master dimension with breed characteristics and derived insights
- `fct_breed_metrics`: Quantitative metrics and physical measurements analysis
- `dim_temperament`: Behavioral analysis with temperament scoring and categorization

#### Testing Strategy
- Schema tests (uniqueness, referential integrity, ranges, accepted values)
- Custom tests: weight-height ratio validation, temperament score validation, cross-model consistency

#### Common dbt Commands
```bash
# Run everything
dbt run && dbt test

# Run a specific model
dbt run --select stg_dog_breeds

# Run a model and its dependencies
dbt run --select dim_breeds+

# Test a specific model
dbt test --select fct_breed_metrics

# Generate and serve docs
dbt docs generate && dbt docs serve
```

### 3. Explore the Data
```sql
-- Find family-friendly small breeds
SELECT 
    breed_name,
    size_category,
    avg_life_span_years,
    family_suitability
FROM dim_breeds d
JOIN dim_temperament t USING (breed_id)
WHERE size_category = 'Small'
  AND family_suitability = 'Excellent for Families'
ORDER BY avg_life_span_years DESC;
```

## 📊 Data Models & Analytics

### Staging Layer
- **`stg_dog_breeds`**: Cleaned data with intelligent range parsing

### Analytical Models
- **`dim_breeds`**: Master breed dimension with 15+ derived insights
- **`fct_breed_metrics`**: Physical measurements with calculated ratios
- **`dim_temperament`**: Advanced behavioral analysis with 0-1 scoring

### Key Analytics Features
- **Smart Range Parsing**: "22 - 25 pounds" → min/max columns
- **Temperament Scoring**: Normalized behavioral trait analysis
- **Family Matching**: AI-driven suitability predictions
- **Data Quality**: Completeness tracking and validation

## 🧪 Quality & Testing

### Comprehensive Test Suite
- **20+ Schema Tests**: Uniqueness, referential integrity, ranges
- **3 Custom Tests**: Business logic validation
- **Data Quality Monitoring**: Completeness scores, validation flags
- **Cross-Model Consistency**: Referential integrity enforcement

### Testing Commands
```bash
# ETL Pipeline
python main.py

# dbt Analytics
dbt test
dbt test --select stg_dog_breeds
dbt test --store-failures  # Store test failures for analysis
```

## 📈 Business Impact

### Findings & Business Impact Narrative
The Overview dashboard highlights that longevity skews toward small and toy companion breeds: the top cohort includes Toy Poodle, Maltese, Pekingese, and several terriers with expected lifespans around 14–17 years. Breed counts cluster in the medium and small size categories, with extra‑large breeds comparatively rare—signaling narrower supply and higher ownership costs. Among breeds rated highly for families, dominant temperament traits are intelligent, affectionate, alert, loyal and friendly, indicating lower training friction and reduced return‑to‑shelter risk.

Business impact: adoption teams can prioritize small/medium, high‑affection/intelligence breeds for family placements to raise success rates; retailers and insurers can align assortments and pricing with the prevalent size mix and lifespan bands; training providers can productize curricula around the most common family‑friendly traits (playfulness, energy, protectiveness). The Find Your Own Dog assistant further shortens time‑to‑match by converting lifestyle descriptions into data‑grounded recommendations, consistently constrained by the active dataset filters.

### Analytical Insights Delivered
- **Family Matching**: Optimal breed recommendations for households
- **Longevity Analysis**: Breed lifespan patterns and predictions
- **Training Programs**: Difficulty assessment based on temperament
- **Veterinary Insights**: Breed-specific health correlations

### Sample Business Questions Answered
1. Which breeds live longest among family-friendly dogs?
2. How does temperament complexity vary by breed group?
3. What physical characteristics correlate with training difficulty?
4. Which breeds are best for first-time dog owners?

### Key Performance Indicators
- **Data Coverage**: 172+ breeds with 90%+ completeness
- **Analytics Depth**: 25+ calculated metrics per breed
- **Quality Score**: 20+ automated validation rules
- **Processing Time**: <60 seconds for complete refresh

## 🛠️ Technology Stack

### Data Pipeline
- **DLT (Data Load Tool)**: Modern Python ETL framework
- **Google Cloud Functions**: Serverless execution
- **BigQuery**: Cloud data warehouse
- **Cloud Storage**: Raw data lake

### Analytics Layer
- **dbt**: Data transformation and modeling
- **BigQuery**: Analytical data warehouse
- **dbt_utils**: Testing and utility macros
- **Git**: Version-controlled transformations

### Development Tools
- **UV**: Fast Python package management
- **Jupyter**: Interactive data exploration
- **Google Cloud SDK**: Cloud deployment

## 📚 Documentation

### Complete Documentation Suite
- **[Project Overview](docs/PROJECT_OVERVIEW.md)**: Business context and roadmap
- **[Architecture](docs/ARCHITECTURE.md)**: Technical architecture and design
- **[API Reference](docs/API_REFERENCE.md)**: Complete data model schemas
- **[Deployment Guide](docs/DEPLOYMENT.md)**: ETL + dbt deployment instructions
- **[File Structure](docs/FILE_STRUCTURE.md)**: Project organization guide

### Additional Resources
- **[Analysis Examples](analyses/breed_insights.sql)**: Sample analytical queries
- **Auto-generated dbt docs**: `dbt docs generate && dbt docs serve`

## 🔧 Development

### Local Development Workflow
```bash
# ETL Development
python -c "from src.dog_api_pipeline import fetch_dog_breeds; print(len(list(fetch_dog_breeds())))"

# dbt Development  
dbt run --select staging
dbt run --select dim_breeds+
dbt docs serve --port 8080
```

### Production Deployment
```bash
# Deploy ETL Pipeline
gcloud functions deploy dog-pipeline-handler \
    --runtime python311 \
    --source . \
    --entry-point dog_pipeline_handler \
    --trigger-http

# Deploy dbt Models
dbt run --target prod
dbt test --target prod
```

### Performance Considerations (dbt)
- Materialization strategy: staging as views for fast iteration; marts as tables for analytics
- Consider clustering on `breed_id` for large datasets
- Partition historical datasets when applicable

## 🎯 Use Cases

### Pet Industry Applications
- **Adoption Centers**: Match breeds to family requirements
- **Training Services**: Tailor approaches by temperament
- **Veterinary Practices**: Breed-specific health insights
- **Insurance**: Risk assessment and pricing models

### Technical Demonstrations
- **Data Engineering Interviews**: Complete modern data platform
- **Architecture Reviews**: Production-ready design patterns
- **Analytics Showcases**: Advanced SQL and business logic
- **ETL Best Practices**: Error handling, testing, monitoring

## 🏆 Project Strengths

### Technical Excellence
✅ **Modern Architecture**: Bronze/Silver/Gold medallion pattern  
✅ **Advanced Analytics**: Complex SQL with array operations, scoring algorithms  
✅ **Comprehensive Testing**: Multi-layer validation and quality assurance  
✅ **Production-Ready**: Error handling, monitoring, documentation  

### Business Value
✅ **Clear ROI**: Immediate analytical value for pet industry  
✅ **Scalable Design**: Architecture supports growth and enhancement  
✅ **Real-World Application**: Genuine business problem solving  
✅ **Educational Value**: Demonstrates best practices and patterns  

## 📊 Data Sample

The pipeline processes 172 dog breeds with rich metadata:

```json
{
  "breed_name": "Golden Retriever",
  "size_category": "Large", 
  "avg_weight_lbs": 65,
  "family_friendliness_score": 0.83,
  "training_difficulty": "Easy to Train",
  "primary_temperament_category": "Social/Friendly"
}
```

## 🤝 Contributing

This project demonstrates production-ready data engineering practices. Areas for expansion:

- **Additional Data Sources**: Cat breeds, health data, training records
- **ML Integration**: Breed recommendation engine
- **Real-time Pipeline**: Streaming data processing
- **Advanced Analytics**: Predictive modeling, anomaly detection

## 📄 License

This project is designed for educational and interview purposes, demonstrating modern data engineering capabilities.

---

**Built with ❤️ showcasing modern data engineering practices**  
*Complete ETL + Analytics platform with real business value*