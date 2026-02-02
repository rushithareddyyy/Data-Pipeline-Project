# Data Pipeline with Apache Airflow 🔄

Build a **production-ready end-to-end data pipeline** that downloads data from AWS S3, transforms it, loads it into Redshift, and runs quality checks - all orchestrated with Apache Airflow. Learn data engineering by building real systems!

---

## What's This About? 🎯

Imagine you need to move and process **millions of records** from AWS S3 to your data warehouse every single day. And it needs to:
- ✅ Run automatically at 2 AM
- ✅ Retry if something breaks
- ✅ Check data quality
- ✅ Alert you if there's a problem
- ✅ Show pretty dashboard

That's exactly what this project teaches you! Apache Airflow is the tool that makes it possible. 📊

---

## Architecture 🏗️

```
AWS S3 (Raw Data)
    ↓
[Staging Operator] → Load JSON to Redshift
    ↓
[Fact Operator] → Create analytics fact table
    ↓
[Dimension Operator] → Create dimension tables
    ↓
[Quality Operator] → Run validation checks
    ↓
Redshift Data Warehouse (Analytics Ready!)
```

---

## What You'll Learn 🧠

✅ How to create DAGs (data pipelines)  
✅ Custom operators for specific tasks  
✅ Task dependencies and scheduling  
✅ Error handling and automatic retries  
✅ Data quality validation  
✅ Monitoring with Airflow UI  
✅ Production-ready pipeline patterns  

---

## Quick Start (15 mins) 🚀

### Step 1: Prerequisites
- Docker & Docker Compose installed
- AWS Account (S3 + Redshift)
- Python 3.8+

### Step 2: Start Airflow
```bash
docker-compose up -d
```

Access at: `http://localhost:8080`  
Default login: `airflow` / `airflow`

### Step 3: Configure AWS Credentials
1. Go to **Admin → Connections**
2. Create connection: `aws_credentials`
   - Type: Amazon Web Services
   - Login: Your AWS Access Key ID
   - Password: Your AWS Secret Access Key
3. Create connection: `redshift`
   - Type: Postgres
   - Host: your-redshift-endpoint.region.redshift.amazonaws.com
   - Database: your_database_name
   - User: awsuser
   - Password: Your Redshift password
   - Port: 5439

### Step 4: Deploy Your DAG
Copy `udacity_project_dag.py` to `airflow/dags/` folder

### Step 5: Trigger & Monitor
1. Go to Airflow UI
2. Find your DAG
3. Click "Trigger DAG"
4. Watch it run in real-time!

---

## Project Components 🔧

### 1. **DAG Template** 
Skeleton pipeline with all tasks connected:
```python
from airflow import DAG
from datetime import datetime, timedelta

default_args = {
    'owner': 'you',
    'start_date': datetime(2024, 1, 1),
    'retries': 3,
    'retry_delay': timedelta(minutes=5)
}

dag = DAG('my_data_pipeline', default_args=default_args, schedule_interval='@daily')
```

### 2. **Custom Operators**

**StageToRedshiftOperator**
- Loads JSON files from S3 → Redshift
- Uses COPY command (super fast!)
- Supports backfills for historical data

**LoadFactOperator**
- Creates fact tables from staging data
- Supports append-only or truncate-insert modes
- Calculates metrics and aggregations

**LoadDimensionOperator**
- Loads dimension tables
- Handles slowly changing dimensions
- Maintains referential integrity

**DataQualityOperator**
- Runs SQL validation tests
- Checks row counts, nulls, relationships
- Fails pipeline if quality checks fail

### 3. **SQL Helpers**
Pre-written SQL for all transformations in `sql_queries.py`

---

## File Structure 📁

```
.
├── docker-compose.yml              # Airflow + Postgres setup
├── airflow/
│   ├── dags/
│   │   └── udacity_project_dag.py  # Your main DAG
│   ├── plugins/
│   │   ├── operators/              # Custom operators
│   │   │   ├── stage_redshift.py
│   │   │   ├── load_fact.py
│   │   │   ├── load_dimension.py
│   │   │   └── data_quality.py
│   │   ├── helpers/
│   │   │   └── sql_queries.py      # SQL transformations
│   │   └── __init__.py
├── create_tables.sql               # Initial database schema
├── final_project.py                # Configuration
└── README.md
```

---

## Building Your First Pipeline ▶️

### Step 1: Create Database Tables
```bash
psql -h your-redshift-endpoint -U awsuser -d your_database -f create_tables.sql
```

### Step 2: Import Operators
```python
from airflow import DAG
from airflow.operators.dummy_operator import DummyOperator
from airflow.operators import StageToRedshiftOperator, LoadFactOperator, LoadDimensionOperator, DataQualityOperator
```

### Step 3: Create DAG with Default Args
```python
default_args = {
    'owner': 'udacity',
    'start_date': datetime(2024, 1, 1),
    'end_date': datetime(2024, 3, 31),
    'email': ['airflow@example.com'],
    'email_on_failure': False,
    'email_on_retry': False,
    'retries': 3,
    'retry_delay': timedelta(minutes=5),
    'catchup': False,
    'depends_on_past': False
}

dag = DAG(
    'udacity_project_dag',
    default_args=default_args,
    description='Load and transform data in Redshift with Airflow',
    schedule_interval='@daily'
)
```

