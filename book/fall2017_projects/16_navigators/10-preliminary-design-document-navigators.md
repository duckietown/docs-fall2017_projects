# The Controllers {#part:navigators status=ready}

# Navigators: preliminary report {#navigators-pdd status=beta}


## Part 1: Mission and Scope

### Mission Statement

The objective of this project is to implement a method that allows Duckiebots to reliably navigate any kind of intersection they may encounter when driving through Duckietown.

### Motto


Motto: TRANSIBITIS <br/>(You shall pass)


### Project Scope

#### What is in scope


* Navigating three- and four-way intersection of predetermined shape.

* (Absolute or relative) localization within the intersection.

* Computing a path or trajectory that guides the Duckiebots across the intersection to the desired lane.

* Computing control inputs to follow path / track trajectory.

* Limiting travel time across intersection.

* Detecting when the Duckiebot successfully navigated across an intersection and finds itself in a regular lane.

* Proposing hardware modifications to the intersection (e.g. traffic lights, additional markers,...).


#### What is out of scope

* Understanding that the Duckiebot is at an intersection.

* Deciding where to go at an intersection.

* Coordinating with other Duckiebots at an intersection (e.g. who drives first, …).

* Object detection and collision avoidance.

* Global localization.


#### Stakeholders

* Smart Cities

* The Controllers

* Anti-Instagram

* The Identifiers

<div figure-id="fig:navigators-io2">
    <figcaption>Project boundary</figcaption>
    <img src="io2.png" style="width: 95%"/>
</div>


## Part 2: Definition of the Problem

### Problem Statement

We seek to find a method that allows a Duckiebot to safely navigate an intersection. In particular, we attempt to solve the following problem: Given that the Duckiebot finds itself at rest at an intersection, 1) devise a method such that the Duckiebot can localize itself at the intersection, 2) compute a path or trajectory that guides the Duckiebot to the desired exit while respecting the Duckiebots system dynamics and control input limitations, 3) find control inputs to track the path or trajectory, and lastly 4) detect when the maneuver is finished and the Duckiebot finds itself in a regular lane.

### Assumptions

* Size, shape of intersections is given and fixed.

* Color and size of lane markings are given and fixed.

* The type of intersection (e.g. three-way or four-way intersection) as well as the desired exit (e.g. left turn, right turn or straight) are provided.

* There are fiducial markers (e.g. April tags, stop signs, …) placed at the intersection at predetermined positions.

* The Duckiebots’ are initially at rest and their pose is within a certain range with respect to the intersection (distance to center of road, distance to stop line, orientation within the lane).

* The intersection is free of obstacles.

* Good light conditions (e.g. no illumination problems,...)

### Approach

The task of navigating an intersection can be roughly split into two tasks: 1) localization and 2) path/trajectory planning and control. The latter problem is assumed that be relatively easy compared to the first. Paths or trajectories can, for example, be found using simple motion primitives such as splines and it is then straightforward to track these using control techniques such as proportional-derivative-integral controllers. Hence, we only list different approaches to solve the localization problem when navigating intersections in the following.

**Open-Loop Maneuver:** Given that the Duckiebot finds itself at an intersection, it simply executes a predetermined trajectory (from the nominal starting position to the desired end position) to cross the intersection. Once the regular lane detection is able to estimate the Duckiebots pose again, the control is handed back to the lane following controller.
*Extension:* Model inaccuracies that are likely to affect the performance of the open-loop maneuvers could be compensated for by using iterative learning based on the state estimate of the lane detection method at the end of the open loop maneuver. If the lane detection algorithm finds that the duckiebot is far off of the nominal path, the nominal path could be adjusted iteratively such that systematic errors are suppressed (Note: This could to some extent interfere with the System Identification Project).

**April-Tags:** At each intersection in Duckietown, April-tags tell the Duckiebots at which intersection they are and what kind of intersection they need to cross (e.g. three-way intersection). The April-tags are placed at predetermined position at the intersection and are standardized. Given the (absolute) size of an april tag and the Duckiebot’s camera parameters, one can compute the Duckiebot’s position relative to the april tag and use this information to track a trajectory (again, relative to an april tag).
*Extension:* The environment in Duckietown is very structured and various different objects, e.g. traffic lights or street signs, are placed at predetermined locations. To increase robustness of this method or to handle the case when the April tags are out of sight of the Duckiebots (likely towards the end of crossing an intersection), these objects could also be used for localization.

**Line Localization:** The disadvantage of using fiducial markers for localization is that it is not applicable to the real world where there are no fiducial markers. However, also there also exist in the real world standardized marks, the line marks on the street. One possible approach could thus be the detection of all the line marks visible at intersection (e.g. stop marks of current lane but also of the different exits) and use this for localization. Since the appearance of intersections are standardized in Duckietown, the homography that best explains the detected line marks could be determined in order to localize the Duckiebot.

