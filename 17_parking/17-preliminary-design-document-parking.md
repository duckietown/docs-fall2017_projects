#  Group name: preliminary design document {#project-name-preliminary-design-doc status=ready}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

Implement parking feature and design specifications (feature and physical)

### Motto

Duck Ex Machina

<div class='check' markdown="1">

Quidquid latine dictum sit, altum videtur

</div>

### Project scope

Implement parking feature and design specifications (feature and physical)


#### What is in scope

*forward parking
*bot localization with april tags
*open spot localization 
*path generation (coming in and out of parking space)
*how to drive backwards
*parking lot full signal
*physical parking lot design specification 


#### What is out of scope

*Develop new algorithms to filter new lane line colors
*Fleet level coordination 
*Handle multiple parking events at once 


#### Stakeholders

*Single SLAM or distributed-est: person
*Controls: Manuel Breitenstein (Nils)
*Smart City: Sam Nickolay (Brett)
*Anti Instagram: Christoph Zuidema (Sam)


## Part 2: Definition of the problem

### Problem statement

We need to park N duckiebots in a designated area in which they are able enter and exit in an efficient manner.

### Assumptions

*Four tile structure with defined inlet, outlet and color scheme
*Known design specification of parking lot
*Assume when leaving parking space, path is free of other duckiebots
*When entering lot and searching for parking space, parking lot is in static state


### Approach

*Enter parking lot in designated inlet lane
*Localize based on april tags within field of view with known locations
*Control with feedback along predetermined path 
*Detect parking space status (full/free) of each parking space in sequential manner
*Locate a free parking space
*Paths generated for maneuvering into parking space
*High fidelity control into parking space
*Signal generated to signify parking space is full
*When we want to leave a space, generate path out of parking space
*High fidelity control out of parking space (with caster wheel dynamics taken into account for feedback control)
*Control with feedback along predetermined path
*Exit parking lot in designated outlet lane


### Functionality-resources trade-offs



### Functionality provided

*Probability of a successful parking maneuver per parking maneuver attempt
*Number of duckiebots within the parking lot boundary per hour


### Resources required / dependencies / costs

*Size of parking space 
*Resources required to develop duckiebot trajectory 
*Number of april tags and infrastructure to support april tags 

### Performance measurement

*Starting at parking lot entrance, measure the number of parking maneuvers completed within boundaries of designated parking spot (over N attempts)
*Starting in designated parking space, measure the number of duckiebots able to arrive at the exit of the parking lot (over N attempts)
*Average time (for N vehicles) to enter and exit parking lot

## Part 3: Preliminary design

### Modules

*Perception
	*Lane filtering
	*April tag detection 
	*“Fleet communication”: detecting
*Localization and parking map generation
	*Ego localization
	*Localization other duckiebots
	*Parking map design
*Planning
	*Parking space allocation 
	*Path planning
*Control
	*“Fleet communication”: publishing


### Interfaces

#### Perception

*Lane filtering
	*Used for pose estimation at entrance and exit of parking lot and maybe at parking space
	*Input: camera image
	*Output: location lanes
*April tags detection and triangulation
	*Use for pose estimation while driving on the parking lot when no lanes can be identified, every parking space has its own april tag, relative duckiebot-tag pose is extracted using computer vision
*Input: camera image
*Output: location of april tag, relative position duckiebot-tag

*“Fleet communication”: detecting
	*Blinking LEDs are used for communication: while parking signal who is driving (one at the time), while parked signal which parking lot is taken (duckiebot on parking space 2, blink led in parking space 2 specific frequency)
	*Input: camera image
	*Output: other occupied signals (other means blinking signals is not from own duckiebot)

#### Localization and parking map generation

*Localization other duckiebots
	*Determines the pose of other duckiebots using specific blinking LEDs 
	*Input: other occupied signal(s)
	*Output: pose other duckiebot(s)
*Parking map design
	*Static map (physical known map with defined parking spaces and areas to move to them, without duckiebots) is merged with pose of other duckiebots to generate a {occupied, free} map of the parking lot
	*Input: static map (has to be defined offline), pose other duckiebots
	*Output: parking map
*Ego localization
	*State estimation of position (x, y) and heading (theta) of own duckiebot using lanes (at entrance/exit of parking lot) and april tags
	*Input: parking map, relative position duckiebot-tag(s), localization april tag(s), location lane(s)
	*Output: pose duckiebot 

#### Path planning

*Parking space allocation
	*Allocates a parking space to the duckiebot given the parking map and the authority to move, executed once per duckiebot 
	*Input: parking map, pose duckiebot
	*Output: pose parking space (x, y, theta)
*Path planning
	*The actual path planning module
	*Input: pose parking space, pose duckiebot, parking map
	*Output: reference path or reference trajectory

#### Control

*High fidelity control algorithm to drive duckiebot on reference trajectory/path to allocated parking space, flag when parked
*Input: reference path, pose duckiebot, (maybe parking map → constrained control)
*Output: motor voltage, parking status = {going to parking space, parked, want to leave, exiting parking space}

#### “Fleet communication” publishing

*Flag status using specific frequency on LEDs (or color for human eye)
*Input: parking status
*Output: own_occupied_signal

### Preliminary plan of deliverables

*Need: infrastructure, localization algorithm using april tags (maybe fusion with lane detection), high fidelity control algorithm, map generation algorithm, localization (ego and other duckiebots), parking space allocation, path planning algorithm
*Exists: Lane detection, color filters, lane control, LED communication, april tag detection, control algorithm (maybe has to be improved), 



### Specifications

Yes, we need to add parking lot specifications. 

### Software modules

A collection of ROS nodes.

### Infrastructure modules

Yes, we will include infrastructure modules to specify parking lot specifications.

## Part 4: Project planning


### Data collection

*April tag localization data
*April tag distance data (detection in a range of ~10 cm until ~1 m away from sign)
*Duckiebot to duckiebot communication using flashing LEDs


### Data annotation

No

#### Relevant Duckietown resources to investigate

*April tag detection and localization (what is done already?)
*Control algorithm with good enough precision
*Transforming pose to configuration space
*Path planning algorithm (RRT*)
*Driving backwards (together with the control guys) while updating the pose of the duckiebot


#### Other relevant resources to investigate

*Transforming pose to configuration space
*Path planning algorithm (RRT*)


### Risk analysis

*Localization fails while driving backwards
*Traffic jam at entrance of parking lot
*Fleet communication fails: incoming duckiebot does not see currently parking duckiebot, two duckiebots leave at the same time
*Detection of april tags and extracting pose of the robot
*Map generation is wrong if duckiebot is not parked to specification
*Control: level of precision adequate for parking
*Exit parking maneuver conflicts: who can drive first (duckiebot which is exiting does probably not see anything) 

