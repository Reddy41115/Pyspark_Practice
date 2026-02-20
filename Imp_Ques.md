# 🚀 Data Engineering Full Interview Notes
## ADF + Databricks + Spark (Complete Revision Guide)

---

# 1️⃣ Spark Versions & Databricks Runtime

## Spark Versions
- Spark 1.x
- Spark 2.x
- Spark 3.x (Most used in projects)

## Spark 3.x Features
- Adaptive Query Execution (AQE)
- Improved SQL Engine
- Better performance
- Dynamic partition pruning

## Databricks Runtime
- Built on Spark versions
- Includes optimizations like Photon Engine
- Includes Delta Lake

**Interview One-Liner:**  
Most projects use Spark 3.x on Databricks Runtime for performance and optimization benefits.

---

# 2️⃣ Spark Storage Levels (Caching)

Storage levels define how data is cached.

Types:
- MEMORY_ONLY
- MEMORY_AND_DISK ✅ (Most used)
- DISK_ONLY
- MEMORY_ONLY_SER
- MEMORY_AND_DISK_SER

**Why Used?**
- Avoid recomputation
- Improve performance

**Interview One-Liner:**  
In real-time projects, we mostly use MEMORY_AND_DISK for large datasets.

---

# 3️⃣ Find 3rd Highest Bonus (Current Year)

## PySpark

```python
from pyspark.sql import functions as F
from pyspark.sql.window import Window

current_year = 2026
df_filtered = df.filter(F.col("year") == current_year)

w = Window.orderBy(F.col("bonus").desc())

df_ranked = df_filtered.withColumn("rnk", F.dense_rank().over(w))

result = df_ranked.filter(F.col("rnk") == 3)\
                  .select("emp_name", "bonus")

result.show()
```

## SQL

```sql
SELECT employee_name, bonus
FROM (
    SELECT employee_name,
           bonus,
           DENSE_RANK() OVER (ORDER BY bonus DESC) AS rnk
    FROM employee_bonus
    WHERE year = YEAR(GETDATE())
) t
WHERE rnk = 3;
```

---

# 4️⃣ Word Count

## PySpark

```python
from pyspark.sql.functions import explode, split, col

df = spark.read.text("/path/file.txt")
words = df.select(explode(split(col("value"), " ")).alias("word"))
word_count = words.groupBy("word").count()
word_count.show()
```

## Python

```python
from collections import Counter

data = ["Bangalore","Hyderabad","Mumbai",
        "London","India","UK","Bangalore"]

word_count = Counter(data)

for word, cnt in word_count.items():
    print(word, cnt)
```

---

# 5️⃣ Only 90% Data Loaded in ADF – How to Fix

## Common Causes
- Wrong watermark filter
- Inner join dropping rows
- Transformation filters
- Data type errors
- Partial pipeline failure

## Steps to Fix

### 1. Compare Counts
```sql
SELECT COUNT(*) FROM source_table;
SELECT COUNT(*) FROM target_table;
```

### 2. Find Missing Records
```sql
SELECT s.id
FROM source_table s
LEFT JOIN target_table t
ON s.id = t.id
WHERE t.id IS NULL;
```

### 3. Check Copy Activity Logs
- rowsRead
- rowsCopied
- rowsSkipped

### 4. Fix Incremental Logic
Use:
```
WHERE updated_date >= last_success_date
```

### 5. Use MERGE (Upsert)

**Interview One-Liner:**  
I compare source and target counts, identify missing records using left join, fix incremental logic, and reload missing data safely.

---

# 6️⃣ Roles & Responsibilities

- Migration project handling
- Data ingestion into Bronze layer
- Data cleaning in Silver layer
- Business rule implementation
- Gold layer creation (Fact & Dimension)
- SCD implementation
- Performance tuning
- CI/CD deployment
- Monitoring & production support

---

# 7️⃣ Architecture (ADF + Databricks + ADLS)

On-Prem SQL Server  
↓  
Self-Hosted Integration Runtime  
↓  
ADF Copy Activity  
↓  
ADLS Gen2 Bronze  
↓  
Databricks Processing  
↓  
Silver Layer  
↓  
Gold Layer  
↓  
Power BI  

---

# 8️⃣ Parameters in ADF

