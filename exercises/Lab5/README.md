# Lab 5: Introduction to Apache Spark

In this lab, we will start working with Apache Spark to filter and analyze a dataset. We will use the provided Java template to build a Spark application.

## Setup

1. **Template**: Download and unzip the provided Spark template.
2. **Import Project**: Open your IDE (e.g., VS Code) and import the project using the "import project" option.
3. **Modify and Export**: Modify the template to implement the required functionalities and export a JAR file.
4. **Submit Job**: Use `spark-submit` to submit the job to the cluster.

### Spark Submission Command

Submit your Spark job using the following command:

```sh
spark-submit --class it.polito.bigdata.spark.example.SparkDriver -deploy-mode client --master yarn SparkProject.jar arguments
```

In this command:
- Replace `SparkProject.jar` with the name of your JAR file.
- Replace `arguments` with the arguments for your application.

## Tasks

### Task 1: Basic Filtering

1. **Filter by Prefix**:
   - Your application should filter lines in the dataset to keep only those that start with a user-provided prefix.
   - The input dataset is located in `/data/students/bigdata-01QYD/Lab2/`.

2. **Statistics**:
   - Print the number of selected lines.
   - Print the maximum frequency (`maxfreq`) among the selected lines.

3. **Arguments**:
   - **Input Folder**: Directory containing the dataset.
   - **Output Folder**: Directory to store the filtered results.
   - **Prefix**: Prefix to filter the words.

### Task 2: Advanced Filtering

1. **Further Filtering**:
   - Among the lines selected in Task 1, keep only those with a frequency greater than 80% of `maxfreq`.
   - This means keeping lines with a frequency (`freq`) greater than `0.8 * maxfreq`.

2. **Output**:
   - Count the number of lines remaining after the second filter.
   - Save the words (without frequencies) in the output folder (one word per line).

## Testing Your Application

1. **Run on Cluster**:
   - Use the `spark-submit` command to run your application on the cluster. For example:

   ```sh
   spark-submit --class it.polito.bigdata.spark.example.SparkDriver --deploy-mode client --master yarn SparkProject.jar /data/students/bigdata-01QYD/Lab2/ HDFSOutputFolder "ho"
   ```

2. **Analyze Output**:
   - Check the contents of the output HDFS folder to verify the results.

## Accessing Log Files

To retrieve logs for your Spark application:

1. **Get Application ID**:
   - You can find the application ID using:

     ```sh
     yarn application -list -appStates ALL | grep 'sXXXXX'
     ```

   - Or print the application ID in your application code:

     ```java
     System.out.println("ApplicationId: " + JavaSparkContext.toSparkContext(sc).applicationId());
     ```

2. **Retrieve Logs**:
   - Use the following command to get logs:

     ```sh
     yarn logs -applicationId application_id
     ```

   - Replace `application_id` with your actual application ID.

## Shutting Down JupyterHub Container

Once you have completed all tasks, shut down the JupyterHub container to allow others to connect:

1. Go to `File -> Hub Control Panel`.
2. Click on the “Stop My Server” button and wait until it disappears.