**Visual Odometry:** The most generic solution would be the use of visual odometry for the localization of the Duckiebot at the intersection, i.e. matching of distinct features between subsequent camera frames and using these point matches to compute the relative camera pose between the two frames. Visual odometry in in general computationally expensive. However, the Duckiebot’s dynamic could be used to reduce the computational burden (e.g. 1-point RANSAC). Due to a lack of sensors on the Duckiebot, visual odometry is unable to determine the scale of the scene. To solve this, the information about e.g. the size of April-Tags, line width, etc. can be used.

The above methods can also be combined, e.g. using visual odometry and if available April-tags.


### Functionality-resources Trade-offs

### Functionality provided

* Localization within an intersection.

* Path/trajectory planning to navigate intersection.

* Possibly control strategy to track above.

### Resources required / dependencies / costs

The proposed method(s) can be evaluated using the following measures (in order of decreasing importance):

* Success rate, i.e. the percentage of trials for which the Duckiebot ends up in the desired lane and successfully hands over the control to the lane following controller.

* Accuracy and precision of  final state, i.e. how close is the Duckiebot’s state relative to the desired final state (e.g. distance relative to the center of the lane, orientation within lane, velocity) and how repeatable is this.

* Duration, i.e. the average time required for the Duckiebot to cross an intersection and if possible an upper limit (worst-case) on the time required.

* Robustness to changes in the size of the intersection (e.g. lane width, size of tiles, …), i.e. how do the above performance measures change with respect to changes of the size of intersection.

* Path following or trajectory tracking error, using for example the maximum absolute distance error from the desired path.


### Performance measurement

* The success rate can evaluated by simply performing the intersection navigation tasks N times and counting the number of successful trials.

* The accuracy and precision of final state can be evaluated using the already existing lane detection method.

* The average duration can simply be computed from N experiments. A (probabilistic) upper bound to navigating an intersection can be found likewise.

* Either experiments on the real system with slightly different intersections or a sensitivity analysis (analytically or numerically) with respect to parameters defining the size of the intersection can be carried to evaluate the system’s robustness.

* An absolute position system (e.g. an overhead motion capture system) can be used to evaluate the Duckiebot's trajectory tracking errors.

## Part 3: Preliminary Design

### Modules

* Initial position understanding

* Information processing (in which kind of intersection we are, where we want to go, ..)

* Trajectory generation

* Control loop

* Conclusion


### Interfaces

The inputs and outputs are also shown on a very high level in the Stakeholders Graph.

### Preliminary plan of deliverables

### Software modules

* Intersection Localization: ROS node

* Path Planner / Trajectory Generator: Python library

* Path Following / Trajectory Tracking Controller: ROS node.


### Infrastructure modules

None (assuming that no hardware changes are required).


## Part 4: Project Planning

### First Steps for the Next Phase

### Data collection

Recordings of the camera feed of Duckiebots navigating intersections.

### Data annotation

No

#### Relevant Duckietown resources to investigate

* Current (open-loop) solution
* April tag detector
* Control strategy
* Lane Detector
* Anti-Instagram
* State Machine (switch between lane-following and navigating intersection)
* Visual Odometry
* Duckiebot system dynamics and control input constraints


#### Other relevant resources to investigate

* April Tags:
https://april.eecs.umich.edu/software/apriltag.html

* Aruco Markers:
https://sourceforge.net/projects/aruco/files/

* Visual Odometry:
D. Scaramuzza, F. Fraundorfer, “Visual odometry [tutorial]”, IEEE Robotics &amp; Automation Magazine, 2011.

D. Scaramuzza, F. Fraundorfer, R. Siegwart, “Real-time monocular visual odometry for on-road vehicles with 1-point RANSAC”, IEEE International Conference on Robotics and Automation, 2009.

* Localization using Line Detection:
J. Barandiaran, D.Borro, “Edge-Based Markerless 3D Tracking of Rigid Objects”, IEEE Conference on Artificial Reality and Telexistence, 2007.


### Risk Analysis

* Not enough distinct features for visual odometry.
* Similarly, not enough lane markings.
* Not sufficient computational power on board the Duckiebot.

#### Mitigation strategies

Mainly the mentioned risks will be related to a closed-loop implementation, which is a basic goal of the project.
However they should not affect the development of an improved open-loop solution, ex. using April Tags detection, so this may be a starting point to improve the current state and to start the  development of the closed-loop solution.

For the application of a Visual Odometry algorithm, if features in the Duckietown will be not enough, we may plan to add/use additional hardware, which will be possibly developed in accord with the Smart CIties group.

Moreover, the paper “Real-time monocular visual odometry for on-road vehicles with 1-point RANSAC” may be useful for addressing issues about computational constraints thanks to the implementation of the 1-point RANSAC algorithm.
