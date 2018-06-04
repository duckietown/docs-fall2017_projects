#  The Controllers: Intermediate Report {#controllers-int-report status=ready}

<!--
_It's time to commit on what you are building, and to make sure that it fits with everything else._

This consists of 3 parts:

- Part 1: System interfaces: Does your piece fit with everything else? You will have to convince both system architect and software architect and they must sign-off on this.

- Part 2: Demo and evaluation plan: Do you have a credible plan for evaluating what you are building? You will have to convince the VPs of Safety and they must sign-off on this.

- Part 3: Data collection, annotation, and analysis: Do you have a credible plan for collecting, annotating and analyzing the data? You will have to convince the data czars and they must sign-off on this.
-->




 <col2 align='center' style="text-align:left"  figure-id="tab:checkoff-people-intermediate-report-controllers" figure-caption="Intermediate Report Supervisors">
    <span>System Architects</span>                         <span>Sonja Brits, Andrea Censi</span>
    <span>Software Architects</span>                       <span>Breandan Considine, Liam Paull</span>
    <span>Vice President of Safety</span>                  <span>Miguel de la Iglesia, Jacopo Tani</span>
    <span>Data Czars</span>                                <span>Manfred Diaz, Jonathan Aresenault</span>
 </col2>





 <col2 align='center' style="text-align:left" id='conventions' figure-id="tab:conventions" figure-caption="Conventions used in the following document">
    <span>Variable</span>                        <span>Description</span>
    <span>$d_{ref}$</span>                         <span>Reference distance from center of lane</span>
    <span>$d_{act}$</span>                       <span>Actual distance from center of lane</span>
    <span>$d_{est}$</span>                  <span>Estimated distance from center of lane</span>
    <span>$\theta_{act}$</span>           <span>Actual angle between robot and center of lane</span>
    <span>$\theta_{est}$</span>           <span>Estimated angle between robot and center of lane</span>
    <span>$c_{act}$</span>          <span>Actual curvature of lane</span>
    <span>$c_{est}$</span>                  <span>Estimated curvature of lane</span>
    <span>$c_{ref}$</span>                   <span>Reference curvature of the path to follow </span>
    <span>$v_{ref}$</span>                   <span>Reference velocity</span>
 </col2>


![image](problem_statement.svg)



## Part 1: System interfaces

<!--
Please note that for this part it is necessary for the system architect and software architect to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.
-->

### Logical architecture

<!--
- Please describe in detail what the desired functionality will be. What will happen when we click "start"?
-->

**Desired functionality**    

We assume that the Duckiebot is placed on the right lane of the road within a defined boundary (as described in our preliminary design document) for the initial pose. By starting the lane following module it should begin to follow the lane, whether it is straight or curved, until we stop the lane following module. The module consists of two parts, a pose-estimator part and a lane-controller part. We will briefly describe these two modules:

<center><img figure-id="fig:curve_plot" figure-caption="Pose of Duckiebot in a curve element." src="curve_plot.png" alt="Curve plot" style="width: 200px;"/></center>




* **Pose-estimator**: Estimates distance $d_{est}$ from the center of the lane and the angle $\theta_{est}$ with respect to the center of the lane as near as possible to the actual values $d_{act}$ and $\theta_{act}$. In a curve, $\theta_{act}$ is the angle between the centerline of the Duckiebot and the tangent to the centerline of the lane at the corresponding position (where the origin of the robot frame (center of the wheel axis) is closest to the centerline of the lane). Additionally, the estimator estimates the curvature $c_{est}$ of the lane. Further if possible, the estimator will approximate the lane width and the width of the side lines to be robust with respect to the geometric specifications of Duckietown. (The curvature $c_{est}$ was not estimated in the previous estimator.)

* **Lane-controller**: Given the pose ($d_{est}$, $\theta_{est}$) and curvature ($c_{est}$) estimation and a reference $d_{ref}$, the lane-controller will control the Duckiebot along this reference. The controller only accepts $d_{ref}$ which allow the Duckiebot to stay in the right lane. In other cases, the Duckiebot will be stopped to avoid accidents and a corresponding flag flag_obstacle_emergency_stop is set. The lane-controller further strictly limits the velocity of the Duckiebot, for values see next section. The lane-controller takes the velocity at all time from implicit coordination except when another team demands a lower value. In case no velocity is received, we set a default constant velocity by ourselves.

