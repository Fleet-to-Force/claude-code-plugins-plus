---
name: databricks-reference-architecture
description: |
  Implement Databricks reference architecture with best-practice project layout.
  Use when designing new Databricks projects, reviewing architecture,
  or establishing standards for Databricks applications.
  Trigger with phrases like "databricks architecture", "databricks best practices",
  "databricks project structure", "how to organize databricks", "databricks layout".
allowed-tools: Read, Grep
version: 1.0.0
license: MIT
author: Jeremy Longshore <jeremy@intentsolutions.io>
compatible-with: claude-code, codex, openclaw
---

# Databricks Reference Architecture

## Overview
Production-ready architecture patterns for Databricks data platforms.

## Prerequisites
- Understanding of Databricks components
- Unity Catalog configured
- Asset Bundles knowledge
- CI/CD pipeline setup

## Project Structure

```
databricks-platform/
├── databricks.yml                    # Main Asset Bundle config
├── bundles/
│   ├── base.yml                      # Shared configurations
│   ├── dev.yml                       # Dev environment overrides
│   ├── staging.yml                   # Staging overrides
│   └── prod.yml                      # Production overrides
├── src/
│   ├── pipelines/                    # Data pipelines
│   │   ├── bronze/
│   │   │   ├── ingest_orders.py
│   │   │   ├── ingest_customers.py
│   │   │   └── ingest_products.py
│   │   ├── silver/
│   │   │   ├── clean_orders.py
│   │   │   ├── dedupe_customers.py
│   │   │   └── enrich_products.py
│   │   └── gold/
│   │       ├── agg_daily_sales.py
│   │       ├── agg_customer_360.py
│   │       └── agg_product_metrics.py
│   ├── ml/
│   │   ├── features/
│   │   │   └── customer_features.py
│   │   ├── training/
│   │   │   └── churn_model.py
│   │   └── serving/
│   │       └── model_inference.py
│   ├── utils/
│   │   ├── __init__.py
│   │   ├── delta.py                  # Delta Lake utilities
│   │   ├── quality.py                # Data quality checks
│   │   └── logging.py                # Structured logging
│   └── notebooks/                    # Development notebooks
│       ├── exploration/
│       └── adhoc/
├── resources/
│   ├── jobs/
│   │   ├── etl_jobs.yml
│   │   ├── ml_jobs.yml
│   │   └── quality_jobs.yml
│   ├── dlt/
│   │   └── sales_pipeline.yml
│   ├── clusters/
│   │   └── cluster_policies.yml
│   └── sql/
│       └── dashboards.yml
├── tests/
│   ├── unit/
│   │   ├── test_delta.py
│   │   └── test_quality.py
│   ├── integration/
│   │   ├── test_bronze_ingest.py
│   │   └── test_ml_pipeline.py
│   └── fixtures/
│       └── sample_data/
├── docs/
│   ├── architecture.md
│   ├── runbooks/
│   └── api/
├── infrastructure/
│   ├── terraform/
│   │   ├── main.tf
│   │   ├── unity_catalog.tf
│   │   └── workspaces.tf
│   └── scripts/
│       └── bootstrap.sh
├── .github/
│   └── workflows/
│       ├── ci.yml
│       ├── deploy-staging.yml
│       └── deploy-prod.yml
├── pyproject.toml
└── README.md
```

## Layered Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                    Presentation Layer                        │
│         (Dashboards, Reports, APIs, Applications)            │
├─────────────────────────────────────────────────────────────┤
│                    Serving Layer                             │
│          (SQL Warehouses, Model Endpoints, APIs)             │
├─────────────────────────────────────────────────────────────┤
│                    Gold Layer                                │
│     (Business Aggregations, Features, Curated Datasets)      │
├─────────────────────────────────────────────────────────────┤
│                    Silver Layer                              │
│        (Cleansed, Conformed, Deduplicated Data)              │
├─────────────────────────────────────────────────────────────┤
│                    Bronze Layer                              │
│            (Raw Data with Metadata, Immutable)               │
├─────────────────────────────────────────────────────────────┤
│                    Ingestion Layer                           │
│       (Auto Loader, DLT, Streaming, Batch Connectors)        │
├─────────────────────────────────────────────────────────────┤
│                    Source Systems                            │
│    (Databases, Files, APIs, Streaming, SaaS Applications)    │
└─────────────────────────────────────────────────────────────┘
```

## Unity Catalog Structure

```sql
-- Catalog hierarchy for data platform
CREATE CATALOG IF NOT EXISTS data_platform;

