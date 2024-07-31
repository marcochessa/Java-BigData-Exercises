# Spark RDD
### SparkContext
- **Usage:** Initializes the Spark application.
- **Java Example:**
  ```java
  SparkConf conf = new SparkConf().setAppName("Application name");
  JavaSparkContext sc = new JavaSparkContext(conf);
  ```

### RDDs Creation
- **From File:** Reads data from a file into an RDD.
  ```java
  JavaRDD<String> lines = sc.textFile("inputFile", numPartitions);
  ```
- **From Local Collection:** Creates an RDD from a local collection.
  ```java
  JavaRDD<T> distList = sc.parallelize(inputList, numPartitions);
  ```

### Save RDDs
- **To HDFS:** Save RDD data to a file in HDFS.
  ```java
  lines.saveAsTextFile("outputPath");
  ```
- **To Local Variable:** Collect data into a local list (caution with large datasets).
  ```java
  List<String> contentOfLines = lines.collect();
  ```

### RDDs Operations
- **Transformations:** Create new RDDs from existing ones, performed lazily.
- **Actions:** Trigger computation and return results.

### Basic RDD Transformations
- **Filter:** Returns a new RDD with elements that satisfy the predicate.
  ```java
  JavaRDD<String> errorsRDD = inputRDD.filter(element -> element.contains("error"));
  ```
- **Map:** Applies a function to each element and returns a new RDD.
  ```java
  JavaRDD<Integer> lengthsRDD = inputRDD.map(element -> element.length());
  ```
- **FlatMap:** Similar to map but each element can produce multiple output elements.
  ```java
  JavaRDD<String> listOfWordsRDD = inputRDD.flatMap(x -> Arrays.asList(x.split(" ")).iterator());
  ```
- **Distinct:** Removes duplicate elements.
  ```java
  JavaRDD<String> distinctNamesRDD = inputRDD.distinct();
  ```
- **Sample:** Returns a random sample of elements.
  ```java
  JavaRDD<String> sampleRDD = inputRDD.sample(false, 0.2);
  ```
- **Union:** Combines elements of two RDDs (duplicates retained).
  ```java
  JavaRDD<Integer> unionRDD = inputRDD1.union(inputRDD2);
  ```
- **Intersection:** Elements common to both RDDs.
  ```java
  JavaRDD<Integer> intersectionRDD = inputRDD1.intersection(inputRDD2);
  ```
- **Subtract:** Elements in the first RDD but not in the second.
  ```java
  JavaRDD<Integer> subtractRDD = inputRDD1.subtract(inputRDD2);
  ```
- **Cartesian:** Returns the Cartesian product of two RDDs.
  ```java
  JavaPairRDD<Integer, Integer> cartesianRDD = inputRDD1.cartesian(inputRDD2);
  ```

### Basic RDD Actions
- **Collect:** Retrieves all elements to the driver (use with caution).
  ```java
  List<Integer> values = inputRDD.collect();
  ```
- **Count:** Counts the number of elements.
  ```java
  long count = inputRDD.count();
  ```
- **CountByValue:** Counts occurrences of each element.
  ```java
  Map<String, Long> counts = namesRDD.countByValue();
  ```
- **Take:** Retrieves the first `n` elements.
  ```java
  List<Integer> firstTwo = inputRDD.take(2);
  ```
- **Top:** Retrieves the top `n` largest elements.
  ```java
  List<Integer> topTwo = inputRDD.top(2);
  ```
- **TakeOrdered:** Retrieves the top `n` smallest elements.
- **TakeSample:** Retrieves `n` random elements.
  ```java
  List<Integer> randomValues = inputRDD.takeSample(false, 2);
  ```
- **Reduce:** Aggregates elements using a specified function.
  ```java
  Integer sum = inputRDD.reduce((a, b) -> a + b);
  ```
- **Fold:** Aggregates elements with a zero value.
- **Aggregate:** Aggregates elements using an initial value and two functions (combining and merging).

