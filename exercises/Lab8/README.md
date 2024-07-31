# Lab 8: Analyzing Barcelona Bike Sharing Data with Spark SQL

In this lab, you'll analyze historical bike-sharing data from Barcelona stations using Spark SQL APIs. Your task is to compute the "criticality" of each station and timeslot, then select the most critical ones.

## Input Files

1. **register.csv**: Contains historical data about the bike-sharing stations.
   - **Path**: `/data/students/bigdata-01QYD/Lab7/register.csv`
   - **Format**:
     ```
     stationId\ttimestamp\tused_slots\tfree_slots
     ```
   - Example:
     ```
     23 2008-05-15 19:01:00 5 13
     ```
   - The first line contains the header.
   - Filter out lines where `used_slots = 0` and `free_slots = 0`.

2. **stations.csv**: Contains station descriptions.
   - **Path**: `/data/students/bigdata-01QYD/Lab7/stations.csv`
   - **Format**:
     ```
     stationId\tlongitude\tlatitude\tname
     ```
   - Example:
     ```
     1 2.180019 41.397978 Gran Via Corts Catalanes
     ```

## Task 1: Using Spark SQL APIs

### Steps:

1. **Setup Spark Session**:
   - Initialize a Spark session to work with DataFrames and SQL queries.

2. **Read Data**:
   - Load `register.csv` and `stations.csv` into DataFrames.
   - Use the following options for reading the CSV files:
     ```java
     Dataset<Row> inputDF = spark.read().format("csv")
       .option("delimiter", "\\t")
       .option("timestampFormat", "yyyy-MM-dd HH:mm:ss")
       .option("header", true)
       .option("inferSchema", true)
       .load(inputPath);
     ```

3. **Filter Data**:
   - Remove rows with `used_slots = 0` and `free_slots = 0` from `register.csv`.

4. **Add Columns**:
   - Extract `dayofweek` and `hour` from the `timestamp` column:
     ```sql
     SELECT 
       stationId,
       timestamp,
       used_slots,
       free_slots,
       dayofweek(timestamp) AS dayofweek,
       hour(timestamp) AS hour
     FROM 
       registerDF
     WHERE 
       used_slots > 0 AND free_slots > 0
     ```

5. **Calculate Criticality**:
   - Compute the criticality for each `(stationId, dayofweek, hour)` pair:
     ```sql
     SELECT 
       stationId,
       dayofweek,
       hour,
       COUNT(CASE WHEN free_slots = 0 THEN 1 END) / COUNT(*) AS criticality
     FROM 
       filteredDF
     GROUP BY 
       stationId, dayofweek, hour
     ```

6. **Filter by Threshold**:
   - Filter pairs with criticality greater than or equal to a minimum threshold:
     ```sql
     SELECT 
       stationId,
       dayofweek,
       hour,
       criticality
     FROM 
       criticalityDF
     WHERE 
       criticality >= minimumCriticalityThreshold
     ```

7. **Join with Stations**:
   - Join the result with `stations.csv` to add longitude and latitude:
     ```sql
     SELECT 
       a.stationId,
       a.dayofweek,
       a.hour,
       a.criticality,
       b.longitude AS stationlongitude,
       b.latitude AS stationlatitude
     FROM 
       filteredCriticalityDF a
     JOIN 
       stationsDF b
     ON 
       a.stationId = b.stationId
     ```

8. **Store Results**:
   - Save the results in CSV format with the following attributes:
     - `stationId`
     - `dayofweek`
     - `hour`
     - `criticality`
     - `stationlongitude`
     - `stationlatitude`
   - Sort by criticality in descending order. For ties, sort by `stationId` (ascending), `dayofweek` (ascending), and `hour` (ascending).

   ```java
   resultDF.orderBy(col("criticality").desc(), col("stationId").asc(), col("dayofweek").asc(), col("hour").asc())
           .write().format("csv")
           .option("header", true)
           .save(outputPath);
   ```

## Example Code Snippet

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;

public class BikeSharingAnalysis {
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("BikeSharingAnalysis")
            .getOrCreate();

        // Load datasets
        Dataset<Row> registerDF = spark.read().format("csv")
            .option("delimiter", "\\t")
            .option("timestampFormat", "yyyy-MM-dd HH:mm:ss")
            .option("header", true)
            .option("inferSchema", true)
            .load("/data/students/bigdata-01QYD/Lab7/register.csv");

        Dataset<Row> stationsDF = spark.read().format("csv")
            .option("delimiter", "\\t")
            .option("header", true)
            .option("inferSchema", true)
            .load("/data/students/bigdata-01QYD/Lab7/stations.csv");

        // Filter and compute criticality
        registerDF.createOrReplaceTempView("register");
        Dataset<Row> filteredDF = spark.sql(
            "SELECT stationId, timestamp, used_slots, free_slots, " +
            "DAYOFWEEK(timestamp) AS dayofweek, " +
            "HOUR(timestamp) AS hour " +
            "FROM register " +
            "WHERE used_slots > 0 AND free_slots > 0"
        );

        filteredDF.createOrReplaceTempView("filtered");
        Dataset<Row> criticalityDF = spark.sql(
            "SELECT stationId, dayofweek, hour, " +
            "COUNT(CASE WHEN free_slots = 0 THEN 1 END) / COUNT(*) AS criticality " +
            "FROM filtered " +
            "GROUP BY stationId, dayofweek, hour"
        );

        // Join with stations and store results
        criticalityDF.createOrReplaceTempView("criticality");
        Dataset<Row> resultDF = spark.sql(
            "SELECT a.stationId, a.dayofweek, a.hour, a.criticality, " +
            "b.longitude AS stationlongitude, b.latitude AS stationlatitude " +
            "FROM criticality a " +
            "JOIN stations b ON a.stationId = b.stationId"
        );

        resultDF.orderBy(col("criticality").desc(), col("stationId").asc(), col("dayofweek").asc(), col("hour").asc())
            .write().format("csv")
            .option("header", true)
            .save("/path/to/output");
    }
}
```

## Shutting Down JupyterHub Container

After completing your tasks:

1. Go to `File -> Hub Control Panel`.
2. Click the “Stop My Server” button and wait until it disappears.