The following diagram shows the input the controller node needs to control for other teams. 

<center><img figure-id="fig:node" figure-caption="Diagram showing values needed by the controller if used by other teams." src="node_IO.png" alt="nodeIO" style="width: 350px;"/></center>


Special events:

* **Detection of red line**: After the stop line has been detected by the stop_line_filter_node, it sends an at_stop_line message (flag_at_stop_line), the controller will continually slow down the velocity of the Duckiebot and stop between 16 to 10 cm from the center of the red line and with an angle $\theta_{act}$ of +-10° (requirements given by Explicit Coordination Team). Furthermore the Duckiebot will stop in the center of the lane within a range of +- 5 cm.

<center><img figure-id="fig:red_line" figure-caption="Pose and distance range in front of red line." src="red_line_stop.png" alt="stop at red line" style="width: 300px;"/></center>

* **Intersection**: When the Parking team set the flag_at_intersection _true_ (because the Duckiebot is at a stop line and there is no april tag for a parking lot), we will stop the pose_estimator of the lane-following module and listen to data provided by the Navigators. It consists of the curvature $c_{ref}$ the Duckiebot needs to follow, as well as a reference $d_{ref}$ (which should be zero in case the Duckiebot needs to drive on the path with given curvature), the estimates of our distance $d_{est}$ and angle $\theta_{est}$ with respect to the path and a desired velocity $v_ref$. Everything on the intersection except of using our standard lane following controller is out of our scope. The pose_estimator of the lane-following module will start again after the intersection, triggered when the flag flag_at_intersection is turned _false_.


* **Obstacle avoidance**: Once flag_obstacle_detected is set _true_ by the Savior Team, they will continuously send us references $d_{ref}$ to lead us around the obstacle, as well as a desired velocity $v_ref$. For better control performance, the velocity can be set lower than the usual velocity. The Saviors will start to send references when the Duckiebot still has a distance to the obstacle of at least 20-30cm to make sure the controller is able to react enough in advance. Out of scope is the controlled obstacle avoidance involving leaving the right lane. This case is determined by a stop flag (flag_obstacle_emergency_stop) sent from the Saviors module and the Duckiebot will stop. Since there is also a stop flag received from the stop_line_filter_node, we will introduce priorities for the several flags received to decide how the Duckiebot should behave.

* **Parking**: At a stop line, if a parking-lot april tag is detected (by the Parking team), the parking flag flag_at_parking_lot will be set _true_ (otherwise the flag_at_parking_lot would be set _false_ and the flag_at_intersection would be set _true_). After this the Parking team will take over responsibility. They will first calculate a path using RRT (Rapidly-Exploring Random Tree). In case, they set the flag_parking_stop to _false_, the lane controller will take over the control along this precalculated path. For this, the parking team needs to send the curvature $c_{ref}$ the Duckiebot needs to follow, as well as a reference $d_{ref}$ (which should be zero in case the Duckiebot needs to drive on the path with given curvature), the estimates of our distance $d_{est}$ and angle $\theta_{est}$ with respect to the path and a desired velocity $v_{ref}$. In case we need to stop, the Parking team will set the flag_parking_stop _true_ again. Driving backwards is not in our scope. After leaving the parking lot, we will take over estimation and control again, unless directly after the parking lot is an intersection, which would be handled by the Navigators (after Explicit and/or Implicit Coordination). 

* **Implicit Coordination**: If Implicit Coordination is running, they send to us the desired reference velocity $v_{ref}$ at all time and set the flag_implicit_coordination.

* **Fleet-level Planning**: As part of simulating pick-up / drop-off of customers, the Fleet-level Planning team will want to stop the Duckiebot at a certain distance from the center of the lane $d_{ref}$. Therefore they give us the desired $d_{ref}$ and a declining reference velocity $v_{ref}$ until the desired full stop.
<!--
- Please describe for each quantity, what are reasonable target values. (The system architect will verify that these need to be coherent with others.)
-->

**Target values**

The Duckiebot should run at a reduced velocity of 0.2 m/s for optimal controllability. The reason for the limited velocity is the low image update frequency which limits our pose estimation update, hence a lower velocity enhances the performance of our lane-follower module. Since not every Duckiebot has the same gain set, we will pass the desired velocity to team System Identification. Their module will convert the demanded velocity to the according input voltages for the motors.

