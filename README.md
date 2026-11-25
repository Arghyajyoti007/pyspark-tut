# Pyspark Tutorial

## Day 4 : DataFrame
- Dataframe is a distributed, in memory table like structure with column and schema. 
- Its inspired by Pandas Dataframe. 
- Schema defines the column and column data type. 

**Creating a Dataframe**
```
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
```
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

```

```
