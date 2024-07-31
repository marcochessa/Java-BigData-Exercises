# Lab 1

This introductory lab is composed of two main tasks. Your first objective is to run your first MapReduce application on the BigData@Polito cluster. For this goal, you must learn how to import a predefined project, compile the source code, produce a JAR file, copy your application on a remote machine (the gateway), and submit your application on the BigData@Polito cluster. The second objective is to write your first MapReduce application.

## 1. Compiling Using the Eclipse IDE

In this task, we will “compile” the source code of a simple Hadoop application and create a JAR file with the class files of the project. The shared Eclipse project contains the basic libraries needed to develop the application and create the class and JAR files. You cannot use this project to run locally on the PC of the Lab or on your own PC (other libraries are needed to run this application locally).

The imported project is the MapReduce implementation of the word count application.

### Steps:

1. Download from the course web page the zip file `Lab1_BigData_with_libraries.zip`, which contains the MapReduce-based implementation of the word count application and the example data folder `example_data` (direct link: [Lab1_BigData_with_libraries.zip](http://dbdmg.polito.it/dbdmg_web/wp-content/uploads/2021/10/Lab1_BigData_with_libraries.zip)).

2. Decompress the zip file inside a local folder on the PC of the Lab or on your PC. Import the project into Eclipse and create the JAR file as follows:
   1. Open Eclipse.
   2. Import the project: `File -> Import... | General -> Existing project... | Choose the folder where you extracted the content of the zip file`.
   3. Examine the source files and the structure of the project. Locate the mapper and the reducer.
   4. Since we cannot use Maven on the PC of the Lab, build the JAR manually using the `File -> Export` command. Avoid including all libraries in your JAR to prevent creating a fat JAR that is heavy to transfer. We need these libraries locally to compile, but they are already present in the classpath of the cluster.
   5. Specify the name of the output JAR file (e.g., set the JAR file textbox to `Exercise1-1.0.0.jar`).

## 2. Upload Your Application on BigData@Polito

The objective of this task is to upload your application on the BigData@Polito cluster.

### Steps:

1. Connect to [https://jupyter.polito.it](https://jupyter.polito.it).
2. Upload the JAR file containing your application to the local file system of the gateway.

## 3. Manage HDFS Through the HUE Web Interface

In this task, you will learn how to perform basic management of the HDFS file system using a web interface called HUE.

### Steps:

1. Go to [https://hue.polito.it/](https://hue.polito.it/) and log in with your usual BigData@Polito credentials.
2. Go to the “Browsers/Files” tab. You should find your HDFS home, as shown below. Note that this is not the same file system as in task 2 (i.e., it is not the local file system of the gateway), so you will probably find an empty folder now.
   
   Your HDFS home is not located on the gateway but is stored in the Hadoop cluster. If the difference between the two “homes” is not clear, spend some time to clarify your ideas.

3. Create the folder `example_data` on HDFS.
4. Upload the sample files from the local folder `example_data` available on your PC to the folder `example_data` on HDFS.
5. Find out how to delete/move the files, or download them. This will be helpful in the next labs.

## 4. Submit a Job

Now we have everything we need to submit our sample application. It is finally time to open a shell.

### Steps:

1. Connect to [https://jupyter.polito.it](https://jupyter.polito.it).
2. Open a terminal by clicking on “Terminal” inside the Launcher area.
3. Launch a MapReduce job (i.e., run your application on the cluster) using the following command:
   ```bash
   hadoop jar Exercise1-1.0.0.jar it.polito.bigdata.hadoop.DriverBigData 2 example_data ex1_out
   ```
   - `Exercise1-1.0.0.jar` is the JAR file containing your application.
   - `example_data` is the input HDFS folder. A relative path starts in your home in HDFS. You can also use an absolute path in HDFS.
   - `ex1_out` is the output folder in HDFS, not on the gateway local file system. You can see its content in HUE (a relative path starts in your home in HDFS).

4. Check the number of mappers (i.e., the number of instances of the mapper class). You can retrieve this information from the terminal output during the execution of your application.

5. Find your job on the HUE interface (JobBrowser tab), check its status (submitted, running, failed, or succeeded).

6. Find the output file on HDFS (from HUE file browser) and view the results associated with the execution of your application.

7. Try to re-run the same job. Does it succeed this time? What’s the problem?
   - To remove a folder from HDFS, use HUE or the `hdfs` command line tool by executing:
     ```bash
     hdfs dfs -rm -r <path of the HDFS folder you want to delete>
     ```

8. Run the application again on the cluster by changing the number of reducers (first parameter of the application) and analyze the content of the output folder of the HDFS file system.

9. If you need to access the log files associated with the execution of your application, use the following commands in the terminal on `jupyter.polito.it`:
   - To retrieve the log associated with the standard output:
     ```bash
     yarn logs -applicationId <id of your application> -log_files stdout
     ```
     - The “id of your application” is printed on the terminal at the beginning of the execution of your application.
     - Example of “application id”: `application_1584304411500_0009`
     - You can also retrieve the application id from the HUE interface.
   - To retrieve the log associated with the standard error:
     ```bash
     yarn logs -applicationId <id of your application> -log_files stderr
     ```

## 5. Test on a Bigger File

Now you will execute your application on a larger file that we have already uploaded to the HDFS file system of BigData@Polito. The absolute path of that file in HDFS is:

`/data/students/bigdata-01QYD/Lab1/finefoods_text.txt`

It is a large collection of Amazon reviews in the food category. Each line of `finefoods_text.txt` contains one review.

### Steps:

1. Launch your application on the Amazon review file:
   - Set the second parameter of your application to `/data/students/bigdata-01QYD/Lab1/finefoods_text.txt`.

2. Analyze the results:
   - Can you understand any interesting facts from your results?
   - Do you see any space for improvements in your analysis?

3. The following figure was done on a small sample of your data (10,000 reviews):
   - Is it consistent with what you found on the complete dataset?
   - Do you think a small sample is enough to represent the whole?

## 6. Bonus Track

A word count can be seen as a special case of an n-gram count, where n is equal to 1. N-grams, in our context, are sets of contiguous words of length n. 

For example, in the sentence “She sells seashells by the seashore“, 2-grams are: “She sells”, “sells seashells”, “seashells by”, “by the”, “the seashore”.

### Tasks:

1. Modify your word count program to count 2-gram frequencies. Consider each line of text as a separate document, as in the Amazon reviews file (so, do not count contiguous words on separate lines).
2. What is the time/space complexity of your program?
3. How long would you expect it to run compared to the simple word count?
4. Try to run it on the toy text and on the Amazon reviews.