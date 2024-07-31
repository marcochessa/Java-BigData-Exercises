# Lab 9: Classification of Amazon Fine-Foods Reviews with Spark MLlib

In this lab, you will build and evaluate classification models using Spark MLlib. Your goal is to classify reviews as "useful" or "useless" based on their helpfulness scores.

## Data and Setup

### Input Dataset

- **File Path**: `/data/students/bigdata-01QYD/Lab9/ReviewsNewfile.csv`
- **Format**:
  ```
  Id\tProductId\tUserId\tProfileName\tHelpfulnessNumerator\tHelpfulnessDenominator\tScore\tTime\tSummary\tText
  ```
  - **Text**: The review text.
  - **HelpfulnessNumerator**: Number of users who found the review helpful.
  - **HelpfulnessDenominator**: Total number of users who voted on the review.

### Tasks

## Task 1: Preprocessing

1. **Read and Clean Data**:
   - Load the CSV file into a DataFrame.
   - Filter out reviews where `HelpfulnessDenominator` is 0 (i.e., reviews that have not been rated).
   - Remove the header from the dataset.

2. **Create Features and Labels**:
   - Compute the helpfulness index as `HelpfulnessNumerator / HelpfulnessDenominator`.
   - Label the review as:
     - **1.0** (useful) if the helpfulness index is greater than 0.9.
     - **0.0** (useless) otherwise.
   - Use the length of the review text as the feature. Create a `LabeledPoint` RDD and convert it to a DataFrame with columns `label` and `features`.

3. **Code Example**:

```java
import org.apache.spark.sql.Dataset;
import org.apache.spark.sql.Row;
import org.apache.spark.sql.SparkSession;
import org.apache.spark.ml.feature.VectorAssembler;
import org.apache.spark.ml.linalg.Vectors;
import org.apache.spark.ml.linalg.Vector;
import org.apache.spark.ml.classification.DecisionTreeClassifier;
import org.apache.spark.ml.classification.LogisticRegression;
import org.apache.spark.ml.Pipeline;
import org.apache.spark.ml.evaluation.MulticlassClassificationEvaluator;
import org.apache.spark.sql.functions;

public class ReviewClassification {
    public static void main(String[] args) {
        SparkSession spark = SparkSession.builder()
            .appName("ReviewClassification")
            .getOrCreate();

        // Read data
        Dataset<Row> reviewsDF = spark.read().format("csv")
            .option("header", "true")
            .option("inferSchema", "true")
            .option("delimiter", "\t")
            .load("/data/students/bigdata-01QYD/Lab9/ReviewsNewfile.csv");

        // Filter and preprocess data
        Dataset<Row> filteredDF = reviewsDF
            .filter(functions.col("HelpfulnessDenominator").gt(0))
            .withColumn("helpfulnessIndex", functions.col("HelpfulnessNumerator")
                .divide(functions.col("HelpfulnessDenominator")))
            .withColumn("label", functions.when(functions.col("helpfulnessIndex").gt(0.9), 1.0).otherwise(0.0))
            .withColumn("textLength", functions.length(functions.col("Text")))
            .select(functions.col("label"), functions.col("textLength").as("features"));

        // Prepare features
        VectorAssembler assembler = new VectorAssembler()
            .setInputCols(new String[] {"textLength"})
            .setOutputCol("features");

        Dataset<Row> assembledDF = assembler.transform(filteredDF);
        
        // Split data
        Dataset<Row>[] splits = assembledDF.randomSplit(new double[] {0.8, 0.2});
        Dataset<Row> trainingData = splits[0];
        Dataset<Row> testData = splits[1];

        // Create and train the model
        DecisionTreeClassifier dt = new DecisionTreeClassifier()
            .setLabelCol("label")
            .setFeaturesCol("features");

        LogisticRegression lr = new LogisticRegression()
            .setLabelCol("label")
            .setFeaturesCol("features");

        // Decision Tree Pipeline
        Pipeline dtPipeline = new Pipeline().setStages(new org.apache.spark.ml.PipelineStage[] {dt});
        PipelineModel dtModel = dtPipeline.fit(trainingData);
        Dataset<Row> dtPredictions = dtModel.transform(testData);
        
        // Logistic Regression Pipeline
        Pipeline lrPipeline = new Pipeline().setStages(new org.apache.spark.ml.PipelineStage[] {lr});
        PipelineModel lrModel = lrPipeline.fit(trainingData);
        Dataset<Row> lrPredictions = lrModel.transform(testData);

        // Evaluate
        MulticlassClassificationEvaluator evaluator = new MulticlassClassificationEvaluator()
            .setLabelCol("label")
            .setPredictionCol("prediction")
            .setMetricName("accuracy");

        double dtAccuracy = evaluator.evaluate(dtPredictions);
        double lrAccuracy = evaluator.evaluate(lrPredictions);

        System.out.println("Decision Tree Accuracy = " + dtAccuracy);
        System.out.println("Logistic Regression Accuracy = " + lrAccuracy);

        spark.stop();
    }
}
```

## Task 2: Creating a Pipeline

1. **Pipeline Creation**:
   - Construct a Pipeline that includes:
     - **Feature Extraction**: Convert text length and other features into a feature vector.
     - **Classification Algorithm**: Train using Decision Tree or Logistic Regression.
     - **Evaluation**: Split the data into training and testing sets, then evaluate the model's precision.

2. **Create and Compare Models**:
   - Implement pipelines for both Decision Tree and Logistic Regression.
   - Compare their performance based on precision and accuracy.

## Task 3: Adding Features

1. **Feature Engineering**:
   - Add additional features such as:
     - Review score.
     - Length of summary.
     - Number of words in the review text.
     - Presence of certain keywords (e.g., "good", "bad").
   - Re-run the pipeline with these new features.

2. **Evaluate the Enhanced Model**:
   - Compare the performance of the model with additional features to the original models.

## Task 4: Using Review Content

1. **Textual Data Classification**:
   - Use the review text to create features.
   - Apply techniques such as TF-IDF or word embeddings to convert text into feature vectors.
   - Train a model using these text-based features.

2. **Code Reference**:
   - Refer to the "Textual data classification example code" provided for guidance on handling textual features.

3. **Evaluate the Text-Based Model**:
   - Assess whether the text-based model performs better than models using only numerical features.

## Shutting Down JupyterHub Container

After completing all tasks:

1. Go to `File -> Hub Control Panel`.
2. Click the “Stop My Server” button and wait until it disappears.