## Local Parameters (Pipeline Level)
- Used inside one pipeline
- Example:
```
@pipeline().parameters.table_name
```

## Global Parameters (ADF Level)
- Used across pipelines
- Example:
```
@pipeline().globalParameters.env
```

## Benefits
- Reusability
- Dynamic processing
- Less duplication
- Easy maintenance

---

# 9️⃣ Read On-Prem SQL Server in ADF

## Steps
1. Install Self-Hosted Integration Runtime
2. Create SQL Server Linked Service
3. Create Dataset
4. Use Copy Activity

Other Activities:
- Lookup
- Stored Procedure
- Get Metadata
- Data Flow

---

# 🔟 Metadata in ADF

Metadata = Data about data.

Examples:
- File name
- Size
- Row count
- Last modified date
- Schema

## Activity Used
Get Metadata

## Real-Time Use
- Dynamic file loading
- Incremental loads
- File validation
- Schema validation

---

# 1️⃣1️⃣ Pipeline Failed – Safe Rerun Strategy

## Steps
1. Check last successful watermark
2. Reload from last success date
3. Use MERGE to avoid duplicates
4. Validate row counts
5. Update watermark after success

If watermark not available:
```
WHERE date >= current_date - 3
```

---

# 1️⃣2️⃣ Spark vs Databricks Engine

## Spark Engine
- Core processing engine
- Distributed computing
- Handles ETL and transformations

## Databricks Engine
- Optimized Spark platform
- Photon Engine
- Delta Lake
- Easy cluster management

---

# 1️⃣3️⃣ Spark Joins

## Types
- Inner (Default)
- Left
- Right
- Full
- Broadcast

Default Join:
```python
df1.join(df2, "id")  # Inner join
```

---

# 1️⃣4️⃣ Spark Architecture

## Components
- Driver
- Cluster Manager
- Worker Nodes
- Executors
- Tasks

## Flow
User Code  
↓  
Driver (Creates DAG)  
↓  
Cluster Manager  
↓  
Executors  
↓  
Tasks  

## Transformations
- Narrow (No shuffle)
- Wide (Shuffle)

---

# 1️⃣5️⃣ Shuffling in Spark

Shuffling = Data redistribution across partitions.

Occurs in:
- join
- groupBy
- orderBy
- distinct

## Why Costly?
- Network transfer
- Disk I/O
- Memory usage

## Default Shuffle Partitions
```python
spark.sql.shuffle.partitions = 200
```

---

# 1️⃣6️⃣ Databricks Workflows – Limitations

- Not ideal for complex orchestration
- Limited cross-system integration
- Basic monitoring
- Limited reusability
- Can be costly

Best for:
- Notebook scheduling
- ETL job execution

---

# 1️⃣7️⃣ Unity Catalog

- Centralized governance layer
- Role-based access control
- Column-level security
- Data lineage
- Audit tracking
- Storage credential management

---

# 1️⃣8️⃣ Handling Bad CSV Data in ADF

## Issues
- Comma inside values
- Schema mismatch
- Column shifts

## Best Practice
CSV → Bronze (Raw)  
↓  
Clean in Databricks  
↓  
Silver  

## In Databricks
```python
df = spark.read.option("header",True)\
               .option("inferSchema",True)\
               .csv("path")
```

Use split(), replace(), schema validation.

---

# 1️⃣9️⃣ How to Confirm Data Loaded Successfully

## 1. Compare Row Counts
```sql
SELECT COUNT(*) FROM source;
SELECT COUNT(*) FROM target;
```

## 2. Missing Record Check
```sql
SELECT s.id
FROM source s
LEFT JOIN target t
ON s.id = t.id
WHERE t.id IS NULL;
```

## 3. Audit Table
Store:
- source_count
- target_count
- load_date
- status

## 4. Checksum Validation (Large Data)

---

# 🔥 Final Rapid Revision Points

- Bronze → Silver → Gold Architecture
- Use watermark for incremental load
- Use MERGE to prevent duplicates
- MEMORY_AND_DISK for caching
- Default join = Inner
- Default shuffle partitions = 200
- Use Self-Hosted IR for on-prem SQL
- Always validate source vs target counts
- Maintain audit table
- Use metadata-driven pipelines

---

# ✅ END OF FULL INTERVIEW NOTES
