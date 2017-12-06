#  Implicit Coordination: intermediate report {#formation-and-implicit-control-int-report status=ready}


## Part 1: System interfaces


### Logical architecture
The first part of the project is to detect and track Duckiebots in general. In addition the motion of the Duckiebot should be predicted in a short time horizon. With this detection it should be able to get information of how many Duckiebots are at an intersection and if the way is free to cross the intersection. We will provide a protocol to trigger the driving through intersections.

The detection is also used to implement a demo in which a number of Duckiebots will follow a leader-Duckiebot. The leader will be controlled with forward navigation in Duckietown. The architecture is divided into the following logical submodules:

Detection

* Object bounding boxes and class outputted in a topic (in image space - per image no tracking) (2pts inside the image 640x480 + class)
* on your laptop you should see in rqt_image_view an image with bounding boxes around robots
- For Follow-the-leader:
    - Easy: with fiducial marker detection
    - Hard: With detection see (i) and (ii)

Tracking

- Output of the 2D trajectory (x,y) (stretch - pose) of each of the detected objects
- visualization of the trajectories in RVIZ

Prediction

- Output of a 2D class of trajectories with probabilities for future motion of objects
- In RVIZ a class of future object trajectories weighted by something

Intersection Coordination

- An agent (Duckiebot) infers when it is safe to go without explicit communication. The agent follows a protocol known to each agent on the road.
- Duckietown Intersection Coordination Protocol (DICP):
    - Inspired by CSMA/CD algorithm
    - Constraint: only one Duckiebot at a time crossing the intersection

Follow-controller

- Controller gives wheel velocity commands to follow a detected/tracked Duckiebot to keep a constant distance behind the Duckiebot in front.
- For centering the Duckiebot behind the one in front of him we will use the lane controller from the “Controllers”.
- Velocity control fells into the scope of our tasks.

<div figure-id="fig:CoordinationModule" figure-caption="The Coordination Module">
     <img src="CoordinationModule.png" style='width: 15em'/>
</div>

<div figure-id="fig:FormationKeepingModule" figure-caption="The Formation Keeping Module">
<img src="FormationModule.png" style='width: 15em'/>
</div>

Easy: Own robot is not moving

Hard : Own robot is moving

**Assumptions about other modules**

Intersection Coordination:

- Duckiebot knows when it is at an intersection, coordination_node receives msg when at stopline. Given by Controllers.
- Duckiebot knows how to navigate through an intersection, only needs a wait/go msg from our side (interface with navigators)
Formation Keeping:
- We can use the lane controller from “Controllers” to center a Duckiebot behind the Duckiebot in front.
We can use the obstacle avoidance for the leader Duckiebot to avoid duckies. Given by Savior.
- The Duckiebot will exit the Formation Keeping mode as soon as the Duckiebot in the front leaves the lane (overtakes or avoids an obstacle) and at intersections.  


<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

The software architecture is divided into the following software nodes:

Detection:

- One node - input camera image, output detections
define a msg type
Tracking:
- One node - input detections, output trajectories - define message type
    - For Hard need the odometry of robot

Prediction : later…

Intersection Coordination:

- One node - input msg from tracking node, msg stop-line filter - output boolean msg go/wait
    - Input: flag_at_stop_line -> True if Duckiebot is at intersection
    - Input: Tracking_msg -> Are other Duckiebots at intersection or will arrive at intersection in the prediction horizon
    - Output: flag_wait_go -> True if Duckiebot can navigate through intersection

Controller (Follow-the-leader/Formation Keeping):

- Easy:
    - One node - input pose of Duckiebot - output car control msg
        - Input: geometry_msgs/PoseStamped -> Estimated pose from Duckiebot in front to follower
        - Output: Msg for lane controller from “Controllers -> Msg for generating motor commands
        - flag_follow set true
- Hard:
    - One node - input pose of Duckiebot - output car control msg
        - Input: Tracking/detection msg
        - Output: Msg for lane controller from “Controllers -> Msg for generating motor commands


**Latency introduced by the subscribed modules**

Will be added

**Latency introduced by our modules**

* We need to do some back of the envelope math for the functionality: turn, look, detect, turn back, go.
* Latency for Follow-the-leader controller: max 15ms
* Latency for boolean msg at intersection: to be benchmarked



<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan

### Demo plan

Dream scenario intersection coordination: Duckiebots loop through the map as indicated in [](#fig:DemoMap). At the intersections the Duckiebots coordinate implicitly. Demo can run indefinitly without collision.  Tunable: density of the traffic (fleet size) and the speed.

Dream scenario follow-the-leader: Leader drives with indefinite navigation through Duckietown and other Duckiebots follow him. The Duckiebots are able to keep up with the leader.

Required Hardware Components

 - Circle grid on Duckiebots
 - The tiles to build the map in [](#fig:DemoMap)

 <div figure-id="fig:DemoMap" figure-caption="The Demo Map">
      <img src="DemoMap.png" style='width: 15em'/>
 </div>


### Plan for formal performance evaluation
<div markdown="1">

 <col5 class="labels-row1" id='implicit-coord-evaluation' figure-id="tab:implicit-coord-evaluation" figure-caption="Plan for performance evaluation">
    <s>**What is evaluated**</s>                      <s>**How**</s>
    <s>**Required**</s>                  <s>**Collected quantities**</s>
    <s>**Performance measure(s)**</s>
    <s>Coordination implementation performance</s>
    <s>Run the demo for intersections</s>
    <s>Up to four Duckiebots at an intersection, different configurations have to be analyzed (three or four way intersection)</s>
    <s>Time required to clear the intersection</s>
    <s>Mean clearing time for each configuration separately and for all combined.</s>
    <s>Vehicle detector</s>
    <s>Run the detection node</s>
    <s>Two to four Duckiebots at an intersection</s>
    <s>Number of TP, FP, FN</s>
    <s>-Precision
    -Recall</s>
    <s>Follow the leader</s>
    <s>Run the demo for follow the leader</s>
    <s>More than 4 Duckiebots
    Duckietown (no special requiremtens)</s>
    <s>Distance between Duckiebots</s>
    <s>-Number of Duckiebots in formation
    -Mean error for desired distance
    </s>

 </col5>

</div>


<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis


### Collection

8000 frames are logged. Sent to the hive.

We don’t need extra help in collecting the data from the other teams.

**Method for taking logs**

Two robots are taking logs and there are 6-7 other robots around town. Different sizes colors. Shells/no shells.

### Annotation


### Analysis


<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
