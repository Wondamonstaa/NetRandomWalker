# CS 441 Project 2 Fall 2023
## Kiryl Baravikou
### UIN: 656339218
### NetID: kbara5

Repo for the NetRandomWalker Project 2 for CS 441 Fall 2023

---

AWS EMR Deployment YouTube link: EDIT

---

## How to Run the Project:
1) Start by downloading the project repository from Git. Please, note, that the git repository contains the folder TO_USE. The following folder has all the essential resources I have been using to successfully run the project.
2) Navigate to the NetRandomWalker directory: use your terminal or command prompt to navigate to the "NetRandomWalker" directory within the downloaded project folder.
3) Before your run the project, please, make sure that your current setting satisfy the requirements: a) Scala version: 2.13.10; b) JDK version: 1.8; c) SBT version: 1.9.6; d) Apache Spark version: 3.5.0.
4) After all the prerequisites are met, please, go to File -> Project Structure -> Modules -> Add -> JARs or Directories -> select TO_USE folder -> netmodelsim.jar -> select the uploaded module -> set scope to 'Compile' -> click 'Apply' -> click 'OK'. By doing so I ensure that the essential injected binary files will not be missing. The following .jar file allows me to get access to the methods implemented in the original project created by Dr. Mark. Specified setting in the build.sbt allow to mix multiple Scala versions without any errors.
5) In your terminal, run the following command to clean, compile, and execute the project: `sbt clean compile run`. The following command successfully launched the Main class of the program which contains the essential methods for program execution of the written methods within other objects.
6) To run tests and ensure the code's functionality, use the following command: `sbt clean test`. The tests written are located within src/test/scala/RandomWalkerTest path directory. All the tests on my end ran successfully among with tests written by Dr. Mark.
7) If you need to create a .jar file for your project, execute the following command: `sbt clean assembly`. The resulting jar will be placed in __NetGameSim/target/scala-3.2.2/netrandomwalker.jar__. Alternatively, a user can use the manual approach to create a .jar file of the project: File -> Project Structure -> Artifacts -> Add -> JAR -> From modules with dependencies -> Specify the main class of the project -> click 'OK' -> go to Build -> Build Artifacts -> in the “Build Artifacts” dialog window, click on “Build” to generate your JAR file. The generated .jar file will be located in the specified output directory.
8) If you prefer using IntelliJ for development: a) Import the project into IntelliJ IDE. b) Build the project. c) Create a "configuration" for the Main.scala file. d) Set the configuration's arguments to be the input and output folders, separated by a space. NB: to run the project properly, a user, i.e. you, should specify the input/output paths in the application.conf file. The current configurations are set to default values which you may have to change to run the project successfully.
9) Ensure that your local input/output folder has the necessary permissions to allow the program to read from and write to it. You can check and adjust folder permissions by right-clicking on the project directory, selecting "Properties," and navigating to the "Security" tab. Make sure that the group "Users" has both "Read" and "Write" permissions.
10) Before running the program, ensure that Hadoop is running on your machine. The program may depend on Hadoop, so it's essential to have it set up and running correctly.
---

## Requirements:

The goal of the project is to create a program for parallel distributed processing of large graphs to produce matching statistics about generated graphs and their perturbed counterparts, and develop a program for simulating parallel random walks on the generated graphs, with the aim of modeling and understanding the behavior of Man-in-the-Middle attackers. The following core functionalities must be met:

1. Statistics Generation: compute a YAML or CSV file that displays the number of successful attacks and the failed attacks for a given number of iterations. Other metrics can include the statistics about random walks, e.g., min/max/median/mean number of nodes in these walks and the ratio of the number of random walks resulting in successful attacks to the total number of random walks.
2. Random Walk: implement the standard Random Walk algorithm using Apache Spark which simulates a traversal of the graph in which the traversed relationships are chosen at random.
3. Attack simulation: using the implemented Random Walk algorithm, simulate an attack on the large distributed system. The goal of the each iteration is to compute the statistics about the number of successful/unsuccessful attacks. 
4. Custom Graph Matching Algorithm: develop a uniquely fine-tuned graph matching algorithm to make determinations about modifications.
5. Comparison with Golden Set YAML: compare each detected modification with the golden set of changes generated by NetGraphSim when creating a perturbed graph.
6. Algorithm Evaluation: assess the goodness of your algorithm, similar to precision and recall ratios, based on experimentation and results.

Other Requirements:
1) The output files should be in the format of .csv or any other human-readible format.
2) 5 or more scalatests should be implemented.
3) Logging used for all programs.
4) Configurable input and output paths for the Apache Spark programs.
5) Compileable through 'sbt clean compile run'
7) Deployed on AWS EMR demonstrating all the steps of deployment.
8) Video explanation of the deployment recorded.
9) RandomWalk algorithm must be implemented.
10) Sophisticated documentation with clarifications of the chosen design rationale.

