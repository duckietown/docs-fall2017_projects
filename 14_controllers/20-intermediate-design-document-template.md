#  The Controllers: Intermediate Report {#template-int-report status=ready}

<!--
_It's time to commit on what you are building, and to make sure that it fits with everything else._

This consists of 3 parts:

- Part 1: System interfaces: Does your piece fit with everything else? You will have to convince both system architect and software architect and they must sign-off on this.

- Part 2: Demo and evaluation plan: Do you have a credible plan for evaluating what you are building? You will have to convince the VPs of Safety and they must sign-off on this.

- Part 3: Data collection, annotation, and analysis: Do you have a credible plan for collecting, annotating and analyzing the data? You will have to convince the data czars and they must sign-off on this.
-->


<div markdown="1">

 <col2 id='checkoff-people-intermediate-report' figure-id="tab:checkoff-people-intermediate-report" figure-caption="Intermediate Report Supervisors">
    <s>System Architects</s>                         <s>Sonja Brits, Andrea Censi</s>
    <s>Software Architects</s>                       <s>Breandan Considine, Liam Paull</s>
    <s>Vice President of Safety</s>                  <s>Miguel de la Iglesia, Jacopo Tani</s>
    <s>Data Czars</s>                                <s>Manfred Diaz, Jonathan Aresenault</s>
 </col2>

</div>

## Part 1: System interfaces

<!--
Please note that for this part it is necessary for the system architect and software architect to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.
-->

### Logical architecture

<!--
- Please describe in detail what the desired functionality will be. What will happen when we click "start"?


We assume that the Duckiebot is placed on the right lane of the roadwithin a defined boundary for the initial pose. By starting the lane-follower it should begin to follow its lane, whether it is straight or curved, until we pressclick “stop”. The lane following module consistsconsisting of two parts, an estimation of the current position and a controller. 

Additionally, the Duckiebot will stop between 6 to 0 cm (given by the Explicit Coordination Team) from the red line in order to enable the navigation through the intersection. The navigation through an intersection is out of scope. The Lane Following module will start again after the intersection, triggered by the Finite state machine. 

We are receiving a d (distance to the middle of the right lane) from the Saviors Team, in case there is an obstacle on the lane. The Saviors Team will compensate the delay and send us the d 30-40 cm from obstacle in advance. Out of scope is the controlled obstacle avoidance involving leaving the right lane. This case is determined by a flag sent from the Saviors module and the Duckiebot will stop. 
There is also a stop flag received from the stop_line_filter_node. We will introduce priorities to the several flags received to decide how to duckiebot should behave.

Parking
When the parking flag is active, the controller node stops running until the flag is inactive

- Please describe for each quantity, what are reasonable target values. (The system architect will verify that these need to be coherent with others.)

The Duckiebot should only run at a maximum velocity of x m/s for optimal controllability. The reason for the limited velocity is the low image update frequency which limits our pose estimation update. Hence a lower velocity enhances the performance of our controlled lane following. Image update too slow hence pose estimation limited=> Slower velocity better pose trajectory. Since not every Duckiebot has the same gain set, we will pass the desired velocity to team System Identification. Their module will convert the demanded velocity to the according input voltage for the motor.  

Whenever we detect a red line, we will slow down the duckiebot. 

Intersection 0-6cm , +-10° 

Savior: not leaving lane, not hitting duck

- Please describe any assumption you might have about the other modules, that must be verified for you to provide the functionality above.
-->

Savior: d enough in advance They will set the obstacle detected flag 20-30cm in front of the obstacle, so we have enough time to avoid the obstacle.
System id : motor control we know nothing yet. => define gain? 
Anti-instagram : has tuned bla very good. Edges superb


<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

- _Lane Filter Node_ :
  - Published topics:
    - lane_pose
      - Maximum latency to be introduced: ??
    - belief_img
      - Maximum latency to be introduced: ??
    - entropy
      - Maximum latency to be introduced: ??
    - in_lane
      - Maximum latency to be introduced: ??
    - switch
      - Maximum latency to be introduced: ??
  - Subscribed topics:
    - segment_list
      - Latency expected: 143 ms
    - velocity
      - Latency expected: 0 ms
    - car_cmd (not yet)
      - Latency expected: [runtime of Lane Controller node]
    - flags_from_other_teams

Total latency from image taken processed through anti-instagram etc to us  70ms (source?)

- _Lane Controller Node_ :
  - Published topic:
    - car_cmd
      - Maximum latency to be introduced: ??
  - Subscribed topic: 
    - lane_pose
      - Latency expected: [runtime of Lane Filter Node]

<!--
- Please describe the list of nodes that you are developing or modifying.

- For each node, list the published and subscribed topics.

- For each subscribed topic, describe the assumption about the latency introduced by the previous modules.

- For each published topic, describe the maximum latency that you will introduce.
-->

<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan


_Please note that for this part it is necessary for the VPs for Safety to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Demo plan

The demo is a short activity that is used to show the desired functionality, and in particular the difference between how it worked before (or not worked) and how it works now after you have done your development.

It should take a few minutes maximum for setup and running the demo.

Proposition 1:
Use 2 duckiebots, one with old lane following code and one with the new one to show performance improvement.

Use hard to follow curvature in which the current implementation struggles to follow the lane.



Proposition 2:
Show it is able to follow the lane even when width is changing or sometimes the white line is missing (is our estimator able to get the correct position?)
Maybe special demo tiles?


- How do you envision the demo?
  - We want to make a demo in which we let two duckiebot run simultaneously on test track (closed loop track) “chasing each other”.
    - Point out faster decay of transient error
    - Point out smaller steady state error

- What hardware components do you need?
    - We need to build a small Duckietown test track: thus we need about 15 tiles, DUCKIEtape and the usual Duckietown decoration.

### Plan for formal performance evaluation

- How do you envision the performance evaluation? Is it experiments? Log analysis?

In contrast with the demo, the formal performance evaluation can take more than a few minutes.

Ideally it should be possible to do this without human intervention, or with minimal human intervention, for both running the demo and checking the results.

Performance evaluation by several benchmarking tests tbd yet.
Suggestions:

Performance: average or median deviation from middle of the lane (or given d) to follow. Summed up error must be as small as possible → maybe add graphic to show how we are going to measure it.

Use same estimator and implement old and new controller to see performance difference.
Use same controller but different estimator to evaluate performance gain.

Count amount of times hitting white or yellow lines (ideal case should be 0).

Count amount of times getting onto the wrong lane.

<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

_Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Collection

- How much data do you need?
For every feature step we need fixed logs and logs from driving. Baseline has been set and logs have been taken with the current implementation of the code to evaluate current performance.

- How are the logs to be taken? (Manually, autonomously, etc.)
Manually
Andy and simon have already recorded some logs
Table with log specification d , theta, curve on line

Autonomous
Logs should also be taken during lane-following-demo to evaluate the estimator & control performance → see performance evaluation

Describe any other special arrangements.

- Do you need extra help in collecting the data from the other teams?



### Annotation

- Do you need to annotate the data? No, because we will receive the needed edges from the Anti-Instagram group. 

- At this point, you should have you tried using [thehive.ai](https://thehive.ai/) to do it. Did you?
Check out thehive.ai and write down why we don’t need to use it.

- Are you sure they can do the annotations that you want?
Probably they could, but so do we with our estimator. There are no obstacles or duckies to annotate.

### Analysis

We don’t need data annotation since we can all the benchmarking by our own. We are not involved in any obstacle detection so we do not need any obstacles annotated.
- Do you need to write some software to analyze the annotations?

- Are you planning for it?

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->