-- Environment-specific schemas
CREATE SCHEMA IF NOT EXISTS data_platform.bronze;
CREATE SCHEMA IF NOT EXISTS data_platform.silver;
CREATE SCHEMA IF NOT EXISTS data_platform.gold;
CREATE SCHEMA IF NOT EXISTS data_platform.ml_features;
CREATE SCHEMA IF NOT EXISTS data_platform.ml_models;

-- Shared reference data
CREATE SCHEMA IF NOT EXISTS data_platform.reference;

-- Governance setup
GRANT USAGE ON CATALOG data_platform TO `data-engineers`;
GRANT SELECT ON SCHEMA data_platform.gold TO `data-analysts`;
GRANT ALL PRIVILEGES ON SCHEMA data_platform.ml_features TO `data-scientists`;
```

## Key Components

### Step 1: Delta Lake Configuration
```python
# src/utils/delta.py
from pyspark.sql import SparkSession
from delta.tables import DeltaTable

class DeltaConfig:
    """Centralized Delta Lake configuration."""

    # Table properties for all tables
    DEFAULT_PROPERTIES = {
        "delta.autoOptimize.optimizeWrite": "true",
        "delta.autoOptimize.autoCompact": "true",
        "delta.deletionVectors.enabled": "true",
        "delta.enableChangeDataFeed": "true",
    }

    # Default retention for time travel
    RETENTION_HOURS = 168  # 7 days

    @staticmethod
    def create_table_with_defaults(
        spark: SparkSession,
        df,
        table_name: str,
        partition_by: list[str] = None,
        cluster_by: list[str] = None,
    ):
        """Create Delta table with standard configurations."""
        writer = df.write.format("delta")

        # Apply partitioning or clustering
        if cluster_by:
            # Liquid clustering (preferred for new tables)
            writer = writer.option("clusteringColumns", ",".join(cluster_by))
        elif partition_by:
            writer = writer.partitionBy(*partition_by)

        # Write table
        writer.saveAsTable(table_name)

        # Apply properties
        for key, value in DeltaConfig.DEFAULT_PROPERTIES.items():
            spark.sql(f"ALTER TABLE {table_name} SET TBLPROPERTIES ('{key}' = '{value}')")
```

### Step 2: Data Quality Framework
```python
# src/utils/quality.py
from pyspark.sql import DataFrame
from dataclasses import dataclass
from typing import Callable

@dataclass
class QualityCheck:
    """Data quality check definition."""
    name: str
    check_fn: Callable[[DataFrame], bool]
    severity: str = "ERROR"  # ERROR, WARNING, INFO

class QualityFramework:
    """Centralized data quality checks."""

    def __init__(self, spark):
        self.spark = spark
        self.checks: list[QualityCheck] = []
        self.results: list[dict] = []

    def add_check(self, check: QualityCheck):
        """Register a quality check."""
        self.checks.append(check)

    def run_checks(self, df: DataFrame, table_name: str) -> bool:
        """Run all registered checks."""
        all_passed = True

        for check in self.checks:
            try:
                passed = check.check_fn(df)
                self.results.append({
                    "table": table_name,
                    "check": check.name,
                    "passed": passed,
                    "severity": check.severity,
                })
                if not passed and check.severity == "ERROR":
                    all_passed = False
            except Exception as e:
                self.results.append({
                    "table": table_name,
                    "check": check.name,
                    "passed": False,
                    "error": str(e),
                })
                all_passed = False

        return all_passed

# Standard checks
def not_null_check(column: str) -> QualityCheck:
    return QualityCheck(
        name=f"not_null_{column}",
        check_fn=lambda df: df.filter(f"{column} IS NULL").count() == 0,
    )

def unique_check(columns: list[str]) -> QualityCheck:
    cols = ", ".join(columns)
    return QualityCheck(
        name=f"unique_{cols}",
        check_fn=lambda df: df.groupBy(columns).count().filter("count > 1").count() == 0,
    )

def range_check(column: str, min_val, max_val) -> QualityCheck:
    return QualityCheck(
        name=f"range_{column}",
        check_fn=lambda df: df.filter(f"{column} < {min_val} OR {column} > {max_val}").count() == 0,
    )
