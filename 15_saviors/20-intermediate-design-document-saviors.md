#  The Saviors: intermediate report {#template-int-report status=ready}

## Part 1: System interfaces
<!--
Please note that for this part it is necessary for the system architect and software architect to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time. -->

### Logical architecture

**Description of the desired functionality:** 

Detect duckies and cones during lane following. When the ducky/cone is not in the middle of the lane we try to avoid them, if avoidance is not possible, we simply stop. Our obstacle avoidance capability is limited due to the fact that the controllers cannot make our Duckiebot cross the lane. In a first step, if we are in an intersection, our node will stop performing its tasks. However, if we progress really fast we will also have a look at the case of obstacles in intersections.  

**What will happen when we click "start"?**

If you click start our nodes are launched and we will start to detect duckies.

***In general our obstacle_detection_node is always active. The "flow" is as follows:*** 

**1.** Our “obstacle_detection”-node is activated and runs continuously. This nodes creates an instance of the class “detector” at the very beginning.

**2.** We have a an incoming filtered image stream from the “anti_instagram”-node to our node with the frequency, the “anti_instagram”-node is publishing. Our “obstacle_detection”-node regularly (i.e. with at least 2 Hz) calls the class functions of the “detector” instance.

**3.** For each frame the detector was called, the class function decides whether obstacles (duckies or traffic cones) are within the range of vision or not. Afterwards the “obstacle_detection”-node publishes an array containing all coordinates of the obstacles which is empty if there are none. The published topic is called “obstacle_coordinates”.

**4.** The “obstacle_avoidance”-node takes this array as input and analyses it regarding which actions should be performed. The following scenarios are possible: 

**4a)** The array is empty, therefore no action is performed and no flag will be published.

**4b)** At least one obstacle (e.g. a Duckie) is within the range of ¼ to ¾ of the lane (i.e. in the middle of our lane) such that it cannot be avoided without going to the opposite lane or driving outside of the street. In this case the “obstacle_avoidance”-node will set the “emergency_brake_flag” to true by publishing the topic “emergency_brake_flag”. This will indicate to “The Controllers” to immediately stop the Duckiebot.

**4c)** If an obstacle is detected and it lies within the left or right quarter of our lane it can be avoided. In this case we will set the “obstacle_flag” to true by publishing this topic and also provide the controllers an input d (again by publishing a topic) to drive on the right side or the left side of the lane respectively.

**5.** If we passed the obstacle or if the obstacle disappeared we reset the “emergency_brake_flag” or the “obstacle_flag” and the controllers can take over again.

**6.** Additionally we are listening to the “lane_following_flag” which indicates, if true, that the lane following is active. This tells us to perform our tasks. Otherwise our module should be inactive as our commands wouldn’t have any effects anyway. In this way we can save computing resources and avoid misunderstandings.

**7.** Additionally we implemented an additional file “obstacle_detection_visualizer” which can be used for visualization of the detected obstacles. It is thought to run on an laptop (which is connected to the same rosmaster) because it is only used for the evaluation of the quality of the detection as well as the 2D-3D projection. In principle it can run on the raspberry pi as well. It subscribes to “obstacle_coordinates”  and visualizes the obstacles in the 2D image by encircling the obstacles with a green rectangle (and publishing this modified image) as well as plotting them in the 3D coordinate frame (e.g. visualizable in RVIZ) as marker which shows the position as well as the size of the detected obstacles. With this additional software the optimization and the debugging are simplified significantly. 

**Expected target values for the following quantities:**

* Detection Distance: 30-40cm
* Certainty of Detections: 90 percent
* Max. Amount of Detections in one frame: 3
* Max. Ratio of False Positives: 20 percent
* Obstacles avoided (without crash) to Obstacle detected ratio: 80 percent
* Object detection frame rate: at least 2 Hz

**Assumptions on other modules:**

* Control reaches desired d at most 10cm after request. Our request is “continuous”. 
* Steady state control error smaller than 2cm
* d (perpendicular position to lane direction) Position estimate accuracy smaller than 2cm
* when setting the emergency_flag, we stop within the next 5 cm

<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

** List of nodes which are to be developped: **

* obstacle_detection 
* obstacle_detection_visualizer
* obstacle_avoidance 

** Published and subscribed topics for each node, including an estimate of the introduced latency for the topics being published and an assumption on the latency for all subscribed topics: **

***obstacle_detection***

*published topics:*

* obstacle_coordinates (max 0.5s)

*subscribed topics:*

* cameraImage (no latency)

***obstacle_detection_visualizer***

*published topics:*

* obst_detect/image/compressed (max 0.5s)

* obst_detect/visual/visualize_obstacle (max 0.5s)

*subscribed topics:*

* obstacle_coordinates (max 0.5s)

* cameraImage (no latency)

***obstacle_avoidance***