Our goal is to control the deviation from the middle of the lane $d_{act}$ smaller than +- 2cm. The estimator should estimate our pose with $d_{est}$ and $\theta_{est}$ with an accuracy of +- 1cm. Further, the Duckiebot will stop in the center of the lane within a range of +- 2 cm, although a larger range of +- 5 cm is given by the explicit coordination team.

Whenever we detect a red line, we will slow down the Duckiebot and stop between 16 to 10 cm from the center of the red line and with an angle $\theta_{act}$ of +-10°, see caption 4. 

In case the flag_obstacle_detected is activated by the Saviors, they will provide us with the input described above to avoid the obstacle without leaving the lane. Otherwise they activate the flag_obstacle_emergency_stop and we will need to stop the Duckiebot within 5 cm from the position at which the flag_obstacle_emergency_stop is received (requirement by Saviors).

<!--
- Please describe any assumption you might have about the other modules, that must be verified for you to provide the functionality above.
-->

**Assumptions**

We assume the following modules will behave in the described manner:

* **Savior**: They will detect obstacles on the road and are able to generate $d_{ref}$ that allows to avoid the object without leaving the lane or touching any object (safety for the Duckies!). They will set the flag_obstacle_detected 20-30cm in front of the obstacle, so we have enough time to avoid the obstacle. They are able to decide if avoidance of an obstacle is feasible or they have to set the flag_obstacle_emergency_stop.

* **Anti-instagram**: They can compensate the colors for different ambient light conditions and allow for good edge detection robust to changing light conditions. In future, they could optionally help by introducing an area of interest of the image to process. Irrelevant image data could be filtered out to speed up the image processing pipeline. If Anti-Instagram could publish the area of interest as a node, we could use it in the estimation part to remove all visual clutter in the line segments.

* **Line detection**: Relevant line segments of the side, center and stop line are detected without introducing an exceeding amount of false detections and passed on in a SegmentList, classified including direction (from background do line or vice versa).

* **Navigators**: They are able to generate a path along the intersection and deliver accurate pose estimates which allow for proper working of the lane controller.

* **Parking**: They are able to generate a path from the parking entrance to a free parking space and deliver accurate pose estimates which allow for proper working of the lane controller. They take care of any backward driving without using the lane controller.

* **Implicit Coordination**: They are able to generate a reference velocity that will not result in a crash with another Duckiebot. They will provide enough distance to the Duckiebot in front, to allow the pose_estimator of the lane-following module and the stop_line_filter_node to work properly.

* **Fleet-level Planning**: They provide a profile of reference distance from the center of the lane $d_{ref}$ and velocity $v_{ref}$, such that they stop at their desired location to pick-up / drop-off their customer.

<!--
The above must have a check-off by the system architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

<!--
- Please describe the list of nodes that you are developing or modifying.

- For each node, list the published and subscribed topics.

- For each subscribed topic, describe the assumption about the latency introduced by the previous modules.

- For each published topic, describe the maximum latency that you will introduce.
-->

_Lane Filter Node_    

The Lane Filter Node will, in addition to the existing fields, also estimate the curvature.    

In the following table, published topics are listed:


<col2 align='center' style="text-align:left" id="lane_filter_published_topics" figure-id="tab:lane_filter_published_topics" figure-caption="Published topics by Lane Filter Node">
    <span>Topic</span>                                <span>Max Latency</span>
    <span>lane_pose</span>                         <span>15 ms </span>
    <span>belief_img</span>                       <span>2 ms </span>
    <span>entropy</span>                      <span>negligible</span>
    <span>in_lane</span>                  <span>negligible</span>
    <span>switch</span>               <span>negligible </span>
</col2>


In the following table, subscribed topics are listed:


<col2 align='center' style="text-align:left" id='lane_filter_subscribed_topics' figure-id="tab:lane_filter_subscribed_topics" figure-caption="Published topics by Lane Filter Node">
    <span>Topic</span>                                    <span>Max Latency</span>
    <span>segment_list</span>                         <span>25 ms</span>
    <span>velocity</span>                               <span>egligible</span>
    <span>car_cmd (not yet)</span>                      <span>negligible</span>