### Step 4: Create Tasks
```python
# Start task
start_operator = DummyOperator(task_id='Begin_execution', dag=dag)

# Stage events from S3
stage_events = StageToRedshiftOperator(
    task_id='Stage_events',
    table='staging_events',
    s3_bucket='my-bucket',
    s3_key='s3://path/to/events',
    redshift_conn_id='redshift',
    aws_credentials_id='aws_credentials',
    dag=dag
)

# Load fact table
load_songplays_fact = LoadFactOperator(
    task_id='Load_songplays_fact_table',
    table='songplays',
    sql_query='SELECT * FROM staging_events',
    append_mode=False,
    dag=dag
)

# Run quality checks
run_quality_checks = DataQualityOperator(
    task_id='Run_data_quality_checks',
    sql_checks=[
        'SELECT COUNT(*) FROM songplays WHERE songid IS NULL',
        'SELECT COUNT(*) FROM users WHERE user_id IS NULL'
    ],
    dag=dag
)

# End task
end_operator = DummyOperator(task_id='Stop_execution', dag=dag)
```

### Step 5: Define Dependencies
```python
start_operator >> stage_events
stage_events >> load_songplays_fact
load_songplays_fact >> run_quality_checks
run_quality_checks >> end_operator
```

---

## Monitoring Your Pipeline 📊

### Airflow UI Dashboard

**Tree View** - See pipeline history and status  
**Graph View** - Visualize task dependencies  
**Log View** - Debug individual task runs  
**Calendar View** - See execution history  

### Key Metrics to Track
- ✅ Task duration (is it slow?)
- ✅ Retry count (is something flaky?)
- ✅ Success/failure rate
- ✅ Data volume processed
- ✅ SLA (Service Level Agreement) violations

---

## Data Quality Checks ✅

Built-in validation ensures your data is correct:

```python
# Check for nulls
SELECT COUNT(*) FROM songplays WHERE songid IS NULL

# Check row counts
SELECT COUNT(*) FROM users

# Check unique values
SELECT COUNT(DISTINCT user_id) FROM songplays

# Check data ranges
SELECT COUNT(*) FROM events WHERE event_time < '2024-01-01'
```

**If any check fails:** Pipeline stops and alerts you! 🚨

---

## Best Practices 🎯

✅ **Idempotent Tasks** - Can run 100x without issues  
✅ **Clear Naming** - Task names describe what they do  
✅ **Error Handling** - Retries on transient failures  
✅ **Monitoring** - Alerts on real failures  
✅ **Backfills** - Handle historical data easily  
✅ **SLA Warnings** - Know when tasks are slow  
✅ **Documentation** - Comment your code  

---

## Common Issues & Solutions 🔧

**Q: "Connection refused to Redshift"**
- Verify AWS credentials in Airflow Connections
- Check Redshift cluster is running
- Ensure security group allows outbound access

**Q: "Table already exists"**
- This is normal! Use `DROP TABLE IF EXISTS` in schema
- Or use different table name
- Or truncate existing table

**Q: "Data looks wrong"**
- Check SQL transformations in `sql_queries.py`
- Run data quality checks
- Review logs for errors

**Q: "Pipeline too slow"**
- Check task duration in Airflow UI
- Consider Redshift cluster size
- Optimize SQL queries
- Parallelize independent tasks

---

## Advanced Topics 🚀

### Setting SLAs (Service Level Agreements)
```python
'sla': timedelta(hours=2)  # Alert if task takes > 2 hours
```

### Email Notifications
```python
'email': ['your_email@example.com'],
'email_on_failure': True,
'email_on_retry': True
```

### Backfill Historical Data
```bash
airflow backfill udacity_project_dag --start-date 2024-01-01 --end-date 2024-03-31
```

### Testing DAGs
```bash
airflow list_dags
airflow test my_dag my_task 2024-01-01
```

---

## Next Steps 🔮

### Features to Add
- [ ] Email alerts on failures
- [ ] Auto-scaling Redshift cluster
- [ ] Data lineage tracking
- [ ] Metric dashboards
- [ ] Slack notifications
- [ ] Incremental loads

### To Deploy to Production
- Use managed Airflow (AWS MWAA, Cloud Composer)
- Set up monitoring and alerting
- Implement high availability
- Add backup and recovery
- Set up version control

---

## Requirements 📦

```
apache-airflow==2.0.0
apache-airflow-providers-amazon==3.0.0
apache-airflow-providers-postgres==3.0.0
psycopg2-binary==2.9.0
```

---

## Resources 📚

- [Airflow Documentation](https://airflow.apache.org/)
- [AWS Redshift Guide](https://docs.aws.amazon.com/redshift/)
- [SQL Best Practices](https://docs.aws.amazon.com/redshift/latest/dg/best-practices.html)

---

## License 📜

MIT License - Learn, modify, share freely!

---

## Made By

**Rushitha**  
Data Engineer | Airflow Expert | Open Source Contributor

Have questions? [Open an issue](https://github.com/rushithareddyyy/Data-Pipeline-Project/issues)  
Want to contribute? [Submit a PR](https://github.com/rushithareddyyy/Data-Pipeline-Project/pulls)

**Last Updated:** February 2026  
**Status:** ✅ Active & Maintained
