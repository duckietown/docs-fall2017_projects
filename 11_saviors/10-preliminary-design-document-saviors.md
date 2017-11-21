#  Saviors: preliminary design document {#project-name-preliminary-design-doc status=ready}

<!-- Preliminary design document of the group saviors as of 19.11.2017
-->

## Part 1: Mission and scope

### Mission statement

Detect obstacles, plan a route and drive around them.  

### Motto

Make Duckietown a safer place.

<div class='check' markdown="1">

*Please insert Latin translation here.*

</div>

### Project scope

**What is in scope**

Detecting cones and duckies of different sizes (obstacles) and plan a reasonable path or stop to avoid hitting them.

Stage 1: 1 obstacle and simply stop no crossing of lines (2 cases: drive by or stop)

Stage 2: 1 obstacle, drive by without crossing of line 

Stage 3: 1 obstacle, potentially cross the line 

Stage 3: Multiple obstacles, crossing line if needed


**What is out of scope**

No obstacles in crossings

Obstacles on the middle line

Complicated situations with oncoming traffic 

**Stakeholders**

*Controllers (Lane following, adaptive curvature control)*
Marco Stalder, Simon Muntwiler, Anna Dai, Manuel Breitenstein, Andreas Aumiller

They ensure following our desired trajectory and we can tell them to stop or reduce the speed.
They provide the heading and position relative to track (for path planning)

*Vehicle detection* - tbd

*Potentionally Anti-Instagram*
They provide classified edges to limit the area where we have to find obstacles.

*Organizational Help* - System Architect - Software Architect - Duckiebook



## Part 2: Definition of the problem

### Problem statement

Relialby detect and avoid obstacles, plan a meaningful path around them or simply stop if nothing else is possible.

Robustness to changes in:

 * Obstacle size
 * Obstacle color (but only slight changes in yellow/orange)
 * Illumination 


### Assumptions
 * Obstacles are only yellow duckies (different sizes) and orange cones.
 * No duckies on the middle line.
 * No obstacles on intersections.
 * Heading and position relative to track given.
 * Control responsible for following trajectory.
 * Possibility to influence vehicle speed (slow down, stop).
 * Calibrated camera

### Approach

Stage 0: Collect enough data and annotate them

Stage 1: Develop a first obstacle detection algorithm

Stage 2: Agree on final internal and external interface

Stage 3: from now on, obstacle detection and trajectory planning can be developed in parallel

Stage 4: handle the case(s) involving: 1 obstacle, no crossing of lines (2 cases: drive by or stop)
→ simple logical conditions

Stage 5: handle the case(s) involving: 1 obstacle, crossing line if needed (1 case: should always be possible to drive by)
→ Either with grid map or obstacle coordinates

Stage 6: handle the case(s) involving: Multiple obstacles, crossing line if needed

Stage 7: verify the whole system 

### Functionality provided

 * Detect Obstacles
 * Plan path around them or decide to stop 

### Resources required / dependencies / costs
 * Calibrated camera.
 * Position estimate and position uncertainty.
 * Execution of our desired control commands
 * Enough computing power

### Performance measurement

 * Avoid/hit-ratio in Stages 4-6 (see Approach)
 * Percentage of correctly classified obstacles on our picture dataset
 * Both of the measures above in case of changing light conditions

### Functionality-resources trade-offs

  * Robust obstacle detection (many filters,...) vs. computational efficiency
  * Maximizing speed (e.g. controlles might want to do that) vs. motion blur

## Part 3: Preliminary design

### Modules
 * **Obstacle Detection in 2D space**
 * **Reconstruct 3D obstacle coordinates and radius**
 * **Path planning/ Decision making**


### Interfaces
 **Detection 2D space**

 * *Input:*
 * Camera image
 * Current position and orientation
 * Lane coordinates
 * Camera intrinsics
 * (Curvature of upcoming track)

 * *Output:*
 * 2D obstacle coordinates

 **Reconstruction of 3D obstacle coordinates and radius**

 * *Input:*
 * 2D obstacle coordinates
 * Extrinsics

 * *Output:*
 * 3D obstacle coordinates and radius

 **Avoid obstacle**

 * *Input:*
 * 3D obstacle coordinates
 * Obstacle size
 * Lane information

 * *Output:*
 * Trajectory
 * Control command

### Preliminary plan of deliverables


### Specifications

No need to revise duckietown specifications

### Software modules

 * Detection and Projection Node
 * Path Planning Node


## Part 4: Project planning


### Timeline

| Date | Task Name | Target Deliverables |
| :--------- | :--------------------------- | :------------- |
| 15/11/17 | First Meeting | Preliminary Design Document |
| 17/11/17 | Record first bags | Pictures and Raw bag data |
| 22/11/17 | Exchange of ideas | Basic concept |
| 29/11/17 | Knowing Interfaces and State of the Art | Fine concept |
| 06/12/17 | | First implementation |
| ... | Testing | Optimized Code |
| ... | Documentation | Duckuments |
| 21/12/17 | | End of Project |

### Data collection

Images of duckies on the road.

Video of a duckiebot in duckietown with recordings of the different stages.

To log:

 * Distance to middle
 * Theta
 * Images
 * Velocity


### Data annotation

Label obstacles

#### Relevant Duckietown resources to investigate

Image processing

feature extraction

MIT2016 object detection

Lane detection

Anti instagram


#### Other relevant resources to investigate
OpenCV (filtering, color and edge detection)

### Risk analysis
Interfaces (control approach of trajectory)

Computation power

### Risk analysis

| Risk | Likelihood (1-10) | Impact | Actions required |
| :--- | :--- | :--- | :--- | 
| Non robust State Estimation | tbd | very high | Communication/Collaboration with controlling subteam |
| Failure of following our desired control commands | tbd | very high | Communication/Collaboration with controlling subteam |
| Lack in computation power | 5 | high | early testing of whole system on duckiebot |
| Failure in duckie detection | 4 | extremely high | thorough testing on bags |
| Erroneously detecting the middle lane as duckie | 7 | middle | more sophisticated detection algorithm |