</col2>


Total latency from image taken, processed through anti-instagram, up until setting the motor control command is on average 140 ms.    

_Lane Controller Node_

In the following table, published topics are listed:


<col3 align='center' style="text-align:left" id='lane_controller_published_topics' figure-id="tab:lane_controller_published_topics" figure-caption="Published topics by Lane Controller Node">
<span>Topic</span> <span>Type</span> <span>Max Latency</span>
<span>car_cmd</span> <span>duckietown_msgs/Twist2DStamped</span>  <span>negligible</span>
</col3>

In the following table, subscribed topics are listed:


<col3 align='center' style="text-align:left" id='lane_controller_subscribed_topics' figure-id="tab:lane_controller_subscribed_topics" figure-caption="Subscribed topics by Lane Controller Node">
  <span>Topic</span>             <span>Type</span>            <span>Max Latency</span>
 <span>lane_pose</span>                  <span>duckietown_msgs/LanePose</span>       <span>15 ms</span> 
 <span>lane_pose_intersection_navigation</span>          <span>duckietown_msgs/ControlMessage_</span>       <span>20 ms</span>
<span>lane_pose_obstacle_avoidance</span>        <span>duckietown_msgs/ControlMessage_</span>                 <span>20 ms </span>
 <span>lane_pose_parking</span>                  <span>duckietown_msgs/ControlMessage_</span>       <span>25 ms </span>
<span>stop_line_reading</span>      <span>duckietown_msgs/StopLineReading</span>       <span> negligible </span>
<span>implicit_coordination_velocity</span> <span>duckietown_msgs/ControlVelocity</span> <span>negligible </span> 
    <span>Flags defined in table below</span>              <span>BoolStamped</span>            <span>negligible </span>
</col3>



**Flags received by other nodes**

These following flags are received from other modules. While one of these flags is _true_, the Duckiebot will behave according to the descriptions in the system architecture section.


 <col2 align='center' style="text-align:left" id='flags' figure-id="tab:flags" figure-caption="Flags received by other modules">
    <span>flag_at_stop_line</span>                         <span>_True_ when the distance to stop line is below a predefined distance.</span>
    <span>flag_stop_line_deteced</span>        <span>_True_ when number of detected red segments are above a threshold</span>
    <span>flag_at_intersection</span>                       <span>_True_ when at intersection. This flag is passed to us by the Parking team.</span>
    <span>flag_obstacle_detected</span>                  <span>_True_ when obstacle is in the lane. This flag is passed to us by the Saviors. </span>
    <span>flag_obstacle_emergency_stop</span>           <span>_True_ when it is not possible to avoid the obstacle without leaving the right lane. </span>
    <span>flag_at_parking_lot</span>           <span>_True_ when stopping at an intersection and the april tag for the parking lot is detected by the Parking team. </span>
    <span>flag_parking_stop</span>        <span>Per default = _true_. If _false_, the lane-follower will move along the given trajectory on the parking lot.</span>
    <span>flag_implicit_coordination</span>        <span>_True_ when implicit coordination is running, in this case we listen to the velocity published by them.</span>
 </col2>



**Structure of the received messages with type _duckietown_msgs/LanePose_**

The following table defines the structure of the pose messages the Lane Controller Node receives.


<col4 align='center' style="text-align:left" id='posemessage' figure-id="tab:posemessage" figure-caption="Structure of pose message to be used.">
<span>Field</span>      <span>Abstract Data Type</span>     <span>SI Units</span> <span>Description</span>
<span>header</span>     <span>Header</span>     <span>-</span>         <span>Header</span>
<span>d</span>         <span>float32</span>     <span>$m$</span>        <span>Estimated lateral offset</span>
<span>sigma_d</span>     <span>float32</span>     <span>$m$</span>        <span>Variance of lateral offset</span>
<span>phi</span>         <span>float32</span>     <span>$rad$</span>        <span>Estimated Heading error</span>
<span>sigma_phi</span>     <span>float32</span>     <span>$rad$</span>        <span>Variance of heading error</span>
<span>c</span>         <span>float32</span>     <span>$1/m$</span>        <span>Reference curvature</span>
<span>status</span>     <span>int32</span>         <span>-</span>        <span>Status of Duckiebot 0 if normal, 1 if error is encountered</span>
<span>in_lane</span>     <span>bool</span>         <span>-</span>    <span>In lane status</span>
 </col4>