```

### Step 3: Job Template
```yaml
# resources/jobs/etl_job_template.yml
resources:
  jobs:
    ${job_name}:
      name: "${bundle.name}-${job_name}-${bundle.target}"

      tags:
        domain: ${domain}
        tier: ${tier}
        owner: ${owner}
        environment: ${bundle.target}

      schedule:
        quartz_cron_expression: ${schedule}
        timezone_id: "UTC"

      max_concurrent_runs: 1
      timeout_seconds: ${timeout_seconds}

      queue:
        enabled: true

      email_notifications:
        on_failure: ${alert_emails}

      health:
        rules:
          - metric: RUN_DURATION_SECONDS
            op: GREATER_THAN
            value: ${duration_threshold}

      tasks:
        - task_key: ${task_name}
          job_cluster_key: ${cluster_key}
          libraries:
            - whl: ../artifacts/data_platform/*.whl
          notebook_task:
            notebook_path: ${notebook_path}
            base_parameters:
              catalog: ${var.catalog}
              schema: ${var.schema}
              run_date: "{{job.parameters.run_date}}"
```

### Step 4: Monitoring Dashboard
```sql
-- Create monitoring views for operational dashboard

-- Job health summary
CREATE OR REPLACE VIEW data_platform.monitoring.job_health AS
SELECT
    job_name,
    DATE(start_time) as run_date,
    COUNT(*) as total_runs,
    SUM(CASE WHEN result_state = 'SUCCESS' THEN 1 ELSE 0 END) as successes,
    SUM(CASE WHEN result_state = 'FAILED' THEN 1 ELSE 0 END) as failures,
    AVG(duration) / 60000 as avg_duration_minutes,
    MAX(duration) / 60000 as max_duration_minutes
FROM system.lakeflow.job_run_timeline
WHERE start_time > current_timestamp() - INTERVAL 7 DAYS
GROUP BY job_name, DATE(start_time);

-- Data freshness tracking
CREATE OR REPLACE VIEW data_platform.monitoring.data_freshness AS
SELECT
    table_catalog,
    table_schema,
    table_name,
    MAX(commit_timestamp) as last_update,
    TIMESTAMPDIFF(HOUR, MAX(commit_timestamp), current_timestamp()) as hours_since_update
FROM system.information_schema.table_history
GROUP BY table_catalog, table_schema, table_name;
```

## Data Flow Diagram

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Sources   │────▶│  Ingestion  │────▶│   Bronze    │
│ (S3, JDBC)  │     │ (Auto Loader│     │  (Raw Data) │
└─────────────┘     │  DLT, APIs) │     └──────┬──────┘
                    └─────────────┘            │
                                               ▼
                    ┌─────────────┐     ┌─────────────┐
                    │   Silver    │◀────│  Transform  │
                    │  (Cleansed) │     │   (Spark)   │
                    └──────┬──────┘     └─────────────┘
                           │
                           ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   Serve     │◀────│    Gold     │◀────│  Aggregate  │
│ (Warehouse, │     │ (Analytics) │     │   (Spark)   │
│   APIs)     │     └─────────────┘     └─────────────┘
└─────────────┘
```

## Instructions

### Step 1: Initialize Project
```bash
# Clone template
git clone https://github.com/databricks/bundle-examples.git
cd bundle-examples/default-python

# Customize for your project
mv databricks.yml.template databricks.yml
```

### Step 2: Configure Environments
Set up dev, staging, and prod targets in `databricks.yml`.

### Step 3: Implement Pipelines
Create Bronze, Silver, Gold pipelines following medallion architecture.

### Step 4: Add Quality Checks
Integrate data quality framework into each layer.

## Output
- Structured project layout
- Medallion architecture implemented
- Data quality framework integrated
- Monitoring dashboards ready

## Error Handling
| Issue | Cause | Solution |
|-------|-------|----------|
| Circular dependencies | Wrong layering | Review medallion flow |
| Config not loading | Wrong paths | Verify bundle includes |
| Quality check failures | Bad data | Add data quarantine |
| Schema drift | Source changes | Enable schema evolution |

## Examples

### Quick Setup Script
```bash
#!/bin/bash
# Initialize reference architecture

mkdir -p src/{pipelines/{bronze,silver,gold},ml,utils,notebooks}
mkdir -p resources/{jobs,dlt,clusters}
mkdir -p tests/{unit,integration}
mkdir -p infrastructure/terraform
mkdir -p docs/runbooks

# Create initial files
touch src/utils/{__init__,delta,quality,logging}.py
touch resources/jobs/etl_jobs.yml
touch databricks.yml

echo "Reference architecture initialized!"
```

## Resources
- [Databricks Architecture](https://docs.databricks.com/lakehouse-architecture/index.html)
- [Medallion Architecture](https://www.databricks.com/glossary/medallion-architecture)
- [Asset Bundles](https://docs.databricks.com/dev-tools/bundles/index.html)
- [Unity Catalog Best Practices](https://docs.databricks.com/data-governance/unity-catalog/best-practices.html)

## Flagship Skills
For multi-environment setup, see `databricks-multi-env-setup`.
