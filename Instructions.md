# Databricks Workflow Setup Guide

## 🎯 Creating a Databricks Job for Netflix Pipeline

This guide walks you through creating a **Databricks Workflow (Job)** that orchestrates the Bronze → Silver → Gold pipeline automatically.

---

## Prerequisites

✓ All 4 notebooks created and working:  
* `01_Bronze_Netflix_Ingestion`
* `02_Silver_Netflix_Transformation`
* `03_Gold_Netflix_Analytics`
* `04_SQL_Dashboard_Queries` (optional for workflow)

✓ Notebooks have been tested manually  
✓ Unity Catalog schemas exist (`main.netflix_bronze`, `main.netflix_silver`, `main.netflix_gold`)

---

## Step-by-Step Workflow Creation

### Step 1: Navigate to Workflows

1. Click **Workflows** in the left sidebar
2. Click **Create Job** button
3. Give it a name: `Netflix_Medallion_Pipeline`

### Step 2: Add Bronze Task

1. Click **+ Add task**
2. Configure task:
   * **Task name**: `Bronze_Ingestion`
   * **Type**: Select **Notebook**
   * **Source**: Select **Workspace**
   * **Path**: Browse to `/Users/<your-email>/databricks_netflix/01_Bronze_Netflix_Ingestion`
   * **Cluster**: Choose **Serverless** (recommended) or create a cluster
3. Click **Create**

### Step 3: Add Silver Task

1. Click **+ Add task** (under the Bronze task)
2. Configure task:
   * **Task name**: `Silver_Transformation`
   * **Type**: Select **Notebook**
   * **Path**: Browse to `/Users/<your-email>/databricks_netflix/02_Silver_Netflix_Transformation`
   * **Cluster**: Use same as Bronze
   * **Depends on**: Select `Bronze_Ingestion` (creates sequential dependency)
3. Click **Create**

### Step 4: Add Gold Task

1. Click **+ Add task** (under the Silver task)
2. Configure task:
   * **Task name**: `Gold_Analytics`
   * **Type**: Select **Notebook**
   * **Path**: Browse to `/Users/<your-email>/databricks_netflix/03_Gold_Netflix_Analytics`
   * **Cluster**: Use same cluster
   * **Depends on**: Select `Silver_Transformation`
3. Click **Create**

### Step 5: Configure Schedule (Optional)

1. Click **Add trigger** in the top section
2. Choose trigger type:
   * **Scheduled**: For recurring runs
   * **File arrival**: For event-driven pipelines
   * **Manual**: No automatic schedule
3. For scheduled:
   * **Schedule type**: Periodic or Cron
   * **Periodic example**: Every day at 2:00 AM
   * **Cron example**: `0 0 2 * * ?` (2 AM daily)
   * **Timezone**: Select your timezone
4. Click **Save**

### Step 6: Configure Notifications (Optional)

1. Click **Add notification** under Email notifications
2. Configure:
   * **Recipients**: Your email
   * **Notify on**: Failure, Success, or both
3. Click **Save**

### Step 7: Review and Run

1. Review the visual task graph (should show Bronze → Silver → Gold)
2. Click **Run now** to test the workflow
3. Monitor execution in real-time
4. Check logs for each task

---

## Workflow Architecture

```
┌─────────────────────────┐
│  Bronze_Ingestion       │  ← Task 1
│  01_Bronze_Netflix...   │
└───────────┬──────────────┘
           │
           ↓ depends on
           │
┌───────────┴──────────────┐
│  Silver_Transformation  │  ← Task 2
│  02_Silver_Netflix...   │
└───────────┬──────────────┘
           │
           ↓ depends on
           │
┌───────────┴──────────────┐
│  Gold_Analytics         │  ← Task 3
│  03_Gold_Netflix...     │
└─────────────────────────┘
```

---

## Using Databricks CLI (Alternative Method)

You can also create the job using the Databricks CLI:

```bash
# Install Databricks CLI
pip install databricks-cli

# Configure authentication
databricks configure --token

# Create job from JSON definition
databricks jobs create --json-file netflix_job_config.json
```

**netflix_job_config.json**:
```json
{
  "name": "Netflix_Medallion_Pipeline",
  "tasks": [
    {
      "task_key": "Bronze_Ingestion",
      "notebook_task": {
        "notebook_path": "/Users/<your-email>/databricks_netflix/01_Bronze_Netflix_Ingestion",
        "source": "WORKSPACE"
      },
      "new_cluster": {
        "spark_version": "13.3.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    },
    {
      "task_key": "Silver_Transformation",
      "depends_on": [{"task_key": "Bronze_Ingestion"}],
      "notebook_task": {
        "notebook_path": "/Users/<your-email>/databricks_netflix/02_Silver_Netflix_Transformation",
        "source": "WORKSPACE"
      },
      "new_cluster": {
        "spark_version": "13.3.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    },
    {
      "task_key": "Gold_Analytics",
      "depends_on": [{"task_key": "Silver_Transformation"}],
      "notebook_task": {
        "notebook_path": "/Users/<your-email>/databricks_netflix/03_Gold_Netflix_Analytics",
        "source": "WORKSPACE"
      },
      "new_cluster": {
        "spark_version": "13.3.x-scala2.12",
        "node_type_id": "i3.xlarge",
        "num_workers": 2
      }
    }
  ],
  "schedule": {
    "quartz_cron_expression": "0 0 2 * * ?",
    "timezone_id": "America/Los_Angeles"
  },
  "email_notifications": {
    "on_failure": ["your-email@example.com"],
    "on_success": ["your-email@example.com"]
  },
  "max_concurrent_runs": 1
}
```