---

## Technical Design

This section will provide a detailed description of each of the classes that I implemented in this project, as well as the rationale for using one or another class when planning and developing the design of an algorithm for calculating similarities in the generated graphs:

1. Main: 

As the main class, I chose the Main object, implemented by Dr. Mark Grechanik. This object includes all the necessary imported libraries and modules that allow the code to work safely. The input point of this object is the 'main' function, within which random graphs are generated by randomized generation of nodes and edges. The generated graphs are saved to the directory specified in application.conf. After the generation of graphs is completed, the algorithm I wrote finds the location of the generated graphs, accesses them and sends the graph data to the ArgumentParser object, which is responsible for collecting the necessary arguments for subsequent cascading calls to the functions of this project. After receiving all the necessary arguments, including graphs in .ngs format, using the load() method, I load the graphs into the system and create two objects of type Option[NetGraph], which I subsequently use as arguments to call the processFile() method from the DataConverter object.

# Sample output:

_________________

![WhatsApp Image 2023-09-11 at 10 02 36 PM](https://github.com/Wondamonstaa/NetGameSim_Project1/assets/113752537/208bc260-e8e5-4099-a62d-7e72a49561a1)


_________________


2. DataConverter:

DataConverter object is responsible for converting the graphs into a human-friendly readable format, and for the subsequent sharding of the generated files for the purpose of parallel processing in Map/Reduce model. After receiving the serialized graphs as arguments, the processFile() function begins processing them simultaneously by calling the processFilesToCsv() and processEdgesToCsv() functions to access nodes and edges of the two graphs via using nodes() and edges() function calls respectively. Subsequently, shuffling of objects occurs with each other in order to create all kinds of combinations that will be used as arguments when calculating similarities via SimRank in the Map/Reduce. After generating combinations, the sharder() function is called, which splits the resulting files into shards, the size of which depends on the argument specified when calling this function. All received shards are saved in two specified folders for user convenience.

# Sample output:

____________________

<img width="1256" alt="Screenshot 2023-10-10 at 10 16 48 AM" src="https://github.com/Wondamonstaa/NetGameSim_Project1/assets/113752537/6b87b301-acce-4afb-b176-e0ed531b2b27">


____________________

3. SimRankMapReduce: 

The purpose of this object is to construct a Map/Reduce model, which operates exclusively on <key, value> pairs, that is, the framework views the input to the job as a set of <key, value> pairs and produces a set of <key, value> pairs as the output of the job, conceivably of different types. Inside the SimRankMapReduce object, an object of the SimRank class is created, which allows you to access methods for highlighting the similarities between two graphs and at the same time reducing the code pollution of the SimRankMapReduce object. Inside Mapper, the main method for the initial processing of received files is the map function. I use LongWritable and Text respectively as the input key and value since the CSV shard is read line by line. The output values for <key, value> pairs are Text and DoubleWritable. Inside map(), while reading a file line by line, its similarity rank is calculated via invoking calculateSimRank() function by passing each line of the processed CSV file as an argument. Thus, each line of the file is processed, followed by registration of NodeID and its similarity score for further processing. The program transfers these values to Reduce, where the final processing of the received data takes place. 

Next, the reducer accepts ad outputs the following arguments [Text, DoubleWritable, Text, Text]. Inside the reducer, using the earlier generated object, I invoke calculateSimRankWithTracebilityLinks() function, and pass (key, values) as parameters. The following function calculates the number of possible traceability links between the original and perturbed nodes, F-1 score, specificity, and precision/recall ratio. The above calculations are recorded in the Text format and written to the output file, generated by a Map/Reduce Hadoop 3.3.6 model. As you can see on the screenshot below, the following algorithm calculates the number of tracebility links by traversing across the graph from node to node via the edges, and comparing the properties of the nodes from the original and perturbed graphs. Based on the results of the comparison, and the number of matches between the two, different score is assigned, which is later compared to the selected threshold. After the calculations are done, they are processed inside the Map/Reduce model, the results are sorted, reduced, and produced to the output file called 'part-r-00000'. Please, keep in mind that this is the file you will need to check in order to obtain the results. The following file will be located at the specified folder for the output files from the selected Map/Reduce job. For further explanation of the obtained results, please, refer to the last part of the report called "Note for Dr. Mark Grechanik and Utsav". 

___________________

# Sample output:

<img width="250" alt="Screenshot 2023-10-10 at 10 41 08 AM" src="https://github.com/Wondamonstaa/NetGameSim_Project1/assets/113752537/bb0ec02c-da52-4704-b1d1-f064863815fa">


___________________


4. SimRank:

The following object, as mentioned earlier, is used to calculate the similarities between two graphs by analyzing the properties nodes and edges from both graphs. Using the given threshold, a similarity score is calculated, which subsequently serves as the output value in the Map and the input value in the Reducer. Based on the similarity score, as stated above, traceability links between nodes and edges are calculated.


5. EdgesSimilarityMapReduce:

This component serves the purpose of establishing a Map/Reduce model that exclusively operates on pairs of data denoted as <key, value>. Inside the EdgesSimilarityMapReduce component, an instance of the SimRank class is created. This instantiation enables us to access methods for evaluating similarities between two graphs, specifically edges, all while reducing the code complexity within the EdgesSimilarityMapReduce object.

In the Mapper phase, the primary method for the initial processing of input files is the map function. LongWritable and Text are used as the input key and value types, as we process the CSV file line by line. The output values for <key, value> pairs consist of Text and DoubleWritable. Within the map() function, while processing each line of the file, we calculate its similarity rank by invoking the calculateEdgeSimRank() function with each line of the processed CSV file as an argument. As a result, each line of the file undergoes processing, accompanied by the registration of NodeID and its corresponding similarity score for subsequent steps. These values are then transmitted to the Reduce phase, where the final processing of the received data unfolds. The driver of this class is the runEdgeSimMR(inputPathE: String, outputPathE: String) method, which takes the path to the directory with shards for edges, and the output path for the produced result as the second argument.


_______________

# Sample output:

<img width="197" alt="Screenshot 2023-10-10 at 10 46 20 AM" src="https://github.com/Wondamonstaa/NetGameSim_Project1/assets/113752537/17c0556c-b977-4310-86c0-eb75f63ac190">

________________


6. NodeSimilarity:

The following object serves its primary role as a helper and the basis for implementing the methods in other objects, including SimRank.


## Test Cases

The tests can be run using 'sbt clean test' command or by directly invoking the class SimRankTest, located under the following path: src/main/Test/SimRankTest.scala

In essence, the battery of tests conducted evaluates the performance and functionality of the SimRank class, alongside the sharding function, assessing their proficiency in executing tasks accurately and effectively. These tests serve as a robust validation mechanism, ensuring that both components operate seamlessly and in accordance with their intended objectives.


## Limitations:

1) For local program execution, the user needs to have Java 8 or a higher version, sbt 1.6, and Hadoop 3.3.6 installed.
2) The program supports processing multiple files within the same input folder if the user chooses to split the files, but it cannot manage input files from different locations. 
3) The user should possess the capability to provide Read/Write permissions to the "Users" group for the LogFileGenerator project folder. This typically requires Administrator-level access.
4) The feature for altering the name and extension of the output file is effective only during local execution. In other words, it does not modify the name and extension in the case of program execution on AWS EMR, particularly in S3 Bucket.


