# Pyspark Tutorial

## Day 4 : DataFrame
- Dataframe is a distributed, in memory table like structure with column and schema. 
- Its inspired by Pandas Dataframe. 
- Schema defines the column and column data type. 

**Creating a Dataframe**
```python
from pyspark.sql.type import StructType, StructField, StringType, IntegerType

data2 = [
  ("Arghyajyoti","Samui","M",24),
  ("John", "Doe","M",25)  
]

schema = StructType(
  [
  StructField("FirstName", StringType(), True),
  StructField("LasttName", StringType(), True),
  StructField("Gender", StringType(), True),
  StructField("Age", IntegerType(), True)
]
)

df = spark.createDataFrame(data = data2, schema = schema)
df.printSchema()
df.show(truncate = False)
type(df)
```

**Creating Dataframe from DDL**
```python
schema_ddl = "first_name STRING, last_name STRING, gender STRING, age STRING"
df2 = spark.createDataFrame(data = data2, schema = schema_ddl)
df2.printSchema()
```
---

## Day 5 : Reading and Creating Dataframe from different file
**Reading CSV**
Using property-sales.csv, which looks like this:
Address,Type,City,SalePrice ($),Agent
1 Rowley Street ,Detached House,New York,745000,Penelope Pullman
13a lollipop avenue,Apartment,Los Angeles,345000,Jack Smith
34 the drive,House,Atlanta,459000,Sheila Sammi

```python
path = "Files/property-sales.csv"
df_csv = spark.read.csv(path, headers = True, inferschema = True)
#inferschema = True is asking spark to do the schema matching during read, it's lazy and costly.
display(df_csv)
df_csv.dtypes # to check the df datatypes
```

**Writing into Files**
***CSV***
```python
df_csv.write.mode("overwrite").format("csv").save("Files/" + csv_table_name)
```
or
```python
df_csv.write.csv("Files/property-sales.csv", mode = "overwrite")
```

***JSON***
```python
df_csv.write.json("Files/property-sales.json", mode = "overwrite")
```

***Parquet***
```python
#reading from JSON
df_spark = spark.read.json("Files/property-sales.json")
#writing to Parquet
df_spark.write.parquet("Files/property-sales.parquet", mode = "overwrite")
#reading from Parquet
spark.read.parquet("Files/property-sales.parquet")
```
---

## Day 6 : Writing from DataFrame to Lakehouse Table
In Lakehouse we store the tables in Delta format. 

Though there are some limitations of Lakehouse column naming. 
```python
df_json = spark.read.json("Files/property-sales.json", header = True)
df_json.printSchema()

# to Rename the Column
df_json.withColumnRenamed("SalePrice ($)", "SalePrice_USD")\
      .withColumnRenamed("Address ", "Address")\
      .withColumnRenamed("City ", "City")
```


## Writing DF to table in different 'modes'
Using saveAsTable() we can save the DataFrame as **Managed Table**. That means spark manages the data as well as the metadata. Using SQL *Drop Table table_name* it can remove the data and metadata both, as spark is managing everything. 
Saving as Unmanaged Table the delete command will remove only the meta-data, not the actual data. 

```python
delta_table_name = "PropertySales"
df_json.write.mode("overwrite").format("delta").saveAsTable(delta_table_name)
```

**Modes to Save Table**
There are 4 write modes
```python
# Overwrite: Overwrites any existing table with new dataframe
df_json.write.mdoe("overwrite").format("delta").saveAsTable(delta_table_name)

# Append: Append new dataframe to the existing table
df_json.write.mode("append").format("delta").saveAsTable(delta_table_name)

# Ignore: Fails silently if data already exists
df_json.write.mode("ignore").format("delta").saveAsTable(delta_table_name)

#  Error: Throws error if data already exists
df_jspon.write.mode("error").format("delta").saveAsTable(detal_table_name)
```





