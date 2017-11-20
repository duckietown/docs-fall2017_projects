#  Saviors: preliminary design document {#project-name-preliminary-design-doc status=ready}

<!-- Preliminary design document of the group saviors as of 19.11.2017
-->

## Part 1: Mission and scope

### Mission statement

Avoid obstacles. 

### Motto

Make Duckietown a safer place.

<div class='check' markdown="1">

*Insert Latin translation here.*

</div>

### Project scope

#### What is in scope

Detecting cones and duckies of different sizes (obstacles).

Stage 1: 1 obstacle, no crossing of lines (2 cases: drive by or stop)

Stage 2: 1 obstacle, crossing line if needed (1 case: should always be possible to drive by)

Stage 3: Multiple obstacles, crossing line if needed


#### What is out of scope

No obstacles in crossings, obstacles on the middle line 

#### Stakeholders

**Controllers (Lane following, adaptive curvature control)** - Marco Stalder, Simon Muntwiler, Anna Dai, Manuel Breitenstein, Andreas Aumiller

**Vehicle detection** - tbd



## Part 2: Definition of the problem

### Problem statement

Detect and avoid obstacles. 


### Assumptions
 * Obstacles are only yellow duckies (different sizes) and orange cones.
 * No duckies on the middle line.
 * No obstacles on intersections.
 * Heading and position relative to track given.
 * Control responsible for following trajectory.
 * Possibility to influence vehicle speed (slow down, stop).
 * Calibrated camera

### Approach

Stage 0: Obstacle detection and full stop<br>
Stage 1: 1 obstacle, no crossing of lines (2 cases: drive by or stop)<br>
→ simple logical conditions<br>
Stage 2: 1 obstacle, crossing line if needed (1 case: should always be possible to drive by)<br>
→ Either with grid map or obstacle coordinates<br>
Stage 3: Multiple obstacles, crossing line if needed <br>

### Functionality-resources trade-offs

The space of possible implementations / battle plans is infinite.<br>
We need to understand what will be the trade-offs.

### Functionality provided

Avoid duckies, measured in avoid/hit-percentage. Maximise avoid/hit ratio. 

### Resources required / dependencies / costs
 * Calibrated camera.
 * Position estimate and position uncertainty.
 * No costs involved.

### Performance measurement

Manual counting of the dead duckies to provide avoid/hit-ratio.

## Part 3: Preliminary design

### Modules
 * **Detection 2D space**
 * **3D obstacle coordinates and radius**
 * **Avoid obstacle**
 * Stage 0: Stop
 * Stage 1: Stop, drive around single obstacle
 * Stage 2: drive around multiple obstacles


### Interfaces
 * **Detection 2D space**
 * *Input:*
 * Camera image
 * Current position and orientation
 * (Curvature of upcoming track)
 * *Output:*
 * 2D obstacle coordinates
 * **3D obstacle coordinates and radius**
 * *Input:*
 * 2D obstacle coordinates
 * *Output:*
 * 3D obstacle coordinates
 * **Avoid obstacle**
 * *Input:*
 * 3D obstacle coordinates
 * Obstacle size
 * *Output:*
 * Trajectory, form tbd

### Preliminary plan of deliverables


### Specifications

No need to revise duckietown specifications

### Software modules

ROS nodes will be created.

### Infrastructure modules

No modules have been designated as infrastructure

## Part 4: Project planning

In a first phase, the controller project group needs to be contacted to identify interfaces

### Data collection

Images of duckies on the road.

Video of a duckiebot in duckietown with recordings of the different stages.

To log:
 * Distance to middle
 * Theta
 * Images
 * Velocity


### Data annotation

tbd

#### Relevant Duckietown resources to investigate
Image processing, feature extraction

MIT2016 object detection

Lane detection

Anti instagram


#### Other relevant resources to investigate
OpenCV

### Risk analysis
Interfaces (control approach of trajectory)

Computation power