## Note for Dr. Mark Grechanik and Utsav

___________

The final part of this assignment requires us to compare the produced results with the results produced by Dr. Mark in the YAML file to estimate the goodness of our algorithm. Carrying out this comparison makes sense only if the algorithm is mostly free of gross errors, which minimizes the chance of obtaining an inaccurate result. In my case, the algorithm is imprecise and can sometimes be confusing when analyzing the generated data. When writing the algorithm, due to the lack of a clear understanding of how exactly to work with this kind of data, and due to lack of experience, I did not take into account a number of important factors that play a key role in determining the percentage of similarity between nodes and edges of both graphs. Taking into account all the above, I conclude that my algorithm is inaccurate and cannot be used at this stage for the purpose of accurately comparing the results obtained with the results of Dr. Mark. The implemented algorithm still needs careful refinement and additional improvements in order to increase the accuracy of calculations.

To draw a conclusion, I would like to thank you for the chance to get a feel for a real example of what real programming is like that we will encounter in a real work environment. Realizing the number of hours that were invested in writing this project, and, frankly speaking, the low level of the result obtained, which cannot but upset, this allows me to rethink the approach I use to writing projects. This project made me realize the importance of planning the design of a future project at its earliest stages, as well as the need to ask questions even if there are even the slightest gaps in understanding what you have to work with. A lot of mistakes made when writing the project in its early stages significantly complicated the work process later, which led to the need to rewrite the code over and over again, which, in turn, gave rise to bugs in other areas of the program. Finally, I would like to note once again that the opportunity to work on tasks of this level is invaluable, and subsequently I will work on mistakes in order to prevent them when writing upcoming projects: CS 441 is notorious among students for its complexity, but at the same time it is one of the best if not the best, course at UIC to gain real world experience that will help anyone who completes it become the best version of themselves.

Thank you for your hard work, time, patience, and understanding!
___________