### Pair RDDs
- **Creation:** Create key-value pairs from RDDs.
  ```java
  JavaPairRDD<String, Integer> nameOneRDD = namesRDD.mapToPair(name -> new Tuple2<>(name, 1));
  ```
- **Operations:** Includes transformations like `reduceByKey`, `join`, and `cogroup`.

### Transformations on Pair RDDs
- **ReduceByKey:** Reduces values by key.
  ```java
  JavaPairRDD<String, Integer> reducedRDD = nameAgeRDD.reduceByKey((a, b) -> Math.min(a, b));
  ```
- **GroupByKey:** Groups values by key (less efficient than `reduceByKey`).
  ```java
  JavaPairRDD<String, Iterable<Integer>> groupedRDD = nameAgeRDD.groupByKey();
  ```
- **MapValues:** Applies a function to each value.
  ```java
  JavaPairRDD<String, Integer> incrementedRDD = nameAgeRDD.mapValues(age -> age + 1);
  ```

### Transformations on Pairs of Pair RDDs
- **SubtractByKey:** Removes pairs with keys that appear in another RDD.
  ```java
  JavaPairRDD<String, Integer> filteredRDD = profilesPairRDD.subtractByKey(bannedPairRDD);
  ```
- **Join:** Joins two RDDs by key.
  ```java
  JavaPairRDD<Integer, Tuple2<String, String>> joinedRDD = questionsPairRDD.join(answersPairRDD);
  ```
- **CoGroup:** Groups by key, producing a list of values from each RDD.
  ```java
  JavaPairRDD<Integer, Tuple2<Iterable<String>, Iterable<String>>> cogroupedRDD = moviesPairRDD.cogroup(directorsPairRDD);
  ```

### Actions on Pair RDDs
- **CountByKey:** Counts the number of elements for each key.
  ```java
  Map<String, Long> movieRatingsCount = movieRatingRDD.countByKey();
  ```
- **CollectAsMap:** Collects pairs into a map.
  ```java
  Map<String, String> collectedMap = usersRDD.collectAsMap();
  ```
- **Lookup:** Retrieves values for a specific key.
  ```java
  List<Integer> movieRatings = movieRatingRDD.lookup("Forrest Gump");
  ```

### Double RDDs
- **MapToDouble:** Maps elements to `double` values.
  ```java
  JavaDoubleRDD lengthsDoubleRDD = surnamesRDD.mapToDouble(surname -> (double) surname.length());
  ```
- **FlatMapToDouble:** FlatMaps elements to `double` values.
  ```java
  JavaDoubleRDD wordLengthsDoubleRDD = sentencesRDD.flatMapToDouble(sentence -> {
      List<Double> lengths = new ArrayList<>();
      for (String word : sentence.split(" ")) {
          lengths.add((double) word.length());
      }
      return lengths.iterator();
  });
  ```

### Persistence and Cache
- **Persist:** Stores RDD in memory/disk.
  ```java
  JavaRDD<String> persistedRDD = sc.textFile("file.txt").persist(StorageLevel.MEMORY_ONLY());
  ```
- **Cache:** Shortcut for `persist(StorageLevel.MEMORY_ONLY())`.
  ```java
  JavaRDD<String> cachedRDD = sc.textFile("file.txt").cache();
  ```

### Accumulators
- **Usage:** Track aggregated values like counters or sums.
  ```java
  LongAccumulator accumulator = sc.sc().longAccumulator();
  accumulator.add(1);
  System.out.println(accumulator.value());
  ```

### Broadcast Variables
- **Usage:** Share read-only variables efficiently across nodes.
  ```java
  HashMap<String, Integer> dictionary = new HashMap<>();
  Broadcast<HashMap<String, Integer>> dictionaryBroadcast = sc.broadcast(dictionary);
  ```

Feel free to add more examples or detailed explanations based on your specific needs or the context in which these RDD operations will be used.