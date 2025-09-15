
# Data Pipelines with Airflow

This project teaches **Apache Airflow** fundamentals by building an end-to-end data pipeline. You'll create **custom operators** to stage data, load a data warehouse, and run data quality checks.

## Project Components
- **DAG Template:** Includes task skeletons and imports; configure task dependencies.
- **Operators:** Four custom operators for staging, fact/dimension loads, and data quality.
- **Helpers:** SQL transformations for ETL operations.

## Getting Started

### 1. Start Airflow
```bash docker-compose up -d ```

Visit http://localhost:8080 (username/password: airflow).

### 2. Configure Connections
Go to Admin → Connections

Add aws_credentials and redshift

Ensure Redshift cluster is running

### 3. DAG Execution
Add default_args: no past dependencies, 3 retries every 5 mins, catchup off, no email.

Set task dependencies according to DAG flow.

Run DAG and verify task success.

# Operator Implementation
StageToRedshiftOperator: Load JSON files from S3 to Redshift using COPY, supports backfills.

LoadFactOperator & LoadDimensionOperator: Transform and load data using SQL helpers; support append-only and truncate-insert modes.

DataQualityOperator: Run SQL tests; raise exceptions on failures.
