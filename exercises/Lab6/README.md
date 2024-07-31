# Lab 6: Analyzing Amazon Food Dataset with Apache Spark

In this lab, we will use Apache Spark to analyze the Amazon food dataset. The goal is to find pairs of products frequently reviewed together and compute relevant statistics.

## Tasks

### Task 1: Pair Frequency Analysis

1. **Data Preparation**:
   - **Input File**: `/data/students/bigdata-01QYD/Lab4/Reviews.csv`
   - **Schema**:
     ```
     Id,ProductId,UserId,ProfileName,HelpfulnessNumerator,HelpfulnessDenominator,Score,Time,Summary,Text
     ```
   - You can use `ReviewsSample.csv` for initial testing.

2. **Steps to Implement**:

   **a. Transpose the Dataset**:
   - Create a PairRDD where each pair consists of:
     ```
     (user_id, list of product_ids reviewed by user_id)
     ```
   - Ensure that each product_id appears only once per user.

   **b. Count Product Pairs**:
   - From the transposed PairRDD, create pairs of products reviewed together.
   - Count how many times each pair is reviewed together.

   **c. Output**:
   - Write the pairs of products that appear more than once and their frequencies to the output folder.
   - The pairs should be sorted by decreasing frequency.

### Task 2: Bonus Task

1. **Find Top 10 Most Frequent Product Pairs**:
   - Extend your application to print the top 10 most frequent product pairs and their frequencies.
   - Use Spark actions like `top(int n, java.util.Comparator<T> comp)` or `takeOrdered(int n, java.util.Comparator<T> comp)` to retrieve the top 10 pairs.

2. **Output**:
   - Print the top 10 pairs and their frequencies to the standard output.

## Accessing Log Files

If you run your application with `--deploy-mode cluster`, the standard output of the driver is stored in the log files. To access these logs:

1. **Retrieve Application ID**:
   - Run your application with `--deploy-mode client` and add the following line to your code to print the application ID:
     ```java
     System.out.println("ApplicationId: " + JavaSparkContext.toSparkContext(sc).applicationId());
     ```
   - Alternatively, use:
     ```sh
     yarn application -list -appStates ALL | grep 'sXXXXX'
     ```
     Replace `sXXXXX` with your username.

2. **Get Log Files**:
   - Use the following command to retrieve logs:
     ```sh
     yarn logs -applicationId application_id
     ```
     Replace `application_id` with the actual application ID.

## Shutting Down JupyterHub Container

After completing your tasks, shut down the JupyterHub container:

1. Go to `File -> Hub Control Panel`.
2. Click on the “Stop My Server” button and wait for it to disappear.
