# Lab 3

In this lab, we analyze a data set containing information about the items reviewed by some Amazon users. Specifically, we will investigate the connections between reviewed items. The data set contains one line per reviewer, where the first field is the reviewer ID, followed by a list of all the products reviewed by them. 

### Sample Data
Here is a sample of the first 11 lines of such a dataset:

```
A09539661HB8JHRFVRDC,B002R8UANK,B002R8J7YS,B002R8SLUY
A1008ULQSWI006,B0017OAQIY
A100EBHBG1GF5,B0013T5YO4
A1017Y0SGBINVS,B0009F3SAK
A101F8M8DPFOM9,B005HY2BRO,B000H7MFVI
A102H88HCCJJAB,B0007A8XV6
A102ME7M2YW2P5,B000FKGT8W
A102QP2OSXRVH,B001EQ5SGU,B000EH0RTS
A102TGNH1D915Z,B000RHXKC6,B0002DHNXC,B0002DHNXC,B000XJK7UG,B00008DFK5
A1051WAJL0HJWH,B000W5U5H6
A1052V04GOA7RV,B002GJ9JY6,B001E5E3JY,B008ZRKZSM,B002GJ9JWS
```

### Exercise 1: "People also like..."

In this exercise, you will build a basic version of a recommendation system to find the top 100 pairs of products most often reviewed together.

**Task Details:**
- **Objective:** Find the top 100 pairs of products reviewed together.
- **Data File:** `/data/students/bigdata-01QYD/Lab3/AmazonTransposedDataset_Sample.txt`
- **Definition:** Two products are considered reviewed together if they appear in the same line of the input file.

**Implementation Steps:**
1. **Template Project:** Use the template project provided (`Lab3_Skeleton_with_libraries.zip`) which includes the skeleton of the Driver, Mapper, and Reducer classes. Fill out the missing parts as needed.
2. **Classes Provided:**
   - **WordCountWritable:** A class to store a pair (word, count), where `word` is a String and `count` is an Integer. Implements `Comparable` and `Writable` interfaces.
   - **TopKVector<T extends Comparable<T>>:** A class to manage the top-k objects of a type that implements `Comparable`. 

   **Usage Example:**
   ```java
   // Create a TopKVector to store top-3 WordCountWritable objects
   TopKVector<WordCountWritable> top3 = new TopKVector<WordCountWritable>(3);

   // Insert objects
   top3.updateWithNewElement(new WordCountWritable("p1,p2", 4));
   top3.updateWithNewElement(new WordCountWritable("p1,p3", 40));
   top3.updateWithNewElement(new WordCountWritable("p2,p4", 3));
   top3.updateWithNewElement(new WordCountWritable("p5,p6", 6));
   top3.updateWithNewElement(new WordCountWritable("p15,p16", 1));

   // Retrieve and print top-k objects
   Vector<WordCountWritable> top3Objects = top3.getLocalTopK();
   for (WordCountWritable value : top3Objects) {
       System.out.println(value.getWord() + " " + value.getCount());
   }
   ```

   **Output Example:**
   ```
   p1,p3 40
   p5,p6 6
   p1,p2 4
   ```

### ⚠️ Shut Down JupyterHub Container ⚠️

As soon as you complete all the tasks and activities in the JupyterHub environment, please remember to shut down the container to allow others to connect and perform their lab activities.

**Steps to Shut Down:**
1. Go into `File -> Hub Control Panel` menu.
2. A new browser tab opens with the “Stop My Server” button. Click on it and wait until it disappears.