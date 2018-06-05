# Implicit Coordination {#part:implicit-coordination status=ready}

# Implicit Coordination: preliminary report  {#formation-and-implicit-control-pdd status=beta}


## Part 1: Mission and scope

### Mission statement

Formation keeping and collision avoidance using implicit communication

### Motto


Motto: ALIIS VIVERE <br/> (Live for others)


### Project scope

#### What is in scope

- Relative pose estimation
- Formation keeping: formalize controller that keeps constant distance from vehicle ahead
- "Follow the leader demo"
- Avoiding traffic waves
- Intersection coordination
    - **Detection**: position (bounding box) + pose estimation per bot in FoV.
    - **Tracking**: trajectory (sequence of pose) estimation per bot tracked, parametrized in time.
    - **Prediction**: predict bots behavior through intersection by integrating tracking information and rules of the road (no explicit communication allowed).
    - Implement traffic rules
    - Intention of the other car has to be predicted
    - Priority traffic rule: First come first serve. Crossings possible without explicit communication
    - Designing Fiducial markers (April tags)
- (Designing a special T intersection with only one stop)


#### What is out of scope
- Adding hardware
- Intersection navigation
- Communication with LEDs / explicit communication



#### Stakeholders

- System Architect
- Software Architect
- Knowledge Tzarina
- Anti-Instagram
- Controllers
- Navigators
- Explicit Coordinators

## Part 2: Definition of the problem

### Problem statement

**Mission:** Formation keeping and intersection navigation with Duckiebots just using implicit communication.

**Problem statement:** Detect other Duckiebots, follow the leader in a secure distance through Duckietown and coordinate intersections using implicit communication.

### Assumptions

- Duckiebots do not use explicit communication, e.g. LEDs, WLAN ...
- Duckiebots have different appearance (different configuaration, LEDs on/off, ...)
- All Duckiebots are autonomous, not remote controlled
- All Duckiebots use the same formation and implicit control algorithm


### Approach

Formation:

