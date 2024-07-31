# Lab 7: Analyzing Barcelona Bike Sharing Data with Apache Spark

In this lab, you will analyze historical data about bike-sharing stations in Barcelona. The goal is to identify the most "critical" timeslot (day of the week, hour) for each station and visualize this data using a KML file.

## Input Files

1. **register.csv**: Contains historical data about bike-sharing stations.
   - **Path**: `/data/students/bigdata-01QYD/Lab7/register.csv`
   - **Format**:
     ```
     stationId\ttimestamp\tusedslots\tfreeslots
     ```
   - Example:
     ```
     23 2008-05-15 19:01:00 5 13
     ```
   - The first line contains the header.
   - Filter out lines where `usedslots = 0` and `freeslots = 0`.

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

## Task 1: Identify Critical Timeslots

1. **Transform Data**:
   - Create a PairRDD from `register.csv` where each pair consists of:
     ```
     (user_id, list of product_ids reviewed by user_id)
     ```
   - Ensure each product_id appears only once per user.

2. **Compute Criticality**:
   - Define criticality as:
     ```
     Criticality = Number of critical readings / Total readings in the timeslot
     ```
   - A station is considered "critical" if `freeslots = 0`.

3. **Filter and Select**:
   - Compute the criticality for each combination of station and timeslot (day of the week and hour).
   - Filter pairs with criticality greater than or equal to a minimum threshold (provided as an argument).
   - For each station, select the timeslot with the highest criticality. If there are ties, choose the earliest hour, and if needed, the lexicographically earliest weekday.

4. **Output in KML Format**:
   - Generate a KML file with the following format:
     ```xml
     <Placemark>
       <name>StationId</name>
       <ExtendedData>
         <Data name="DayWeek"><value>DayOfTheWeek</value></Data>
         <Data name="Hour"><value>Hour</value></Data>
         <Data name="Criticality"><value>CriticalityValue</value></Data>
       </ExtendedData>
       <Point>
         <coordinates>Longitude,Latitude</coordinates>
       </Point>
     </Placemark>
     ```
   - Include only stations with critical timeslots satisfying the threshold.

   - **Example KML Format**:
     ```xml
     <Placemark>
       <name>44</name>
       <ExtendedData>
         <Data name="DayWeek"><value>Mon</value></Data>
         <Data name="Hour"><value>3</value></Data>
         <Data name="Criticality"><value>0.5440729483282675</value></Data>
       </ExtendedData>
       <Point>
         <coordinates>2.189700,41.379047</coordinates>
       </Point>
     </Placemark>
     ```

## How to Visualize the KML File

- Copy the KML content into a file formatted as follows:
  ```xml
  <kml xmlns="http://www.opengis.net/kml/2.2">
    <Document>
      <!-- Paste the output here -->
    </Document>
  </kml>
  ```
- Use KML viewers such as:
  - [KML Viewer](https://kmlviewer.nsspot.net)
  - [Ivan Rublev KML Viewer](https://ivanrublev.me/kml)
  - [GPS Visualizer](https://www.gpsvisualizer.com)

## Accessing Log Files

If you submit your application with `--deploy-mode cluster`, the standard output is stored in the log files. To retrieve these logs:

1. **Retrieve Application ID**:
   - Run your application with `--deploy-mode client` and add the following line:
     ```java
     System.out.println("ApplicationId: " + JavaSparkContext.toSparkContext(sc).applicationId());
     ```
   - Or use:
     ```sh
     yarn application -list -appStates ALL | grep 'sXXXXX'
     ```
     Replace `sXXXXX` with your username.

2. **Get Log Files**:
   - Use:
     ```sh
     yarn logs -applicationId application_id
     ```
     Replace `application_id` with the actual application ID.

## Shutting Down JupyterHub Container

After completing your tasks:

1. Go to `File -> Hub Control Panel`.
2. Click the “Stop My Server” button and wait until it disappears.