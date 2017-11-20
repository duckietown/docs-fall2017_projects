#  Fleet-level Planning: preliminary design document {#fleet-level-planning-preliminary-design-doc status=ready}


## Part 1: Mission and scope

### Mission statement

“Create the mobility-on-demand service for Duckietown.”

### Motto



<div class='check' markdown="1">

VICTORIA CONCORDIA CRESCIT (Victory through harmony)

</div>

### Project scope

Are you going to rewrite Duckietown from scratch? Probably not. You need to decide what are the boundaries in which you want to move.

#### What is in scope

* Send mobility commands to each duckiebot
* Receive each duckiebot’s location
* Calculate a set of mobility commands for all duckiebots
* Implement status LED patterns (i.e. like a taxi)
* (commands to arrange duckiebots in a pre-specified pattern)
* Implement customer request’s for pickup and destination at a desired location


#### What is out of scope

* Hardware modifications
* Fleet wireless communication
* Improvements to existing graphical representation of map and locations
* Fleet localization improvements (e.g. accuracy..)


#### Stakeholders

* Fleet-comms: for querying their API (contact: )
* Devel-coordination and multi-slam: 
* The Architects (smart city): accurate map of city, sufficiently big map to accommodate ~25 duckiebots at once



## Part 2: Definition of the problem

### Problem statement

We need to combine parts of many different stakeholders to achieve planning for a fleet of duckiebots. 

### Assumptions

* Sufficiently large duckietown to accommodate all duckiebots
* Collision avoidance and navigation works well to allow duckiebots to reach target destination
* Duckiebots can never park (i.e. stop still anywhere, unless waiting for other duckiebot at intersections etc.). 


### Approach

![alt text](git hub link to fleet-level_planning_diagram.png "Logo Title Text 1")

Necessary steps:

* [See Part 4: Project planning](Part 4)



### Functionality-resources trade-offs

Functionality includes:

* Visualization of N duckiebots
* Pick up and drop-off on request
* Functional standby distribution of duckiebots waiting for a pickup/ drop-off request
* Ability to arrange duckiebots in formations related to christmas videos 
* Taxi status lamps

Metrics:

* Minimize time for each service request to be completed


### Functionality provided

* Average handling time for each request
* Get the location for all duckiebots via the fleet-comms team and agree on the interface
* Get topological map of duckietown for planning


### Resources required / dependencies / costs

* Calculate time taken to complete request
* Number of requests served per time


### Performance measurement

How would you measure the performance/resources above? If you don't know how to measure it, it is not a good quantity to choose.

<div class="example-usage" markdown="1">
we dress up Brian as a Dragon and see how long it takes to kill him.
</div>

## Part 3: Preliminary design

### Modules

![alt text](git hub link to fleet-level_planning_diagram.png "Logo Title Text 1")

### Interfaces

Duckiefleet -  request handling server: 

* List of duckiebots and corresponding locations and statuses - will be sorted with the fleet-wireless-communications team, see Resources required / dependencies / costs

Customer - request handling server: 

* Pickup location and desired target location via clicking on map

Request handling server - Duckiefleet:

* List of target locations for each duckiebot such that request is completed 
* Each duckiebot displays its status via its LEDs


### Preliminary plan of deliverables

What needs to be designed?

What needs to be implemented?

What already exists and needs to be revised?

### Specifications

No revision of existing duckietown specification necessary.

### Changes to existing Software

Revisit visualization of Duckiebots on map and adapt it for visualization of N Duckiebots

### Software modules

* ROS node for the request handling server
  * Offers ROS service for customer requests
* (platinum) map on which customer can click to issue request, potentially as a separate ROS node


### Infrastructure modules

None

## Part 4: Project planning

| Week of | Task | Deliverable |
|---------|------|-------------|
| 13.11.2017 | Project kick-off and planning | Preliminary Design Document |
| 20.11.2017 | Look at state of current infrastructure | Running visualization of 1 duckiebot on map as currently implemented |
| 27.11.2017 | Visualization of n duckiebots |  |
| 04.12.2017 | Mission planner, implement m-stochastic queue median policy (or similar, tbd with Claudio) |  |
| 11.12.2017 | ...continued | Run test cases (e.g. send n reference locations to n duckiebots) |
| 18.12.2017 | ...continued | Run test cases (e.g. send n reference locations to n duckiebots) |
| 25.12.2017 | Implement customer request handling | Run test cases to establish reliable customer request handling routine |
| 01.01.2018 | Physical visualization of status, ETH formation | Verify that it works |





### Data collection

None

### Data annotation

None

#### Relevant Duckietown resources to investigate

According to meeting notes:

* Click and send for a single duckiebot is (probably) possible --> find corresponding code
* Graphical representation is running --> find corresponding code
* Read current documentation on tile-level localization
* We want to be able to send a go-to-position to a Duckiebot.
  * Already implemented. Video: [https://www.dropbox.com/s/93pbcktmwln4fqo/dp6b-draft.mov?dl=0](https://www.dropbox.com/s/93pbcktmwln4fqo/dp6b-draft.mov?dl=0)
  * DP6 :  link to the report
* Navigation to a point already implemented --> look at the code
* Visualization of 1 Duckiebot on a 2D map
* Look at code from Claudio re m-stochastic queue median policy + [paper](http://dx.doi.org/10.1109/CDC.2010.5717552)


#### Other relevant resources to investigate

Papers:

“Fundamental performance limits and efficient polices for Transportation-On-Demand systems“ by  Marco Pavone,  Kyle Treleaven and  Emilio Frazzoli  [link](http://dx.doi.org/10.1109/CDC.2010.5717552)


### Risk analysis

* Dependency on the Fleet-communications project. Closely work together with that team to get notified early about any upcoming problems that could delay the delivery of the needed parts for this project. 
* See Part 4: Project planning

