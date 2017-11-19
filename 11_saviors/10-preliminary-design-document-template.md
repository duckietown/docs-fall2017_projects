#  Group name: preliminary design document {#project-name-preliminary-design-doc status=ready}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

Avoid obstacles. 

### Motto

Make Duckietown a safer place.

<div class='check' markdown="1">

<b><i>Insert Latin translation here.</i></b>

</div>

### Project scope

#### What is in scope

Detecting cones and duckies of different sizes (obstacles). <br>
Stage 1: 1 obstacle, no crossing of lines (2 cases: drive by or stop)<br>
Stage 2: 1 obstacle, crossing line if needed (1 case: should always be possible to drive by)<br>
Stage 3: Multiple obstacles, crossing line if needed <br>


#### What is out of scope

No obstacles in crossings, obstacles on the middle line 

#### Stakeholders

<b>Controllers (Lane following, adaptive curvature control)</b> - Marco Stalder, Simon Muntwiler, Anna Dai, Manuel Breitenstein, Andreas Aumiller<br>
<b>Vehicle detection</b> - tbd



## Part 2: Definition of the problem

### Problem statement

Detect and avoid obstacles. 


### Assumptions
<ul>
	<li>Obstacles are only yellow duckies (different sizes) and orange cones. </li>
	<li>No duckies on the middle line. </li>
	<li>No obstacles on intersections.  </li>
	<li>Heading and position relative to track given.  </li>
	<li>Control responsible for following trajectory.  </li>
	<li>Possibility to influence vehicle speed (slow down, stop). </li>
	<li>Calibrated camera </li>
</ul>

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
<ul>
	<li>Calibrated camera.</li>
	<li>Position estimate and position uncertainty.</li>
	<li>No costs involved.</li>
</ul>


### Performance measurement

Manual counting of the dead duckies to provide avoid/hit-ratio.

## Part 3: Preliminary design

### Modules
<ul>
	<li>Detection 2D space</li>
	<li>3D obstacle coordinates and radius</li>
	<li>Avoid obstacle
		<ul>
			<li>Stage 0: Stop</li>
			<li>Stage 1: Stop, drive around single obstacle</li>
			<li>Stage 2: drive around multiple obstacles</li>
		</ul>
	</li>
</ul>


### Interfaces
<ul>
	<li>Detection 2D space
		<ul>
			<li>Input:
				<ul>
					<li>Camera image</li>
					<li>Current position and orientation</li>
					<li>(Curvature of upcoming track)</li>
				</ul>
			<li>
			<li>Output:
				<ul>
					<li>Output: </li>
				</ul>
			</li>
		</ul>
	</li>
	<li>3D obstacle coordinates and radius
		<ul>
			<li>Input:
				<ul>
					<li>2D obstacle coordinates</li>
				</ul>
			</li>
			<li>Output:
				<ul>
					<li>3D obstacle coordinates</li>
				</ul>
			</li>
		</ul>
	</li>
	<li>Avoid obstacle
		<ul>
			<li>Input:
				<ul>
					<li>3D obstacle coordinates</li>
					<li>Obstacle size</li>
				</ul>
			</li>
			<li>Output:
				<ul>
					<li>Trajectory, form tbd</li>
				</ul>
			</li>
		</ul>
	</li>
</ul>

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

Images of duckies on the road.<br>
Video of a duckiebot in duckietown with recordings of the different stages.<br>
To log:<br>
<ul>
	<li>Distance to middle</li>
	<li>Theta</li>
	<li>Images</li>
	<li>Velocity</li>
</ul>


### Data annotation

tbd

#### Relevant Duckietown resources to investigate
Image processing, feature extraction<br>
MIT2016 object detection<br>
Lane detection<br>
Anti instagram<br>


#### Other relevant resources to investigate
OpenCV

### Risk analysis
Interfaces (control approach of trajectory)<br>
Computation power<br>

