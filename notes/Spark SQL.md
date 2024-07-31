# Spark SQL

Spark SQL introduces an abstraction known as **Dataset**, which serves as a distributed SQL engine.\
Datasets offer enhanced performance compared to RDD-based programs and are designed for structured data.\
A **Dataset** is a distributed collection of structured data, while a **DataFrame** is a specialized Dataset organized into named columns (i.e., a Dataset of `Row` objects).

```java
SparkSession spark = SparkSession.builder().appName("example").getOrCreate();
spark.stop();
```

- [DataFrames](#dataframes)
- [Datasets](#datasets)
- [Dataset Operations](#dataset-operations)
- [Aggregate Functions](#aggregate-functions)
- [Datasets, DataFrames, and SQL](#datasets-dataframes-and-sql)
- [Saving Datasets and DataFrames](#saving-datasets-and-dataframes)
- [User Defined Functions (UDFs)](#user-defined-functions-udfs)

## DataFrames

A DataFrame is a distributed collection of data arranged into named columns, similar to a relational table (`Dataset<Row>`).

- **Loading from CSV:**

   ```java
   DataFrameReader reader = spark.read().format("csv").option("header", true).option("inferSchema", true);
   Dataset<Row> df = reader.load("persons.csv");
   ```

- **Loading from JSON:**
   Supports JSON Lines format (newline-delimited JSON)

   ```java
   DataFrameReader reader = spark.read().format("json");
   // For multiple files: DataFrameReader reader = spark.read().format("json").option("multiline", true);
   Dataset<Row> df = reader.load("persons.json");
   // For multiple files: Dataset<Row> df = reader.load("folder_JSONFiles/");
   ```

- **Converting DataFrame to RDD:**
   Useful methods of the `Row` class include `fieldIndex(String)`, `getAs(String)`, `getString(int)`, `getDouble(int)`

   ```java
   JavaRDD<Row> rdd = df.javaRDD();
   JavaRDD<String> namesRDD = rdd.map(row -> (String) row.getAs("name"));
   // Alternatively: .map(row -> row.getString(row.fieldIndex("name")));
   ```

## Datasets

**Datasets** are more flexible than DataFrames, representing collections of objects. They offer greater efficiency compared to RDDs, thanks to specialized encoders for object serialization and transmission. Objects stored in Datasets must be JavaBean-compliant (implementing `Serializable` with private attributes and public getters/setters).

- **From a Collection:**

   ```java
   public class Person implements Serializable {
      // private attributes and public getters/setters
   }
   List<Person> personList = new ArrayList<>(); // Add persons to the list
   Encoder<Person> encoder = Encoders.bean(Person.class); // Default encoders include Encoders.INT(), Encoders.STRING()
   Dataset<Person> dataset = spark.createDataset(personList, encoder);
   ```

- **From a DataFrame:**

   ```java
   DataFrameReader reader = spark.read().format("csv").option("header", true).option("inferSchema", true);
   Dataset<Row> df = reader.load("persons.csv");
   Encoder<Person> encoder = Encoders.bean(Person.class);
   Dataset<Person> dataset = df.as(encoder);
   ```

- **From CSV or JSON:**
   Load a DataFrame from input files and convert it into a Dataset using the `.as()` method with an encoder.

- **From RDDs:**
   Note that the first parameter should be a Scala RDD, not a JavaRDD.

## Dataset Operations

Key operations include `show()`, `printSchema()`, `count()`, `distinct()`, `select()`, `filter()`, `map()`, `flatMap()`, etc.

- **Show:**
   Displays the first n Rows or all Rows of the Dataset.

   ```java
   dataset.show(2);
   dataset.show();
   ```

- **PrintSchema:**
   Displays the schema of the Dataset.

   ```java
   dataset.printSchema();
   ```

- **Count:**
   Returns the number of rows in the Dataset (as a `Long`).

   ```java
   System.out.println("Total number of persons: " + dataset.count());
   ```

- **Distinct:**
   Returns a new Dataset containing only unique rows.

   ```java
   Dataset<String> uniqueNames = dataset.distinct();
   ```

- **Select:**
   Returns a new DataFrame containing specified columns.

   ```java
   Dataset<Row> namesAndAges = dataset.select("name", "age");
   ```

- **SelectExpr:**
   Returns a new DataFrame with columns computed by expressions.

   ```java
   Dataset<Row> updatedDataset = dataset.selectExpr("name", "age", "gender", "age + 1 as newAge");
   ```

- **Map:**
   Encoder is used for encoding the resulting objects.

   ```java
   Dataset<PersonNewAge> updatedDataset = dataset.map(person -> {
      PersonNewAge newPerson = new PersonNewAge();
      newPerson.setName(person.getName());
      newPerson.setAge(person.getAge());
      newPerson.setGender(person.getGender());
      newPerson.setNewAge(person.getAge() + 1);
      return newPerson;
   }, Encoders.bean(PersonNewAge.class));
   ```

- **Filter:**
   Filters rows based on conditions.

   ```java
   Dataset<Person> filteredDataset = dataset.filter("age >= 20 and age <= 31");
   ```

- **Filter with Lambdas:**
   Filters using lambda expressions.

   ```java
   Dataset<Person> filteredDataset = dataset.filter(person -> person.getAge() >= 20 && person.getAge() <= 31);
   ```

- **Where:**
   An alias for filter using expressions.

- **Join:**
   Joins two Datasets based on common columns.

   ```java
   Dataset<Row> joinedDataset = datasetPersons.join(datasetUidSports, datasetPersons.col("uid").equalTo(datasetUidSports.col("uid")), "inner");
   // Different join options, e.g., leftanti for subtraction
   ```

## Aggregate Functions

Perform aggregate operations on column values, such as `avg(column)`, `count(column)`, `sum(column)`, `abs(column)`.\
The `agg(aggregate functions)` method specifies which aggregate functions to apply.

```java
Dataset<Row> ageStatistics = dataset.agg(avg("age"), count("*"));
```

- **GroupBy:**
   Groups data by columns and computes aggregate functions for each group.

   ```java
   RelationalGroupedDataset groupedDataset = dataset.groupBy("name");
   Dataset<Row> averageAgeByName = groupedDataset.avg("age");
   Dataset<Row> ageCountByName = groupedDataset.agg(avg("age"), count("name"));
   ```

- **Sort:**
   Returns a new Dataset sorted by specified columns.

   ```java
   Dataset<Person> sortedDataset = dataset.sort(new Column("age").desc(), new Column("name"));
   // .desc() is used for descending order
   ```

## Datasets, DataFrames, and SQL

Spark SQL allows querying Datasets using SQL. Each Dataset must be assigned a table name.

```java
dataset.createOrReplaceTempView("people");
Dataset<Row> selectedPeople = spark.sql("SELECT * FROM people WHERE age >= 20 and age <= 31");
```

## Saving Datasets and DataFrames

- **Convert to RDD and Save as Text File:**

   ```java
   dataset.javaRDD().saveAsTextFile(outputPath);
   ```

- **DataFrameWriter:**

   ```java
   dataset.write().format("csv").option("header", true).save(outputPath);
   ```

## User Defined Functions (UDFs)

UDFs can be registered using `udf().register(String name, UDF function, DataType datatype)` on `SparkSession`.

```java
spark.udf().register("length", (String name) -> name.length(), DataTypes.IntegerType);
Dataset<Row> result = spark.sql("SELECT length(name) FROM profiles");
```