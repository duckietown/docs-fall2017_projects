#  Distributed Estimation: preliminary design document {#fleet-wireless-communication-preliminary-design-doc status=ready}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

Enable Duckiebots to communicate with each other wirelessly

### Motto

BI UNUM IBI OMNES (Where there is one, there is everybody)

<div class='check' markdown="1">

</div>

### Project scope

#### What is in scope

**Communication in a centralized network:**
* Communication between Duckiebots in one Duckietown
* Define interfaces for communication
* Performance testing on the communication system
* One network for one Duckietown
* TBD: does anything need to be synchronized?

**Option 1:**
Communication in a centralized network with a redundant centralized component (multiple routers)

**Option 2:**
Communication in a de-centralized network (ad-hoc)

![System Layout](https://github.com/duckietown/duckuments/blob/devel-distribution-est-fleet-wireless-communication/docs/atoms_85_fall2017_projects/16_distrubuted_est/Duckietown_Project_Image.png "System Layout")

#### What is out of scope

* Shared network communication between Duckietowns in case of a centralized network
* Communication redundancy with another physical layer
* Data processing → Message data is just de-serialized and provided to other components


#### Stakeholders

* Distributed Estimation
* Fleet planning
* Integration Heroes


## Part 2: Definition of the problem

### Problem statement

**Mission = Enable Duckiebots to communicate wirelessly**

**Problem Statement = Create a  communication framework for the Duckiebots**

### Assumptions

* Duckiebots can connect to a wifi network
* Data to be sent is already synchronized ???? TBD
* Duckiebots leave network when they are out of range or switched off
* If a Duckiebot is not connected to the communication network, it is not in Duckietown

### Approach

**Step 0:** Define contracts with distributed estimation / fleet planning teams

**Step 1:** Create the communication framework for the duckiebots and test it on a centralized network
* Create wifi communication network (physical hardware e.g. wifi router, configuration, etc.)
* Create a software component that serializes data (create/format datapackets)
* Create a software component to send and receive messages to/from other duckiebots
* Integrate Serialization and Messaging software components into THE communication software component (e.g. into a ROS node)
* Testing:
..* Live visualization of bandwidth (and/or latency, etc. → Network performance measures) of network

### Functionality-resources trade-offs

The space of possible implementations / battle plans is infinite.
We need to understand what will be the trade-offs.

### Functionality provided

How do you measure the functionality (what this module provides)?
What are the "metrics"?

<div class="example-usage" markdown="1">
numbers of dragons killed per hour
</div>


Note that this is already tricky. In fact, the above is not a good metric. Maybe we kill the dragon with an explosion, and also the prince/princess is killed. A better one might be:

<div class="example-usage" markdown="1">
numbers of royals freed per hour
</div>

<div class="example-usage" markdown="1">
probability of freeing a royal per attempt
</div>

It works better if you can choose the quantities so that functionality is something that you maximize to maximize. (so that you can "maximize performance", and "minimize resources").

### Resources required / dependencies / costs

How do you measure the resources (what this module requires)?

<div class="example-usage" markdown="1">
numbers of knights to hire
</div>

<div class="example-usage" markdown="1">
total salary for the mercenaries.
</div>

<div class="example-usage" markdown="1">
liters of poison to buy.
</div>

<div class="example-usage" markdown="1">
average duration of the battle.
</div>

It works better if you think of these resources as something to minimize.

### Performance measurement

How would you measure the performance/resources above? If you don't know how to measure it, it is not a good quantity to choose.

<div class="example-usage" markdown="1">
we dress up Brian as a Dragon and see how long it takes to kill him.
</div>

## Part 3: Preliminary design

### Modules

Can we decompose the problem?

Can you break up the solution in modules?

Note here we talk about logical modules, not the physical architecture (ROS nodes).

### Interfaces

For each module, what is the input, and what is the output?

How is the data represented?

Note we are not talking about ROS messages vs services vs UDP vs TCP etc.

### Preliminary plan of deliverables

What needs to be designed?

What needs to be implemented?

What already exists and needs to be revised?

### Specifications

Do you need to revise the Duckietown specification?

### Software modules

Here, be specific about the software:is it a ROS node, a Python library, a cloud service, a batch script?

### Infrastructure modules

Some of the modules have been designated as infrastructure

## Part 4: Project planning

Now, make a plan for the next phase.

### Data collection

What data do you need to collect?

### Data annotation

Do you have data that needs to be annotated? What would the annotations be?

#### Relevant Duckietown resources to investigate

List here Duckietown packages, slides, previous projects that are relevant to your quest

#### Other relevant resources to investigate

List papers, open source code, software libraries, that could be relevant in your quest.

### Risk analysis

What could go wrong?

How to mitigate the risks?