- Vehicle detection
    - April tags
        - Analyze already existing code for vehicle detection ([Software](https://github.com/duckietown/Software/tree/UROP_Perception_Chandan/catkin_ws/src/vehicle_detection_tracking), [Duckumentation](https://docs.google.com/document/d/1enqjECNa-1_OZIZvHZekZ1tp872uYohSdGUyRf0s9BA/edit))
    - CNN-based
        - Image annotation
- Tracking
     - model based
     - learning based
- Prediction
     - model based
     - learning based
- Distance Control

Implicit Coordination

- Data Collection
    - Annotation tool: if only detecting bots, any custom manual approach will work, if not, then we will use thehive.ai API.
    - Types of intersections: 4-way, 3-way, 3-way with 1 stop sign.
    - No pedestrians.
    - Motions: moving vs. static traffic, moving vs. static self.
    - Including “look-around” behavior.
    - Illumination variances: add a predominant light to Duckietown so we can have strong shadows and different lighting conditions.
    - Appearance variances: different bots configurations (e.g: with/without the shell, a duck, etc.)
    - Frames will be extracted from videos at a 3 fps framerate.

- Detection
    -Instance-level 2D/3D Bounding box + pose estimation.
    - Explore OpenCV built-in detection capabilities.
    - Explore supervised learning methods for detection, including existing models (e.g: YOLO2, SDD, etc).
- Tracking
    - Estimate prior trajectory of each bot based on the instance level detection generated.
    - Explore OpenCV tracking algorithms ([Doc](https://www.learnopencv.com/object-tracking-using-opencv-cpp-python/))
    - Explore MOT algorithms based on Deep Learning ([Paper]( https://arxiv.org/pdf/1708.02843.pdf))
- Prediction
    - 3D bounding box + orientation + velocity + Position prediction (1, 5, 20) + estimating a policy ("behavior") applying rules of the road.
    - APPROACH: TBD.

### Functionality-resources trade-offs

#### Functionality provided

Formation keeping and intersection coordination

- Detection of other Duckiebots within field-of-view
- Tracking
- Prediction

#### Resources required / dependencies / costs

- Multiple duckiebots
- Duckietown
- April tags



#### Performance measurement

Robustness of Formation keeping and Vehicle Detection

- Number of duckiebots in the “Follow the leader” demo
- Duration of the demo (time)
- Transient error of distance and perpendicular offset

Implicit coordination

- **Detection**: bounding-box (intersection over union, precision + recall), uncertainty in pose estimation
- Throughput rate of duckiebots through intersection
- Waiting time at intersection
Number of successful crossings at intersection before a collision happens
- Robustness to duckiebot pose, appearance, scale, orientation ambiguity and LED-lightning



## Part 3: Preliminary design

### Modules

- Detection of Duckiebot
- Estimation of relative pose
- Controller for following the leader
- Intersection behavior


### Interfaces

Detection of Duckiebot

- Input: April tag
- Output: Position in formation and duckiebot parameters

Estimation of relative pose

- Input: Camera image
- Output: Estimate of Relative pose between bots

Controller

- Input: Estimated pose
- Output: Command to motors

Intersection behavior

- Input: State at stopping line
- Output: Preferences at intersection


### Preliminary plan of deliverables



#### Specifications

(Special T-intersection with only one stop in Duckietown)

#### Software modules

- Detection / Estimation node
- Tracking node
- Controller node
- Implicit coordination behavior node


#### Infrastructure modules

None


## Part 4: Project planning

<div markdown="1">
 <col3 figure-id="tab:dates" figure-caption="Schedule">
    <span>**Date**</span>
    <span>**Task Name**</span>
    <span>**Target Deliverables**</span>
    <span>17/11/2017</span>
    <span>Kick-Off</span>
    <span>Preliminary Design Document</span>
    <span>24/11/2017</span>
    <span>Review state of the art, theoretical formalisation</span>
    <span>List of what has to be implemented and how and what can be reused from last year. Benchmarking current state</span>
    <span>1/12/2017</span>
    <span>Implementation</span>
    <span>-</span>
    <span>8/12/2017</span>
    <span>Implementation and Testing</span>
    <span>Code</span>
    <span>15/12/2017</span>
    <span>Evaluation, Optimization</span>
    <span>Performance measurement</span>
    <span>22/12/2017</span>
    <span>Buffer</span>
    <span>-</span>
    <span>29/12/2017</span>
    <span>Duckumentation</span>
    <span>Duckuments</span>
    <span>05/01/2018</span>
    <span>End of project</span>
    <span>-</span>
 </col3>
</div>

End of project



### Data collection

Annotated images

### Data annotation
Duckiebots in bounded boxes under different conditions (Illumination, LEDs on/off, Duckiebot configurations, ...)


#### Relevant Duckietown resources to investigate

Exisiting intersection safty rules ([Duckumentation](https://docs.google.com/document/d/1lNTIPN5HFunX06IU5tAq_964eOHSSKyTbdd9nVGhb-A/edit#heading=h.gjdgxs))

Existing vehicle detection algorithm ([Code](https://github.com/duckietown/Software/tree/UROP_Perception_Chandan/catkin_ws/src/vehicle_detection_tracking), [Duckumentation](https://docs.google.com/document/d/1enqjECNa-1_OZIZvHZekZ1tp872uYohSdGUyRf0s9BA/edit))


#### Other relevant resources to investigate
Robotics Handbook: Ideas for controller for 'follow-the-leader' or driving in traffic on pg. 808

### Risk analysis

Likelihood from 1-10, where 10 is very likely and 1 very unlikely.

Impact from 1-10, where 10 is a huge negative impact and 1 not so bad.

<div markdown="1">
 <col5 figure-id="tab:risks" figure-caption="Risks">
    <span>**Event**</span>
    <span>**Likelihood**</span>
    <span>**Impact**</span>
    <span>**Risk response Strategy (avoid, transfer, mitigate, acceptance)**</span>
    <span>**Actions required**</span>
    <span>False Negative in Vehicle detection can lead to crash</span>
    <span>4</span>
    <span>9</span>
    <span>mitigate</span>
    <span>The effect of FPs are less bad than those of FNs. Punish FNs more in an eventual cost function.</span>
    <span>False Positive in Vehicle detection makes bot stand still.</span>
    <span>4</span>
    <span>2</span>
    <span>accept</span>
    <span>-</span>
    <span>Inaccurate estimation of distance between bots</span>
    <span>3</span>
    <span>7</span>
    <span>mitigate</span>
    <span>Early testing, big enough safety margin, improve controller</span>
    <span>Collision of 2 duckiebots in an intersection</span>
    <span>3+4</span>
    <span>9</span>
    <span>mitigate</span>
    <span>Combination of Event 1 and 3</span>
 </col5>
</div>
