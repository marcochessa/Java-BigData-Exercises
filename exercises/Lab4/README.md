# Lab 4

In this lab, we will continue working with the Amazon food dataset and focus on the ratings given by users to products. The goal is to normalize the ratings according to the user's tendency to give higher ratings.

### Example

Consider the following rating matrix:

```
    A1  A2  A3  A4  A5
B1  5   4   5   1   5
B2  3   3   5   4   3
B3  5   5   4   4   4
B4  5   4   4   3   5
B5  5   3   4   2   3
```

To remove user bias, normalize each column (user) by subtracting the mean rating given by that user. For example:

- **User A2** has given 5 stars to products B1, B3, and B5, and 3 stars to B2. 
  - Mean rating for A2 = (5 + 5 + 3) / 3 = 4.33
  - Normalized ratings: B1: 5 - 4.33 = 0.67, B2: 3 - 4.33 = -1.33, etc.

Compute the average of these normalized ratings for each product to obtain a "normalized average rating."

### Task

Write a Hadoop application to compute the normalized average ratings of products based on the ratings in the following HDFS file:

- **Input File:** `/data/students/bigdata-01QYD/Lab4/Reviews.csv`
- **File Format:** Id,ProductId,UserId,ProfileName,HelpfulnessNumerator,HelpfulnessDenominator,Score,Time,Summary,Text
- **Fields of Interest:** ProductId, UserId, and Score

**Output:** Each line should contain the product ID and its normalized average rating.

### Sample Data for Testing

For initial testing, you can use the small sample dataset:

- **Sample File:** ReviewsSample.csv (available on the course web page)

### Hints

1. **Header Filtering:** The input file includes a header that must be filtered out.
2. **Sparse Matrix:** The matrix is sparse, meaning many values are null/unknown. You can assume the number of ratings per user is small and can be stored in a local Java variable to avoid excessive network communication.

### ⚠️ Shut Down JupyterHub Container ⚠️

As soon as you complete all tasks in the JupyterHub environment, remember to shut down the container to allow others to connect and perform their lab activities.

**Steps to Shut Down:**
1. Go into `File -> Hub Control Panel` menu.
2. A new browser tab opens with the “Stop My Server” button. Click on it and wait until it disappears.