#  Implicit Coordination: preliminary design document {#Formation-and-Implicit-Control-preliminary-design-doc status=draft}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

Formation keeping and collision avoidance using implicit communication

### Motto

AlIIS VIVERE

### Project scope

#### What is in scope


* Relative pose estimation

* Formation keeping: formalize controller that keeps constant distance from vehicle ahead

* "Follow the leader demo"

* Avoiding traffic waves

* Intersection coordination

 * **Detection**: position (bounding box) + pose estimation per bot in FoV.

 * **Tracking**: trajectory (sequence of pose) estimation per bot tracked, parametrized in time.

 * **Prediction**: predict bots behavior through intersection by integrating tracking information and rules of the road (no explicit communication allowed).

  * Implement traffic rules

  * Intention of the other car has to be predicted

  * Priority traffic rule: First come first serve. Crossings possible without explicit communication

  * Designing Fiducial markers (April tags)

* (Designing a special T intersection with only one stop)


#### What is out of scope
* Adding hardware
* Intersection navigation
* Communication with LEDs / explicit communication



#### Stakeholders


* System Architect: Sonja Brits
* Software Architect: Breandan Considine
* Knowledge Tzarina: Lucy
* Anti-Instagram: Christoph Zuidema, Milan Schilling, David Yunis, Shenjie Lin?
* Controllers: Marco Stalder, Simon Muntwiler, Anna Dai, Manuel Breitenstein, Andreas Aumiller?
* Navigators: Theodore Koutros, Flin Höpflinger, Giavanni Cioffi, Dario Brescianini?
* Explicit Coordinators: Robin Deuber, Valentina Cavinato, Nicolas Lanzetti, Giulia Zobrist, Gioele Zardini?



## Part 2: Definition of the problem

### Problem statement

**Mission:** Formation keeping and intersection navigation with Duckiebots just using implicit communication.

**Problem statement:** Detect other Duckiebots, follow the leader in a secure distance through Duckietown and coordinate intersections using implicit communication.
### Assumptions

* Duckiebots do not use explicit communication, e.g. LEDs, WLAN ...
* Duckiebots have different appearance (different configuaration, LEDs on/off, ...)
* All Duckiebots are autonomous, not remote controlled
* All Duckiebots use the same formation and implicit control algorithm


### Approach

###### Formation
* Vehicle detection
 * April tags
   * Analyze already existing code for vehicle detection ([Software](https://github.com/duckietown/Software/tree/UROP_Perception_Chandan/catkin_ws/src/vehicle_detection_tracking), [Duckumentation](https://docs.google.com/document/d/1enqjECNa-1_OZIZvHZekZ1tp872uYohSdGUyRf0s9BA/edit))
 * CNN-based
   * Image annotation        
* Tracking
 * model based
 * learning based
* Prediction
 * model based
 * learning based
* Distance Control

###### Implicit Coordination

* Data Collection

 * Annotation tool: if only detecting bots, any custom manual approach will work, if not, then we will use thehive.ai API.

 * Types of intersections: 4-way, 3-way, 3-way with 1 stop sign.

 * No pedestrians.

 * Motions: moving vs. static traffic, moving vs. static self.

 * Including “look-around” behavior.

 * Illumination variances: add a predominant light to Duckietown so we can have strong shadows and different lighting conditions.

 * Appearance variances: different bots configurations (e.g: with/without the shell, a duck, etc.)

 * Frames will be extracted from videos at a 3 fps framerate.

* Detection

 * Instance-level 2D/3D Bounding box + pose estimation.
 * Explore OpenCV built-in detection capabilities.

 * Explore supervised learning methods for detection, including existing models (e.g: YOLO2, SDD, etc).
* Tracking
 * Estimate prior trajectory of each bot based on the instance level detection generated.
 * Explore OpenCV tracking algorithms ([Doc](https://www.learnopencv.com/object-tracking-using-opencv-cpp-python/))
 * Explore MOT algorithms based on Deep Learning ([Paper]( https://arxiv.org/pdf/1708.02843.pdf))
* Prediction
 * 3D bounding box + orientation + velocity + Position prediction (1, 5, 20) + estimating a policy ("behavior") applying rules of the road.
 * APPROACH: TBD.

### Functionality-resources trade-offs

#### Functionality provided

Formation keeping and intersection coordination

* Detection of other Duckiebots within field-of-view
* Tracking
* Prediction

#### Resources required / dependencies / costs

* Multiple duckiebots
* Duckietown
* April tags
* Annotated data set


#### Performance measurement

* Robustness of Formation keeping and Vehicle Detection
 * Number of duckiebots in the “Follow the leader” demo
 * Duration of the demo [time]
 * Transient error of distance and perpendicular offset
 * Vehicle Detection ROC-Curve
 * Uncertainty in pose estimation
 * Tracking Error / Prediction Accuracy
 * Robustness to data annotation errors
* Intersection coordination
 * Throughput rate of duckiebots through intersection
 * Waiting time at intersection
 * Number of successful crossings at intersection before a collision happens
 * Vehicle Detection ROC-Curve
 * Uncertainty in pose estimation
 * Tracking Error / Prediction Accuracy
 * Robustness to data annotation errors
 * Robustness to duckiebot pose, appearance, scale, orientation ambiguity and LED-light


## Part 3: Preliminary design

### Modules

* Detection of Duckiebot
* Estimation of relative pose
* Controller for following the leader
* Intersection behavior


### Interfaces

* Detection of Duckiebot     
 * Input: April tag
 * Output: Position in formation and duckiebot parameters
* Estimation of relative pose     
 * Input: Camera image
 * Output: Estimate of Relative pose between bots
* Controller:             
 * Input: Estimated pose
 * Output: Command to motors
* Intersection behavior:        
 * Input: State at stopping line
 * Output: Preferences at intersection


### Preliminary plan of deliverables



#### Specifications

T-intersection in Duckietown?

#### Software modules

* Detection / Estimation node
* Tracking node
* Controller node
* Implicit coordination behavior node


#### Infrastructure modules

None


## Part 4: Project planning



### Data collection



### Data annotation



#### Relevant Duckietown resources to investigate
Exisiting intersection safty rules ([Duckumentation](https://docs.google.com/document/d/1lNTIPN5HFunX06IU5tAq_964eOHSSKyTbdd9nVGhb-A/edit#heading=h.gjdgxs))  
Existing vehicle detection algorithm ([Code](https://github.com/duckietown/Software/tree/UROP_Perception_Chandan/catkin_ws/src/vehicle_detection_tracking), [Duckumentation](https://docs.google.com/document/d/1enqjECNa-1_OZIZvHZekZ1tp872uYohSdGUyRf0s9BA/edit))


#### Other relevant resources to investigate
Robotics Handbook: Ideas for controller for 'follow-the-leader' or driving in traffic on pg. 808

### Risk analysis

Likelihood from 1-10, where 10 is very likely and 1 very unlikely.
Impact from 1-10, where 10 is a huge negative impact and 1 not so bad.

Event
Likelihood
Impact
Risk response Strategy (avoid, transfer, mitigate, acceptance)
Actions required
False Negative in Vehicle detection can lead to crash
4
9
mitigate
The effect of FPs are less bad than those of FNs. Punish FNs more in an eventual cost function.
False Positive in Vehicle detection makes bot stand still.
4
2
accept


Inaccurate estimation of distance between bots
3
7
mitigate
Early testing, big enough safety margin, improve controller
Collision of 2 duckiebots in an intersection
3+4
9
mitigate
Combination of Risk 1 and 3
