# Spark MLlib

Spark MLlib provides a range of machine learning and data mining algorithms for tasks such as preprocessing, classification, clustering, and association rule mining. It includes both the `mllib` and `ml` packages.

- [Data Types](#data-types)
- [Main Concepts](#main-concepts)
- [Classification Algorithms](#classification-algorithms)
- [Categorical Class Labels](#categorical-class-labels)
- [Sparse Labeled Data](#sparse-labeled-data)
- [Textual Data Classification](#textual-data-classification)
- [Parameter Tuning](#parameter-tuning)
- [Clustering Algorithms](#clustering-algorithms)
- [Itemset and Association Rule Mining](#itemset-and-association-rule-mining)
- [Regression Algorithms](#regression-algorithms)

## Data Types

MLlib utilizes various data types including local vectors, labeled points, local matrices, and distributed matrices. DataFrames for ML contain objects based on these fundamental data types.

- **Local Vectors:**
   These are used to represent vectors of double values, either in dense format `[1.0, 0.0, 3.0]` or sparse format `(3, [0, 2], [1.0, 3.0])`, where the first parameter is the length, the second is the indices of non-zero values, and the third is the actual values.

   ```java
   Vector sparseVector = Vectors.sparse(3, new int[] {0, 2}, new double[] {1.0, 3.0});
   Vector denseVector = Vectors.dense(1.0, 0.0, 3.0);
   ```

- **Labeled Points:**
   These are local vectors of doubles paired with a label, where the label is also a double. They are used in supervised learning algorithms to represent individual data points.

   ```java
   LabeledPoint point1 = new LabeledPoint(1.0, Vectors.dense(1.0, 0.0, 3.0));
   LabeledPoint point2 = new LabeledPoint(0.0, Vectors.dense(2.0, 5.0, 3.0));
   ```

## Main Concepts

In Spark MLlib, DataFrames are used as input data for machine learning algorithms. The inputs must be represented as "tables" before applying MLlib algorithms.

- **Transformer:** An ML algorithm that transforms one DataFrame into another.
- **Estimator:** An ML algorithm that takes a DataFrame and produces a Transformer (model).
- **Pipeline:** A sequence of multiple Transformers and Estimators to define an ML or data mining workflow.
- **Parameter:** Common APIs are used to specify parameters for Transformers and Estimators.

- **Pipeline Workflow:**
   1. Instantiate the Transformers and Estimators.
   2. Create a Pipeline object and specify the sequence.
   3. Fit the Pipeline to training data to create a model.
   4. Apply the model to new data.

## Classification Algorithms

Classification involves two main phases: model training and prediction. Only numerical attributes are used, and algorithms require a DataFrame with at least two columns: label and features.

- **Logistic Regression with Structured Data:**

   ```java
   SparkSession spark = SparkSession.builder().appName("Logistic Regression").getOrCreate();
   JavaSparkContext context = new JavaSparkContext(spark.sparkContext());

   // Training
   JavaRDD<String> trainingData = context.textFile("inputFileTraining");
   JavaRDD<LabeledPoint> trainingRDD = trainingData.map(record -> {
       String[] fields = record.split(",");
       double label = Double.parseDouble(fields[0]);
       double[] featuresArray = new double[] {
           Double.parseDouble(fields[1]),
           Double.parseDouble(fields[2]),
           Double.parseDouble(fields[3])
       };
       Vector features = Vectors.dense(featuresArray);
       return new LabeledPoint(label, features);
   });
   Dataset<Row> trainingDF = spark.createDataFrame(trainingRDD, LabeledPoint.class).cache();
   
   LogisticRegression lr = new LogisticRegression();
   lr.setMaxIter(10);
   lr.setRegParam(0.01);
   Pipeline pipeline = new Pipeline().setStages(new PipelineStage[] {lr});
   PipelineModel model = pipeline.fit(trainingDF);

   // Prediction
   JavaRDD<String> testData = context.textFile("inputFileTest");
   JavaRDD<LabeledPoint> testRDD = testData.map(record -> {
       String[] fields = record.split(",");
       double[] featuresArray = new double[] {
           Double.parseDouble(fields[1]),
           Double.parseDouble(fields[2]),
           Double.parseDouble(fields[3])
       };
       Vector features = Vectors.dense(featuresArray);
       return new LabeledPoint(-1.0, features); // Unlabeled data
   });
   Dataset<Row> testDF = spark.createDataFrame(testRDD, LabeledPoint.class);
   Dataset<Row> predictions = model.transform(testDF);
   Dataset<Row> predictionsDF = predictions.select("features", "prediction");

   // Save results
   predictionsDF.javaRDD().saveAsTextFile("outputPath");
   context.close();
   ```

- **Decision Trees with Structured Data:**
   Similar to the above example but using a `DecisionTreeClassifier`.

   ```java
   DecisionTreeClassifier dt = new DecisionTreeClassifier();
   dt.setImpurity("gini");
   Pipeline pipeline = new Pipeline().setStages(new PipelineStage[] {dt});
   ```

## Categorical Class Labels

Often, class labels are categorical values (strings). The `StringIndexer` and `IndexToString` Estimators can convert categorical labels to numerical values and vice versa.

```java
public class MyLabeledPoint implements Serializable {
   private String categoricalLabel;
   private Vector features;

   public MyLabeledPoint(String categoricalLabel, Vector features) {
      this.categoricalLabel = categoricalLabel;
      this.features = features;
   }

   public String getCategoricalLabel() {
      return categoricalLabel;
   }

   public Vector getFeatures() {
      return features;
   }

   public void setFeatures(Vector features) {
      this.features = features;
   }
}
```

Define a `StringIndexer` to convert categorical labels and an `IndexToString` to reverse the conversion.

```java
StringIndexerModel labelIndexer = new StringIndexer()
   .setInputCol("categoricalLabel")
   .setOutputCol("label")
   .fit(trainingData);
IndexToString labelConverter = new IndexToString()
   .setInputCol("prediction")
   .setOutputCol("predictedLabel")
   .setLabels(labelIndexer.labels());
Pipeline pipeline = new Pipeline().setStages(new PipelineStage[] {labelIndexer, decisionTree, labelConverter});
```

## Sparse Labeled Data

MLlib can read training data in LIBSVM format, which uses sparse feature vectors.

```java
Dataset<Row> data = spark.read().format("libsvm").load("sample_libsvm_data.txt");
```

## Textual Data Classification

For textual data, convert text to feature vectors for classification:

1. Convert text data into attributes to form a DataFrame.
2. Remove common words (stop words).
3. Use TF-IDF to weigh words based on their frequency.

```java
Tokenizer tokenizer = new Tokenizer().setInputCol("text").setOutputCol("words");
StopWordsRemover remover = new StopWordsRemover().setInputCol("words").setOutputCol("filteredWords");
HashingTF hashingTF = new HashingTF().setNumFeatures(1000).setInputCol("filteredWords").setOutputCol("rawFeatures");
IDF idf = new IDF().setInputCol("rawFeatures").setOutputCol("features");
Pipeline pipeline = new Pipeline().setStages(new PipelineStage[] {tokenizer, remover, hashingTF, idf, logisticRegression});
```

## Parameter Tuning

Parameter tuning in Spark MLlib can be approached using brute-force methods or cross-validation. Spark supports grid-based parameter search for fine-tuning models.

## Clustering Algorithms

Clustering algorithms in Spark work exclusively with numerical data. The input DataFrame must include a column named "features", and clustering is performed based solely on the content of this column.

## Itemset and Association Rule Mining

Spark MLlib provides algorithms for itemset mining and association rule mining, including FP-growth for frequent itemset mining and various rule mining techniques.

## Regression Algorithms

Spark MLlib also includes algorithms for regression tasks, enabling predictive modeling for continuous values.