**Structure of the received messages with type _duckietown_msgs/ControlMessage_**

The following table defines the structure of the control messages the Lane Controller Node receives from all the teams who want to send commands to our controller.


<col4 align='center' style="text-align:left" id='controlmessage' figure-id="tab:controlmessage" figure-caption="Structure of control message to be used.">
<span>Field</span>      <span>Abstract Data Type</span> <span>SI Units</span> <span>Description</span>
<span>header</span>   <span>Header</span>     <span>-</span>   <span>Header</span>
<span>d_est</span>   <span>float32</span>  <span>$m$</span>   <span>Estimated lateral offset</span>
<span>d_ref</span>   <span>float32</span> <span>$m$</span> <span>Reference lateral offset</span>
<span>phi_est</span> <span>float32</span> <span>$rad$</span> <span>Estimated Heading error</span>
<span>phi_ref</span> <span>float32</span> <span>$rad$</span>  <span>Reference heading</span>
<span>c_ref </span>  <span>float32</span> <span>$1/m$</span>  <span>Reference curvature</span>
<span>v_ref</span>  <span>float32</span>  <span>$m/s$</span>  <span>Reference Velocity</span>
</col4>


**Structure of the received messages with type _duckietown_msgs/ControlVelocity**

The following table defines the structure of the control messages that mostly the implicit coordination group will send us to control the velocity.


<col4 align='center' style="text-align:left" id='ControlVelocity' figure-id="tab:ControlVelocity" figure-caption="Structure of velocity message to be used.">
<span>Field</span> <span>Abstract Data Type</span> <span>SI Units</span> <span>Description</span>
<span>header</span> <span>Header</span>     <span>-</span> <span>Header</span>
<span>v_ref</span> <span>float32</span> <span>$m/s$</span><span>Reference Velocity</span>
</col4>


**Information to be provided from each team**

The following table defines the information we need from each team that uses the lane controller.


<col2 align='center' style="text-align:left" id='InfosNeeded' figure-id="tab:'InfosNeeded'" figure-caption="Information needed from each team.">
<span>Team</span> <span>Information</span>
<span>Saviors</span> <span>$d_{ref}$, $v_{ref}$</span>
<span>Navigators</span> <span>$d_{est}$, $d_{ref}$, $\theta_{est}$, $c_{ref}$, $v_{ref}$</span>
<span>Parking team</span> <span>$d_{est}$, $d_{ref}$, $\theta_{est}$, $c_{ref}$, $v_{ref}$</span>
<span>Implicit Coordination</span> <span>$v_{ref}$</span>
<span>Fleet-level Planning</span> <span>$d_{ref}$, $v_{ref}$</span>
 </col2>


_Stop Line Filter Node_

In the following table, published topics are listed:


<col2 align='center' style="text-align:left" id='stop_line_filter_node_published_topics' figure-id="tab:stop_line_filter_node_published_topics " figure-caption="Published topics by Stop Line Filter Node">
  <span>Topic</span> <span>Max Latency</span>
    <span>stop_line_reading</span>                         <span>negligible</span>
    <span>flag_at_stop_line</span>                       <span>negligible </span>
</col2>


In the following table, subscribed topics are listed:


<col2 align='center' style="text-align:left" id='stop_line_filter_node_subscribed_topics' figure-id="tab:stop_line_filter_noder_subscribed_topics" figure-caption="Published topics by Stop Line Filter Node"> 
<span>Topic</span> <span>Max Latency</span>
 <span>segment_list</span>  <span>25 ms</span>
 <span>lane_pose</span>  <span>15 ms</span>
</col2>



<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan

<!--Please note that for this part it is necessary for the VPs for Safety to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.-->

### Demo plan
<!--
The demo is a short activity that is used to show the desired functionality, and in particular the difference between how it worked before (or not worked) and how it works now after you have done your development.

It should take a few minutes maximum for setup and running the demo.
- How do you envision the demo?

Our demos should show that the implemented lane following module provides a better performance not only driving on straight and curved lanes but is in addition also more robust to geometric inconsistency such as lane width and line width.    
--> 

