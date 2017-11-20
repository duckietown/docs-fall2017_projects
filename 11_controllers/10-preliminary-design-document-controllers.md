#  The Controllers: preliminary design document {#project-name-preliminary-design-doc status=ready}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

Make lane following more robust to model assumptions and Duckietown geometric specification violations and provide control for a different reference control. 

### Motto

<div class='check' markdown="1">

IMPERIUM ET POTESTAS EST (with control comes power)


</div>

### Project scope


#### What is in scope  

* Control Duckiebot on straight lane segments and curved lane segments.

* Robustness to geometry (width of lane, width of lines)

* Detection and stopping at red (stop) lines

* Providing control for a given reference **d** for avoidance and intersections (but for intersections, we additionally need the pose estimation and a curvature from the navigators team)  

#### What is out of scope  

* Pose estimation and curvature on Intersections (plus navigation / coordination)

* Model of Duckiebot and uncertainty quantification of parameters (System Identification) 

* Object avoidance involving going to the left lane

* Extraction and classification of edges from images (anti-instagram)

* Any hardware design

* Controller for Custom maneuvers (e.g. Parking, Special intersection control)

* Robustness to non existing line  



#### Stakeholders  

**System Architect**  

She helps us to interact with other groups. We talk with her if we change our project.   
  
**Software Architect**  

They give us Software guidelines to follow.   
They give a message definition.  
  
**Knowledge Tzarina**  
Duckiebook  
  
**Anti-Instagram**  
They provide classified edges (differentiation of centerline, outer lines and stop lines)  

Direction of the edges (background to line vs. line to background)  
  
**Intersection Coordination (Navigators)**   
They tell where to stop at red line.  
We give a message once stopped.  
They give pose estimation and curvature (constant) to navigate on intersection.  
We provide controller for straight line or standard curves.   
  
**Parking**  
They tell where to stop at red line.  
We give a message once stopped  
  
**System Identification**  
They provide model of Duckiebot.  
  
**Obstacle Avoidance Pipelines (Saviors)**  
They provide reference **d**.  
  
**SLAM**  
They might want to know some information from our pose estimation (e.g. lane width or theta).  
  


## Part 2: Definition of the problem

### Problem statement  

We must keep the Duckiebots at a given distance d from center of the lane, on straight and curved roads, under bounded variations of the city geometric specifications.    
    
Geometric specifications:   

* Nominal lane width
* Tape width (white, yellow, red)
* Spacing between yellow dashed line
* Curvature of the curves  

Given at every time step a reference **d** (the reference $\theta$ gets choosen in a way to match the reference d):  
\begin{equation}
    q_r(t)=[x_r(t),y_r(t),\theta_r(t)]^t \rightarrow [d_r(t),\theta_r(t)]
\end{equation}

The following image shows the definition of the parameters $d$ and $\theta$.

![image](problem_statement.svg)


($d$: distance perpendicular to the lane. 0 is defined in the center of the lane (see definition in duckumentation))  

($\theta$: angle between the lane and the robot body frame. )     
  
and given a **model of the system**,  
  
define a control action:  
  
* the heading and velocity of the center between the wheels of robot, leading to a sequence of motor commands
  
at every time step, such that the pose (estimate of the pose) converges to the target.  
  
**Performance:**  

* Steady state within a tile
* Never leaves the lane
* Small steady state error  
  
**Robustness to slight changes in:**  

* Model parameters
* Width of the lane
* Width of the lines
* Curvature of the road  
  

### Assumptions

* There is a reason for the caster wheel
* A system model of the Duckiebot is provided
* The system model parameters for every duckiebot are given
* A set of classified lane edges are given with a certain frequency, latency, resolution and a maximum number of false positives and maximum number of misclassifications.
* Anti-instagram people do low level vision -> extract lines in image space
* Only small deviation from the specified geometric values
* Surface properties of Duckietown tiles are similar in each Duckietown
* Bounded initial pose, when driving straight no wheel of the Duckiebot should touch any line within 25cm


### Approach

* Benchmark actual system → identify bottlenecks (in estimation and control)
- Identify bottlenecks by modifying different parts of the system and adding them each at a time and have a look at how much is the improvement
→ Make new branches for each of those modifications
* We want to improve the pose estimation by applying a particle filter. To measure the impact of the improved pose estimation, we will design a test procedure.
* One possible test procedure: Set a calibrated duckiebot to many known points on straight lanes and curved lanes. Save the information of these actual poses (measured by hand) together with the images taken by the Duckiebot’s camera in the respective poses. On this data, different anti-instagram methods and different pose estimations can be run and evaluated directly, without any physical duckiebot nor Duckietown.
* We want to improve the parameters of the current controller by tuning it experimentally.
* We want to increase the frequency of the controller update.
* We want to handle actuator saturation, if we adding an I part to the controller.  


