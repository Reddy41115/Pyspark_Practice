# PySpark Complete Practice Guide

This notebook covers important PySpark concepts used in real-world Data Engineering projects such as:

- Data Reading
- Transformations
- Joins
- Window Functions
- UDF
- Data Writing
- Spark SQL

---

# 1️⃣ Data Reading

## Reading JSON File

Reads a JSON file into a Spark DataFrame.

```python
df_json = spark.read.format('json')\
    .option('inferSchema',True)\
    .option('header',True)\
    .load('/FileStore/tables/drivers.json')

df_json.display()
```

---

## Reading CSV File

Reads a CSV file with automatic schema detection.

```python
df = spark.read.format('csv')\
    .option('inferSchema',True)\
    .option('header',True)\
    .load('/FileStore/tables/BigMart_Sales.csv')

df.display()
```

---

# 2️⃣ Schema Definition

## DDL Schema (Manual Schema Definition)

Instead of auto-detecting schema, we define column names and data types manually.

```python
my_ddl_schema = '''
Item_Identifier STRING,
Item_Weight STRING,
Item_Fat_Content STRING,
Item_Visibility DOUBLE,
Item_Type STRING,
Item_MRP DOUBLE
'''

df = spark.read.format('csv')\
    .schema(my_ddl_schema)\
    .option('header',True)\
    .load('/FileStore/tables/BigMart_Sales.csv')
```

---

# 3️⃣ Transformations

Transformations are operations applied to DataFrames.

---

## Select

Select specific columns.

```python
df.select('Item_Identifier','Item_Weight').display()
```

---

## Filter

Filter rows based on condition.

```python
df.filter(col('Item_Fat_Content')=='Regular').display()
```

---

## withColumn

Adds or modifies a column.

```python
df = df.withColumn('flag',lit("new"))
```

---

## Sort

Sort DataFrame in ascending or descending order.

```python
df.sort(col('Item_Weight').desc()).display()
```

---

## Drop

Removes columns from DataFrame.

```python
df.drop('Item_Visibility').display()
```

---

# 4️⃣ Group By & Aggregations

Used to perform aggregations like sum, avg, count.

```python
df.groupBy('Item_Type')\
  .agg(sum('Item_MRP').alias('Total_MRP'))\
  .display()
```

---

# 5️⃣ Pivot

Converts row values into columns.

```python
df.groupBy('Item_Type')\
  .pivot('Outlet_Size')\
  .agg(avg('Item_MRP'))\
  .display()
```

---

# 6️⃣ Joins

Combines two DataFrames based on a condition.

## Inner Join

```python
df1.join(df2, df1['dept_id']==df2['dept_id'],'inner').display()
```

## Left Join

```python
df1.join(df2, df1['dept_id']==df2['dept_id'],'left').display()
```

## Anti Join

Returns unmatched rows from left table.

```python
df1.join(df2, df1['dept_id']==df2['dept_id'],'anti').display()
```

---

# 7️⃣ Window Functions

Used for ranking and cumulative calculations.

## Row Number

```python
from pyspark.sql.window import Window

df.withColumn('row_number',
    row_number().over(Window.orderBy('Item_Identifier'))).display()
```

## Rank vs Dense Rank

```python
df.withColumn('rank',
    rank().over(Window.orderBy(col('Item_Identifier').desc())))\
  .withColumn('denseRank',
    dense_rank().over(Window.orderBy(col('Item_Identifier').desc())))\
  .display()
```

---

# 8️⃣ UDF (User Defined Function)

Used when built-in Spark functions are not enough.

```python
def my_func(x):
    return x*x

my_udf = udf(my_func)

df.withColumn('square_value', my_udf('Item_MRP')).display()
```

---

# 9️⃣ Handling Null Values

## Drop Nulls

```python
df.dropna().display()
```

## Fill Nulls

```python
df.fillna('NotAvailable').display()
```

---

# 🔟 Data Writing

## Write CSV

```python
df.write.format('csv')\
    .mode('overwrite')\
    .option('path','/FileStore/tables/CSV/data.csv')\
    .save()
```

## Write Parquet

```python
df.write.format('parquet')\
    .mode('overwrite')\
    .saveAsTable('my_table')
```

---

# 1️⃣1️⃣ Spark SQL

Create temporary view and query using SQL.

```python
df.createTempView('my_view')

df_sql = spark.sql("""
select * from my_view 
where Item_Fat_Content = 'Lf'
""")

df_sql.display()
```

---

# ✅ Conclusion

This notebook demonstrates:

- Reading structured data (CSV, JSON)
- Data transformation techniques
- Aggregations & pivot
- Joins
- Window functions
- UDF creation
- Writing data
- Spark SQL usage

These are core PySpark concepts used in real-world Azure Databricks Data Engineering projects.
