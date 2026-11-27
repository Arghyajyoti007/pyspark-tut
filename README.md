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


**Writing DF to table in different 'modes'**
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

**Writing to an Unmanaged Delta Table (Perhaps to export to external file [DataBricks/Snowflake])**
```python
df_json.write.mode("overwrite").format("delta").save("File/delta/Unmanaged.delta")
```

**Read from Table to DataFrame**
```python
df = spark.read("SELECT * FROM lakehouse_name.PropertySales LIMIT 1000")
display(df)
```

---

## Day 7 : Basic Operations in Dataframe

```python
# Reading data from DataFrame
df = spark.read.csv("Files/property_sales.csv", header = True, inferschema = True)
```
**Display Operations**
```python
# Shows a printed representation of the DF
df.show()

# Shows a interactive view of the DF
display(df)

# Shows interactive view of first 2 rows
display(df.head(2))
```

**Exploring DataFrames**
```python
df.printSchema()

# Gives the datatype of the columns
df.dtypes

# Gives the actual schema
df.schema

# We can store the schema and use it for other tables
source_schema = df.schema
new_df_with_exsisting_schema = spark.read.csv( schema = source_schema)
```
**Column Operation**
```python
# shows all the column
df.column

# Shows a single column
df.select('Type').show()
type(df.select('Type'))

# renaming existing column
df.withColumnRenamed("Address ", "Address")
df.select('Address').show()

# Selecting multiple column
df.select(["Address","City"]).show()

# Adding Column
df = df.withColumn("2x_Sales", df["SalesPrice ($)]*2)
df.show()

# Removing Column
df.drop("2x_Sales")
df.show()

# Renaming multiple column
df_new = df.selectExpr("Address as ADD", "SalesPrice ($) as SalesPrice_USD")
df_new.show()
```

---

## Day 8 : DataFrame Filtering
filer() or where() function are used to filter the DataFrame results. 

```python
# Read Data
df = spark.read.csv("Files/properties_sales.csv", header = True)

# select data for NY City
df.filter(df["City"] == "New York").show()

# select data for non NY City
df.filter(df["City"] != "New York").show()
df.filter(~(df["City"] == "New York")).show()

# startsWith, endsWith
# startsWith
df.filter(df.City.startsWith("N")).show()
# endsWith
df.filter(df.City.endsWith("al").show()

# Multiple Condition
df.filter((df["City"] != "New York") & (df["SalesPrice_USD"]>400000)).show()
df.filter((df["City"] == "New York") | (df["SalesPrice_USD"]>400000)).show()

# Filter df if df.City is in the list cities_we_care_about
cities_we_care_about = ["New York","Amsterdam"]
df.filter(df.City.isin(cities_we_care_about)).show()

# filter df if the Type contains 'House' 
df.filter(df.Type.contains("House")).show()

# filter where 'House' appears somewhere in df.Type
df.filter(df.Type.like(%House%))

# filter where 'House' in start of df.Type
df.filter(df.Type.like(House%))

# filter where 'avenue' in end of df.Type
df.filter(df.Type.like(%avenue))

# Other ways to use SQL expressions
df.filter("City != 'Los Angeles'").show()
df.filter("City <> 'Los Angeles'").show()
```

