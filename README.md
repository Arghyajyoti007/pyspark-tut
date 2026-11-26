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
df_csv.display()
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

