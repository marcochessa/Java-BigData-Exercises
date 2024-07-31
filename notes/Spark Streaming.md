# Spark Streaming

Spark Streaming is a framework for processing large-scale data streams in near-real-time. It processes live data streams and produces results that are nearly instantaneous. The framework can handle large volumes of data distributed across hundreds of nodes.

- [Discretized Stream Processing](#discretized-stream-processing)
- [Transformations](#transformations)
- [Actions](#actions)
- [Start and Run](#start-and-run)
- [Example Word Count](#example-word-count)
- [Window Operations](#window-operations)
- [Checkpoints](#checkpoints)
- [Stateful Transformations](#stateful-transformations)
- [Advanced Transformations](#advanced-transformations)

## Discretized Stream Processing

Spark Streaming processes data by dividing it into small, manageable batches, creating a sequence of RDDs (Resilient Distributed Datasets) from the live data stream. Each RDD is processed as a batch, and the results are returned in batches.

1. **Batching:** The live data stream is split into batches based on a specified interval (e.g., every 10 seconds).
2. **RDD Operations:** Each batch is treated as an RDD and processed using RDD operations.
3. **Batch Processing:** The processed results from RDD operations are returned as new batches.

- **DStream:** Represents a sequence of RDDs. Each RDD in a DStream represents a batch of data.
- **PairDStream:** Represents a sequence of PairRDDs, where each RDD consists of key-value pairs.
- **Transformations:** Operations that transform one DStream into another.
- **Actions:** Operations that output results to external systems.

### Basic Steps to Set Up Streaming

1. **Define SparkStreamingContext:** Create a `JavaStreamingContext` with a batch interval.
2. **Specify Input Stream:** Define the source of the streaming data, such as a socket or a file directory.
3. **Specify Operations:** Define the transformations and actions to be performed on the data.
4. **Start the Context:** Begin processing the data stream.
5. **Run Until Termination:** Keep the application running until explicitly terminated or until a timeout occurs.

```java
JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(10));
// Content emitted by a TCP socket on localhost:9999
JavaReceiverInputDStream<String> lines = jssc.socketTextStream("localhost", 9999);
```

For data in a HDFS folder, use:

```java
JavaDStream<String> lines = jssc.textFileStream(inputFolder);
```

## Transformations

Transformations are operations that create a new DStream from an existing one.

- **map(func):** Applies a function to each element of the source DStream and returns a new DStream.
- **flatMap(func):** Similar to `map`, but each input item can be mapped to multiple output items.
- **filter(func):** Filters out elements based on a predicate function.
- **reduce(func):** Aggregates elements of the DStream into a single result.
- **reduceByKey(func):** Aggregates values of each key in a PairDStream.
- **countByValue():** Counts the occurrences of each element in the DStream.
- **count():** Counts the number of elements in each batch.
- **union(otherStream):** Combines the elements of two DStreams into a new DStream.
- **join(otherStream):** Joins two PairDStreams based on their keys.
- **cogroup(otherStream):** Groups the elements of two PairDStreams by their keys.

## Actions

Actions are operations that produce results or output data to external systems.

- **print():** Prints the first 10 elements of each batch to the console.
- **saveAsTextFile(prefix, [suffix]):** Saves the content of the DStream as text files with the specified prefix and optional suffix.

## Start and Run

- **`start()` Method:** Starts the streaming computation.
- **`awaitTerminationOrTimeout(long milliseconds)` Method:** Keeps the application running until either terminated or until the specified timeout.

## Example Word Count

Here's a simple example of a word count application:

```java
SparkConf conf = new SparkConf().setAppName("WordCount");
JavaStreamingContext jssc = new JavaStreamingContext(conf, Durations.seconds(10));

JavaReceiverInputDStream<String> lines = jssc.socketTextStream("localhost", 9999);

JavaDStream<String> words = lines.flatMap(line -> Arrays.asList(line.split("\\s+")).iterator());

JavaPairDStream<String, Integer> wordPairs = words.mapToPair(word -> new Tuple2<>(word.toLowerCase(), 1));

JavaPairDStream<String, Integer> wordCounts = wordPairs.reduceByKey((i1, i2) -> i1 + i2);

wordCounts.print();
wordCounts.saveAsTextFiles(outputPathPrefix, "");

jssc.start();
jssc.awaitTerminationOrTimeout(120000);
jssc.close();
```

## Window Operations

Window operations allow you to apply transformations over a sliding window of data. Windows can overlap and are defined by a window length and a sliding interval.

- **window(windowLength, slideInterval):** Applies a transformation over a window of data with a specified length and sliding interval.
- **countByWindow(windowLength, slideInterval):** Counts the number of elements within each window.
- **reduceByKeyAndWindow(func, windowLength, slideInterval):** Reduces values by key over a sliding window of data.

## Checkpoints

Checkpoints are used to store the state of the computation to enable recovery in case of failure. They are especially useful for maintaining state in streaming applications.

```java
jssc.checkpoint("checkpointFolder"); // Specify the folder for checkpointing
```

## Stateful Transformations

Stateful transformations allow you to maintain state across batches of data.

- **UpdateStateByKey:** Maintains and updates the state of keys across batches. You define a state update function, and Spark applies it to all existing keys.

Example of updating state in a word count application:

```java
JavaPairDStream<String, Integer> totalWordCounts = wordPairs.updateStateByKey((newValues, state) -> {
   Integer newSum = state.orElse(0);
   for(Integer value : newValues) {
      newSum += value;
   }
   return Optional.of(newSum);
});
```

## Advanced Transformations

Advanced transformations provide more control over the data processing:

- **transform(func):** Applies an RDD-to-RDD function to each RDD in the source DStream, allowing custom processing of RDDs.
- **transformToPair(func):** Applies a PairRDD-to-PairRDD function to each PairRDD in the source PairDStream, useful for advanced PairRDD operations.