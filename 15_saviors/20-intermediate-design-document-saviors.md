#  Group name: intermediate report {#template-int-report status=ready}

_It's time to commit on what you are building, and to make sure that it fits with everything else._

This consists of 3 parts:

- Part 1: System interfaces: Does your piece fit with everything else? You will have to convince both system architect and software architect and they must sign-off on this.

- Part 2: Demo and evaluation plan: Do you have a credible plan for evaluating what you are building? You will have to convince the VPs of Safety and they must sign-off on this.

- Part 3: Data collection, annotation, and analysis: Do you have a credible plan for collecting, annotating and analyzing the data? You will have to convince the data czars and they must sign-off on this.


<div markdown="1">

 <col2 id='checkoff-people-intermediate-report' figure-id="tab:checkoff-people-intermediate-report" figure-caption="Intermediate Report Supervisors">
    <s>System Architects</s>                         <s>Sonja Brits, Andrea Censi</s>
    <s>Software Architects</s>                       <s>Breandan Considine, Liam Paull</s>
    <s>Vice President of Safety</s>                  <s>Miguel de la Iglesia, Jacopo Tani</s>
    <s>Data Czars</s>                                <s>Manfred Diaz, Jonathan Aresenault</s>
 </col2>

</div>

## Part 1: System interfaces

Please note that for this part it is necessary for the system architect and software architect to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.

### Logical architecture

*- Please describe in detail what the desired functionality will be. 

Detect duckies and cones during lane following. If possible (ducky/cone not in the middle of the lane) avoid them or simply stop, because it is not possible for us to cross the lane (due to functionality boundaries from the controllers). If we are in an intersection our node will stop performing its tasks. 

*What will happen when we click "start"?*

If you click start our nodes are launched and we will start to detect duckies.
In general our obstacle_detection_node is always active. The structure is as follows: 

1. Our “obstacle_detection”-node is activated and runs continuously. This nodes creates an instance of the class “detector” at the very beginning. 
2. We have a filtered image stream from the “anti_instagram”-node to our node with the frequency the “anti_instagram”-node is publishing. Our “obstacle_detection”-node regularly (i.e. with at least 2 Hz) calls the class functions of the “detector” instance. 
3. For each frame the detector was called, the class function decides whether obstacles (duckies or traffic cones) are within the range of vision or not. Afterwards the “obstacle_detection”-node publishes an array containing all coordinates of the obstacles which is empty if there are none. The published topic is called “obstacle_coordinates”. 
4. The “obstacle_avoidance”-node takes this array as input and analyses it regarding which actions should be performed. The following scenarios are possible: 
	a. The array is empty, therefore no action is performed and no flag will be published.
	b. At least one obstacle (e.g. a Duckie) is within the range of ¼ to ¾ of the lane (i.e. in the middle of our lane) such that it cannot be avoided without going to the opposite lane or driving outside of the street. In this case the “obstacle_avoidance”-node will set the “emergency_brake_flag” to true by publishing the topic “emergency_brake_flag”. This will indicate to “The Controllers” to immediately stop the Duckiebot.
	c. If an obstacle is detected and it lies within the left or right quarter of our lane it can be avoided. In this case we will set the “obstacle_flag” to true by publishing this topic and also provide the controllers an input d (again by publishing a topic) to drive on the right side or the left side of the lane respectively. 
5. If we passed the obstacle or if the obstacle disappeared we reset the “emergency_brake_flag” or the “obstacle_flag” and the controllers can take control on their own again.
6. Additionally we are listening to the “lane_following_flag” which indicates, if true, that the lane following is active. This tells us to perform our tasks. Otherwise our module should be inactive as our commands wouldn’t have any effects anyway. In this way we can save computing resources and avoid misunderstandings. 
7. Additionally we implemented an additional file “obstacle_detection_visualizer” which can be used for visualization of the detected obstacles. It is thought to run on an laptop (which is connected to the same rosmaster) because it is only used for the evaluation of the quality of the detection as well as the 2D-3D projection. In principle it can run on the raspberry pi as well. It subscribes to “obstacle_coordinates”  and visualizes the obstacles in the 2D image by encircling the obstacles with a green rectangle (and publishing this modified image) as well as plotting them in the 3D coordinate frame (e.g. visualizable in RVIZ) as marker which shows the position as well as the size of the detected obstacles. With this additional software the optimization and the debugging are simplified significantly. 

*- Please describe for each quantity, what are reasonable target values. (The system architect will verify that these need to be coherent with others.)*
* Detection Distance: 30-40cm
* Certainty of Detections: 90%
* Max. Amount of Detections in one frame: 3
* False Positives : Obstacle - ratio: 20%
* Obstacles avoided (without crash): Obstacle detected - ratio: 80%
* Object detection frame rate: at least 2 Hz

*- Please describe any assumption you might have about the other modules, that must be verified for you to provide the functionality above.*
* Control reaches desired d at most 10cm after request. Our request is “continuous”. 
* Steady state control error < 2cm
* d (perpendicular position to lane direction) Position estimate accuracy < 2cm
* when setting the emergency_flag, we stop within the next 5 cm

<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

*- Please describe the list of nodes that you are developing or modifying.*

* obstacle_detection 
* obstacle_detection_visualizer
* obstacle_avoidance 

- For each node, list the published and subscribed topics.