---

## Best Practices

### 1. Compute Selection

**Serverless (Recommended)**:
* ✅ Fastest startup time
* ✅ Auto-scaling
* ✅ No cluster management
* ✅ Cost-effective for small workloads

**Job Cluster**:
* ✅ Dedicated resources
* ✅ Better for large datasets
* ✅ Predictable performance

**Shared Cluster** (❌ Not recommended for production):
* ❌ Can be terminated by others
* ❌ Performance varies
* ❌ Good only for development

### 2. Error Handling

**Add retries**:
* Set **Max retries**: 2-3
* Set **Retry delay**: 60 seconds
* Useful for transient failures

**Add timeouts**:
* Set **Timeout**: 1 hour per task
* Prevents stuck jobs

### 3. Monitoring

**Enable notifications**:
* ✅ On failure (critical)
* ✅ On success (optional)
* ✅ Include run logs in email

**Check job history**:
* View **Runs** tab for historical executions
* Analyze runtime trends
* Identify optimization opportunities

### 4. Cost Optimization

**Use auto-termination**:
* Set cluster **Auto-termination**: 10 minutes
* Prevents idle cluster costs

**Right-size compute**:
* Start small (1-2 workers)
* Scale up based on actual runtime
* Use Spot/Preemptible instances for dev

**Use shared clusters for dev**:
* Bronze/Silver/Gold can share same cluster
* Faster execution (no startup time)
* Lower cost

---

## Troubleshooting

### Job Fails on Bronze Task

**Common causes**:
* CSV download URL unreachable
* Schema creation permissions issue
* Cluster startup timeout

**Solutions**:
* Check internet connectivity
* Verify Unity Catalog permissions
* Increase cluster startup timeout
* Use cached dataset instead of downloading

### Job Fails on Silver Task

**Common causes**:
* Bronze table doesn't exist
* Schema mismatch
* Memory issues

**Solutions**:
* Verify Bronze task completed successfully
* Check Bronze table exists: `SELECT * FROM main.netflix_bronze.raw_titles LIMIT 10`
* Increase cluster size if OOM errors

### Job Fails on Gold Task

**Common causes**:
* Silver table doesn't exist
* Aggregation OOM
* OPTIMIZE command timeout

**Solutions**:
* Verify Silver task completed
* Remove OPTIMIZE commands for initial testing
* Increase cluster memory

### Job Runs Too Long

**Optimizations**:
* Use Photon acceleration (Databricks Runtime)
* Cache intermediate DataFrames (`.cache()`)
* Reduce data volume for testing
* Partition Gold tables

---

## Production Enhancements

### 1. Incremental Processing

Modify Bronze to only load new data:
```python
# Add to Bronze notebook
last_run_timestamp = spark.sql("""
    SELECT MAX(ingestion_timestamp) 
    FROM main.netflix_bronze.raw_titles
""").collect()[0][0]

# Only process files newer than last_run_timestamp
```

### 2. Data Quality Gates

Add validation task between layers:
```python
# Create validation notebook
def validate_bronze():
    row_count = spark.table("main.netflix_bronze.raw_titles").count()
    assert row_count > 0, "Bronze table is empty!"
    assert row_count < 1000000, "Unexpected row count!"
```

### 3. Alerting & Monitoring

* Integrate with PagerDuty or Slack
* Create custom metrics dashboard
* Track data freshness SLAs

### 4. Parameterization

Add job parameters for flexibility:
```python
# Access in notebook
dbutils.widgets.text("environment", "prod")
env = dbutils.widgets.get("environment")

# Use in table names
table_name = f"main.netflix_{env}_bronze.raw_titles"
```

---

## Next Steps

✅ **Test the workflow**: Run manually and verify all tasks complete  
✅ **Schedule it**: Set daily/hourly schedule  
✅ **Monitor for a week**: Check for failures  
✅ **Optimize performance**: Add caching, partitioning  
✅ **Add alerting**: Email/Slack notifications  
✅ **Document runbooks**: Troubleshooting guides for operators  

---

## Learning Resources

* [Databricks Workflows Documentation](https://docs.databricks.com/workflows/index.html)
* [Job Scheduling Best Practices](https://docs.databricks.com/workflows/jobs/jobs-best-practices.html)
* [Unity Catalog Permissions](https://docs.databricks.com/data-governance/unity-catalog/manage-privileges/index.html)
* [Delta Lake Optimization](https://docs.databricks.com/delta/optimize.html)

---

**Congratulations! You've built a production-ready data pipeline with automated orchestration! 🎉**