### Functionality-resources trade-offs  


### Functionality provided

Drive on straight lane and curves without large deviations from the center of the lane.  

### Resources required / dependencies / costs

Hardware resources:  

* Tapes for lanes 
* Tiles to make different straight lanes and curved lanes
* Timer 
* Functional Duckiebot
  
Dependencies: see assumptions  

We assume to have image space line segments extracted and classified from images.

* frequency
* latency
* accuracy (resolution)
* maximum false positives
* maximum misclassification error (confusion matrix)


### Performance measurement

Drive on the track for one minute and count the number of times the bot touches the side or center line. Repeat this 5 times.   

**Metrics**  
Error from the reference distance d when driving straight.  
- Mean and variance of 5 experiments 
Estimate of lane width.  
- Estimate lane width and compare to measurement
Estimate road curvature.  
- Estimate curvature and compare to measured radius of curve
Speed - make is a control variable.  
Robustness to initial pose.  
- Run lane following using 5 different initial poses
Transient error after curved section (e.g. dies in one tile length).  
- 5 experiments of measuring the error d when driving straight after a curved segment $\rightarrow$ Did the transient error die?  
Robustness to the curvature.  
- Run curve following on 5 lanes made of different combinations of curve tiles (left-left-right, left-right-left, … )
Robustness to lane specifications   
- Run lane following on 5 lanes with different lane width when driving straight


## Part 3: Preliminary design

### Modules

Estimation of Position:

* Input: segments detected by Anti-Instagram-Filter
* Output: 
- Distance from center of lane, 
- Heading angle, 
- Curve or straight lane
- Curvature 

Controller:   

* Input: state (Distance from center of lane, heading angle, other) by an Estimator
* Output: Control Output to motors


### Interfaces

* Anti-instagram: Labelled (centerline, outer line, stop line) edges with color and direction in image plane. The messages is defined already.
* Odometry calibration: get the model parameters. Yaml file
* Obstacle avoidance: get reference distance d from center of lane.. Zero assumed otherwise.
* Estimator: state vector at regular intervals.
* Control: Motor voltages at regular intervals


### Preliminary plan of deliverables


### Specifications

We do not need to change the Duckietown specifications.

### Software modules

* Estimator: NODE. There is a markovian approach. A particle filter should be implemented
* Controller: NODE. there is a P controller. A feedforward should be implemented. Pure-pursuit, or simple FeedForward.
* Outlier rejection: NODE (or part of estimator). there is nothing. A system has to be designed to detect edges that clearly don’t belong to the line.
* Automated testing: NODE. There is nothing. A system should be implemented to test estimation from recorded data. It should be easy to update this data.


## Part 4: Project planning

### Timeline

| Date | Task Name | Target Deliverables |
|:-----------|:------------|:-------------|
| 17/11/17 | Kick-Off | Preliminary Design Document |
| 24/11/17 | Get familiar with state of the art | Benchmark state of the art, Identifiy bottlenecks |
| 01/12/17 | Theoretical derivation of Controller and Estimator | |
| 08/12/17 | Implementation of Controller and Estimator | |
| 15/12/17 | Benchmark new implementation | Performance measure of new implementation |
| 22/12/17 | Buffer | |
| 29/12/17 | Documentation | Duckuments |
| 05/01/18 | | End of Project |


### Data collection

Take rosbag logs every time.  

Rosbag:  

- Image
- Edges from Anti-Instagram
- Motor control values


### Data annotation

Curvature of road.

#### Relevant Duckietown resources to investigate

Vision odometry
Lane detection
Anti instagram

#### Other relevant resources to investigate

[Particle Filter coded in python and useful intro to the subject](https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python/blob/master/12-Particle-Filters.ipynb)  


### Risk analysis

| Risk | Likelihood | Impact | Risk response Strategy | Actions required |
| :--- | :--- | :--- | :--- | :--- |
| Cannot estimate curvature | 2 | 5 | mitigate | Start early with testing thresholds for curvature identification |
| Cannot define distance to curve | 2 | 4 | mitigate | Try various methods to identify the distance to curve |
| Duckiebot leaves the lane after curve (current situation) | 2 | 5 | mitigate | |
| Cannot handle the inputs given by other teams | 4 | 4 | mitigate | Get more information from Sonja, Talk to other teams, Clear comments in the code for easier problem detection |