* obstacle_detection
	
	* published topics:

		* obstacle_coordinates (max 0.5s)

	* subscribed topics:

		* cameraImage (no latency)

* obstacle_detection_visualizer

	* published topics:

		* obst_detect/image/compressed (max 0.5s)
		* obst_detect/visual/visualize_obstacle (max 0.5s)

	* subscribed topics:

		* obstacle_coordinates (max 0.5s)
		* cameraImage (no latency)

* obstacle_avoidance

	* published topics:

		* desired d (computing time)
		* obstacle_avoidance_active_flag (computing time)
		* obstacle_emergency_stop_flag (computing time)

	* subscribed topics:

		* obstacle_coordinates (max 0.5s)
		* state_estimation (current d, velocity. probably theta) (0.2s?)


*- For each subscribed topic, describe the assumption about the latency introduced by the previous modules.*

See above

*- For each published topic, describe the maximum latency that you will introduce.*

See above 
<!--
The above must have a check-off by the software architect:

Software architect check-off: I, Liam Paul, agree that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan

_Please note that for this part it is necessary for the VPs for Safety to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Demo plan

The demo is a short activity that is used to show the desired functionality, and in particular the difference between how it worked before (or not worked) and how it works now after you have done your development.

It should take a few minutes maximum for setup and running the demo.

- How do you envision the demo?

Duckiebot driving around Duckietown with duckies in the street (on straights, (in curves), not in intersections)

* First the standard code will be running, which will result in Duckiebots crashing into duckies on the street.
* Afterwards launching of the Software from last year which will clearly show some strong losses in the quality of the obstacle detection.
* With our code running, the duckiebot will avoid duckies by driving around them or stopping. If stopped we will remove the duckie which will make the bot drive again.

- What hardware components do you need?

* Enough duckies and traffic cones for being able to perform the evaluation. Traffic cones are already ordered and will be shipped on Monday 11th of December at the latest.
* The minimum city size to properly perform the demo is in our opinion an equivalent size as the one in the ML building. 

### Plan for formal performance evaluation

- How do you envision the performance evaluation? Is it experiments? Log analysis?

**Performance evaluation of the Saviors only:** by logs. 
- General evaluation: See if obstacles were detected correctly, and if the avoidance reacted as expected to prevent crash.

DETAILED:

- compare the labelled data (= groundtruth) to the results when putting the same image into our pipeline
- checking the acutal frequency of our node
- test duckie recognition at different orientations and evaluate the maximum angle possible in which we still detect duckies
- evaluate the precision when having duckies only on our lane and on both, our lane and the opposite lane!!! 
- evaluate the number of false positives in the 3 different situations: on straights, in “normal” curves, in the S-curve in duckietown
- measure the accuracy in centimeters of the real position of a duckie and the position we estimate!!!

**Performance evaluation for the Saviors / Controllers / State estimation:**
In a first step we will evaluate our trajectory generation and control commandy by our obstacle_avoidance node in designed situations. This means that we assume having a certain state, position of an obstace and then we verify that we plan a feasible path around the obstacle without crossing the lane or to make the decision to stop

EVALUATION PARAMETERS:

- percentage of correct decisions 

The second step will be with the real state estimation, lane following and obstacle detection. This evaluation will be based on experiments with our duckiebot driving around town with obstacles in the lane.

EVALUATION PARAMETERS:

1. percentage of correct decisions
2. percentage of correct executions when having made a correct decision, meaning really driving around the duckie without hitting it or sending the stop command early enough to not crash into duckies

the 2 measures above will be evaluated in different situations

1. only place duckies on straights 
2. only place duckies in turns and 
3. only place duckies in the S-Turns
4. no limit on placing of the duckies

All of the above 4 situation will be also evaluated with:

- Variation 1: only duckies in our lane
- Variation 2: also duckies on the opposite lane possible


*In contrast with the demo, the formal performance evaluation can take more than a few minutes.*

*Ideally it should be possible to do this without human intervention, or with minimal human intervention, for both running the demo and checking the results.*

Humans have to remove obstacles in case of an emergency brake. No way around that since controller is not able to drive on the left lane. 

<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, believe that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

_Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Collection

*- How much data do you need?*

We already recorded 13 rosbags and are testing and improving the current obstacle detection. For the first step we think that this should work out.

*- How are the logs to be taken? (Manually, autonomously, etc.)*

We took them manually but tried to “simulate” some non optimal controller behaviour, also because the autonomous mode did not work. For the future we might consider taking autonomous logs but only if we don’t crash into obstacles

*Describe any other special arrangements.*

*- Do you need extra help in collecting the data from the other teams?*

No

### Annotation

*- Do you need to annotate the data?*

Yes!

*- At this point, you should have you tried using [thehive.ai](https://thehive.ai/) to do it. Did you? Are you sure they can do the annotations that you want?*

Yes we already tried it and it worked quite well. We use this platform to label duckies for us so that it will be possible to measure the performance of our duckie detection automatically on logs. Otherwise a human would need to label all of the pictures which would take a lot of time.

### Analysis

*- Do you need to write some software to analyze the annotations?*

We plan to write some software which applies our algorithm on every image of the log and which will mark the duckies using a box around them. Afterwards it will use the annotated data to read out where they have drawn the box and calculate the distance as well as visualize these results and differences. We then try to calculate the percentage of reasonable results and the percentage of outliers. 

*- Are you planning for it?*

Yes.

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
