# Netflix Data Pipeline - Medallion Architecture

## 📚 Project Overview

This project demonstrates a **production-ready data pipeline** using Databricks, implementing the **Medallion Architecture** (Bronze-Silver-Gold) for Netflix titles data.

### Architecture Layers:

```
┌─────────────────┐
│  Raw CSV Data   │
└────────┬────────┘
         │
         ▼
┌─────────────────┐
│  🥉 BRONZE      │  Raw data ingestion
│  raw_titles     │  • Immutable source
└────────┬────────┘  • Audit trail
         │
         ▼
┌─────────────────┐
│  🥈 SILVER      │  Clean & standardized
│  clean_titles   │  • Null handling
└────────┬────────┘  • Type conversion
         │           • Deduplication
         ▼           • Derived columns
┌─────────────────┐
│  🥇 GOLD        │  Business analytics
│  6 Analytics    │  • Pre-aggregated
│  Tables         │  • Dashboard-ready
└─────────────────┘  • Optimized
```

---

## 📁 Project Structure

```
databricks_netflix/
├── 01_Bronze_Netflix_Ingestion.ipynb      # Bronze layer - Raw data ingestion
├── 02_Silver_Netflix_Transformation.ipynb # Silver layer - Data cleaning
├── 03_Gold_Netflix_Analytics.ipynb        # Gold layer - Analytics tables
├── 04_SQL_Dashboard_Queries.ipynb         # Dashboard SQL queries
├── Instructions.md                        # Workflow setup guide
└── README.md                              # This file
```

---

## 🗄️ Unity Catalog Tables

### Bronze Layer
* `main.netflix_bronze.raw_titles` - Raw CSV data with audit columns

### Silver Layer
* `main.netflix_silver.clean_titles` - Cleaned, typed, enriched data

### Gold Layer
* `main.netflix_gold.top_genres` - Genre popularity rankings
* `main.netflix_gold.content_trend_by_year` - Yearly growth trends
* `main.netflix_gold.duration_by_genre` - Duration analysis by genre/type
* `main.netflix_gold.content_by_country` - Geographic content distribution
* `main.netflix_gold.content_by_rating` - Rating category analysis
* `main.netflix_gold.content_by_decade` - Historical content trends

---

## 🚀 How to Run

### Option 1: Manual Execution (Learning)

Run notebooks **in sequence**:

1. **Bronze Layer** (`01_Bronze_Netflix_Ingestion`)
   * Downloads Netflix CSV dataset
   * Creates Unity Catalog schemas
   * Loads raw data to Delta table

2. **Silver Layer** (`02_Silver_Netflix_Transformation`)
   * Cleans and standardizes data
   * Handles nulls and duplicates
   * Derives analytical columns

3. **Gold Layer** (`03_Gold_Netflix_Analytics`)
   * Creates 6 business analytics tables
   * Optimizes with OPTIMIZE command

4. **Dashboard Queries** (`04_SQL_Dashboard_Queries`)
   * Run SQL queries for visualizations
   * Create charts and dashboards

### Option 2: Automated Workflow (Production)

Create a Databricks Job with 3 sequential tasks. See `Instructions.md` for detailed steps.

---

## 📊 Key Features

### Data Engineering Best Practices

✅ **Medallion Architecture** - Clear separation of raw, clean, and analytics layers  
✅ **Delta Lake** - ACID transactions, time travel, versioning  
✅ **Unity Catalog** - Centralized governance and metadata  
✅ **Data Quality** - Validation checks at each layer  
✅ **Performance Optimization** - OPTIMIZE commands applied  
✅ **Documentation** - Comprehensive comments for beginners  

### Transformations Applied

**Bronze → Silver:**
* Standardized column names (snake_case)
* Null value handling
* Date type conversions
* Duplicate removal
* Derived columns: `content_age`, `decade`, `duration_numeric`

**Silver → Gold:**
* Genre analysis with exploded categories
* Time-series aggregations
* Geographic distribution
* Rating and duration analytics

---

## 💡 Learning Objectives

1. **Medallion Architecture** - Industry-standard data organization
2. **PySpark Fundamentals** - DataFrame API, transformations
3. **Delta Lake Operations** - Create, optimize, time travel
4. **Unity Catalog** - Schema management, table governance
5. **Data Quality** - Validation, cleansing, standardization
6. **SQL Analytics** - Business queries, dashboard design
7. **Workflow Orchestration** - Job scheduling, dependencies

---

## 👨‍🏫 Next Steps

1. **Run the pipeline** - Execute all notebooks in order
2. **Create dashboards** - Use SQL queries to build visualizations
3. **Schedule as job** - Automate with Databricks Workflow
4. **Experiment** - Modify transformations, add new analytics
5. **Scale up** - Try with larger datasets

**Happy Learning! 🎉**