*published topics:*

* desired d (computing time)

* obstacle_avoidance_active_flag (max. 0.3s)

* obstacle_emergency_stop_flag (max. 0.3s) 

*subscribed topics:*

* obstacle_coordinates (max 0.5s)

* state_estimation (current d, velocity. probably theta) (0.2s?)

<!--
The above must have a check-off by the software architect:

Software architect check-off: I, Liam Paul, agree that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan
<!--
_Please note that for this part it is necessary for the VPs for Safety to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._ -->

### Demo plan
<!--
*The demo is a short activity that is used to show the desired functionality, and in particular the difference between how it worked before (or not worked) and how it works now after you have done your development.* -->

**How do you envision the demo?**

Duckiebot driving around Duckietown with duckies in the street (on straights, (in curves), not in intersections)

* First the standard code will be running, which will result in Duckiebots crashing into duckies on the street.

* Afterwards we will launch the Software from last year which will clearly show some strong losses in the quality of the obstacle detection.

* With our code running, the Duckiebot will avoid duckies by driving around them or stopping. If stopped we will remove the duckie which will make the bot drive again.

**What hardware components do you need?**

* Enough duckies and traffic cones for being able to perform the evaluation. Traffic cones are already ordered and will be shipped on Monday 11th of December at the latest.

* The minimum city size to properly perform the demo is in our opinion an equivalent size as the one in the ML building. 

### Plan for formal performance evaluation

**How do you envision the performance evaluation? Is it experiments? Log analysis?**

**1. Performance evaluation of the Saviors only:** 

*This first evaluation will be a general evaluation based on logs. We basically want to check if obstacles are detected correctly, and if the avoidance reacted as expected to prevent crashes.* 

**1a)** Detailed Evaluation Parameters:

- compare the labelled data (= groundtruth) to the results when putting the same image into our pipeline
- checking the acutal frequency of our node
- test duckie recognition at different orientations and evaluate the maximum angle possible in which we still detect duckies
- evaluate the precision when having duckies only on our lane and on both, our lane and the opposite lane!!! 
- evaluate the number of false positives in the 3 different situations: on straights, in “normal” curves, in the S-curve in duckietown
- measure the accuracy in centimeters of the real position of a duckie and the position we estimate!!!

**2. Performance evaluation for the Saviors / Controllers / State estimation:**

*In a first step we will evaluate our trajectory generation and control commandy by our obstacle_avoidance node in designed situations. This means that we assume having a certain state, position of an obstace and then we verify that we plan a feasible path around the obstacle without crossing the lane or to make the decision to stop*

**2a)** Detailed evaluation Parameters:

- percentage of correct decisions 

*The second step will be with the real state estimation, lane following and obstacle detection. This evaluation will be based on experiments with our duckiebot driving around town with obstacles in the lane.*

**2b)** Detailed evaluation Parameters:

1. percentage of correct decisions

2. percentage of correct executions when having made a correct decision, meaning really driving around the duckie without hitting it or sending the stop command early enough to not crash into duckies

**2c)** the 2 measures above will be evaluated in different situations:

1. only place duckies on straights

2. only place duckies in turns 

3. only place duckies in the S-Turns

4. no limit on placing of the duckies

**2d)** All of the above 4 situation will be also evaluated with:

- Variation 1: only duckies in our lane

- Variation 2: also duckies on the opposite lane possible


In our case, only the performance evaluation on bags can be designed for minimal human intervention whereas all the other test have to be done in presence of a human who is able to stop the system when a potential crash occurs.

<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, believe that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis
<!--
_Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._-->

### Collection

**How much data do you need?**

We already recorded 13 rosbags and are testing and improving the current obstacle detection. For the first step we think that this should work out.

**How are the logs to be taken? (Manually, autonomously, etc.)**

We took them manually but tried to “simulate” some non optimal controller behaviour, also because the autonomous mode did not work. For the future we might consider taking autonomous logs but only if we don’t crash into obstacles

**Do you need extra help in collecting the data from the other teams?**

No

### Annotation

**Do you need to annotate the data?**

Yes!

**At this point, you should have you tried using [thehive.ai](https://thehive.ai/) to do it. Did you? Are you sure they can do the annotations that you want?**

Yes we already tried it and it worked quite well. We use this platform to label duckies for us so that it will be possible to measure the performance of our duckie detection automatically on logs. Otherwise a human would need to label all of the pictures which would take a lot of time.

### Analysis

**Do you need to write some software to analyze the annotations?**

We plan to write some software which applies our algorithm on every image of the log and which will mark the duckies using a box around them. Afterwards it will use the annotated data to read out where they have drawn the box and calculate the distance as well as visualize these results and differences. We then try to calculate the percentage of reasonable results and the percentage of outliers. 

**Are you planning for it?**

Yes.

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