The main goal is to demonstrate the improved curve driving. For this purpose, we will run two Duckiebots with different versions of Estimator and Controller running on the same test track, see picture. With the old lane following module, the Duckiebot corrected its position when it was not parallel to the white lines and the correction caused an overshoot. The new module allows the Duckiebot to detect an upcoming curve early and the controller will be adjusted to the curve. Our module also improves the execution of other tasks such as stopping at an intersection or in front of a Duckie. Further, the Duckiebot will drive with an offset of at most 2 cm. While running in an endless loop, we can also show that the steady state error has been minimized.
 
<center><img figure-id="fig:demo_map" figure-caption="Possible map for lane following demo." src="demo_map.png" alt="Drawing" style="width: 400px;"/></center>





_What hardware components do you need?_    
For our demo we want to build a small Duckietown test track, see picture. Thus we need about 21 tiles, DUCKIEtape and the usual Duckietown decoration.

### Plan for formal performance evaluation

<!--
- How do you envision the performance evaluation? Is it experiments? Log analysis?    

In contrast with the demo, the formal performance evaluation can take more than a few minutes.

Ideally it should be possible to do this without human intervention, or with minimal human intervention, for both running the demo and checking the results.
-->

We will run all of the tests **5 times**. 

* **Stopping in front of red line**: in the demo mode, we will let the Duckiebot drive to a red line and measure the distance between the center of the stop line to the wheel axis of the Duckiebot after it stopped. 

* **Pose Estimation**: We will manually drive the Duckiebot along a pre-taped route in the Duckietown, of which the **d** is equal to zero, and collect multiple sets of data with the old pose estimator running and then with the new pose estimator running. From the bag data, we can analyse the estimation deviation from both estimators.

* **Offset minimization in straight lanes**: In the demo mode, we will let the Duckiebot drive down a straight lane. At the end of the straight lane, we will fix two laser pointers pointing to a wall and count how many times we can’t see the light point on the wall while driving. In the case, a light dot disappears, the Duckiebot has left the target range. The distance between the laser pointers will be the width of a Duckiebot plus 4 cm. 

* **Performance of the controller on curvy roads**: For curvy roads we will check the visual performance of the line following by counting how many times the Duckiebot touches a line on the S-curve section of the Zurich Duckietown. Additionally we want to compare the control motor commands in the curve section with the commands of the old controller and verify their smoothness.

* **Performance of the controller on lanes with dynamic width**: If we altered the controller to be more robust for non nominal appearance, we eventually check if the Duckiebot is robust to changes in lane specifications, such as narrower lanes or different width of lane tapes. We will let the Duckiebot drive on modified tiles and check the performance of estimation and lane following.

<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

<!--Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.-->

### Collection

_How much data do you need?_    
For every future step we need fixed logs and logs from driving. Baseline has been set and logs have been taken with the current implementation of the code to evaluate current performance.    

_How are the logs to be taken? (Manually, autonomously, etc.)_

* **Manually**: Static logs with different values for $d_{act}$ and $\theta_{act}$ have been taken. These can be used for a unit test of the estimator.




<center><img figure-id="fig:log_table" figure-caption="Table of static logs taken to evaluate the estimator." src="log_table.png" alt="Log Table" style="width: 400px;"/></center>

* **Autonomous**: Logs should also be taken during lane-following-demo to evaluate the estimator and control performance (see performance evaluation).






_Do you need extra help in collecting the data from the other teams?_    
We do not need data from other teams and therefore do not need help.



### Annotation

_Do you need to annotate the data?_    
No, because we will receive the needed edges from the Anti-Instagram group. 

_At this point, you should have tried using [thehive.ai](https://thehive.ai/) to do it. Did you?_    
In autonomous driving thehive.ai is mostly used to annotate images in order to detect and recognize obstacles and for semantic segmentation. As our project does not rely on these information, we do not need it.

_Are you sure they can do the annotations that you want?_    
Probably they could, but so do we with our estimator. There are no obstacles or Duckies to annotate.

### Analysis

We don’t need data annotation since we can do all the benchmarking by our own. We are not involved in any obstacle detection so we do not need any obstacles annotated.    

_Do you need to write some software to analyze the annotations?_    
No, because we do not use the annotations of thehive.ai.

_Are you planning for it?_    
No

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->


