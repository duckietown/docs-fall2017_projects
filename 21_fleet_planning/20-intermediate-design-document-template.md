#  Fleet Level Planning: Intermediate Report {#fleet-level-planning-int-report status=ready}

This document describes system that the Fleet-planning team is planning to implement. Part 1 describes the interfaces to other parts of the system. I.e. How it communicates with all of them. In Part 2 a plan for the demo and evaluation is presented. And finally part 3 focuses on data collection, annotation and analysis. As for this project not much annotated data is used, part 3 is rather short.


## Part 1: System Interfaces

### Logical Architecture

The fleet planning system shall provide a graphical interface for visualization of n Duckiebots in a Duckietown. Duckiebots shall provide their location regularly to a central “planning node” (not running on a Duckiebot). Furthermore, an interface should exist to generate “taxi” commands (i.e. pickup guest at tile k and bring him to tile m). For such a request the system shall react with a command sent to one of the duckies to pick that customer up and transport him.

See Figure 1 for an overview of the logical structure of the fleet planning system.

In detail the complete process consists of the following steps:

1. Enter desired pickup and drop off location in GUI.
2. Planning node (see Figure 1) finds a Duckiebot that is available for a pickup. The matching is based on the availability of Duckiebots and their current locations. The planning node knows the location of each Duckiebot because they broadcast their position on a regular interval.
3. The location of the pickup is sent to the selected Duckiebot and that Duckiebot changes its fleet planning status to picking customer up as soon as it receives the message.
4. Based on the Duckiebot’s location and the received target location, the Duckiebot calculates locally a shortest path. This shortest path is nothing else than a list of directions for all the intersection that will be crossed on the path.
5. When the Duckiebot arrives at an intersection (realized by listening to flag), it publishes the instruction for this intersection.
6. Once the Duckiebot arrives at the location where it picks up the customer, the planning node sends the target location to the Duckiebot. Then steps 3, 4, and 5 are repeated until the Duckiebot arrives at the dropoff location.
7. The fleet planning state of the Duckiebot is set back to rebalancing.

<div figure-id="fig:logical-system-architecture" figure-caption="Fleet-level planning Logical System Architecture">
    <img style="width: 80%" src="logical-architecture.png" alt="Fleet-level Planning Logical System Architecture"/>
</div>

#### Assumptions

The communication between Duckiebots and the central planning node relies on the communication team of the distributed estimation project. To exchange messages on a fleet level we need this system to work reliably (i.e. no message loss) and with as little latency as possible (i.e. as little delay as possible between sending and receiving a message). We assume that the distributed estimation team can provide such a system within a reasonable timeframe. In part 2 of this document the interface to the communication layer is described. Based on this interface we can mock the communication and work out the fleet level planning part without a already working communication.

We further assume that the system has a map of Duckietown available. This map can be created by hand or with the system implemented by the distributed estimation team.

As described by the preliminary design document, the localization of the Duckiebot within Duckietown is out of scope. We utilize the existing april tag based localization from last years Duckietown course. First experiments are very promising and give good results. So we assume that the Duckiebot is able to localize himself within Duckietown if it is given a map of Duckietown. Furthermore, the localization can be enhanced by using the current speed information from the controllers. With that we can get an estimated position between intersections.

General assumptions that collision avoidance, line detection etc. function flawlessly are also made. Furthermore we ignore parking spots and parked Duckiebots and possible parking actions as they are not represented in our map. Lastly we assume that the clocks of the Duckiebots are reasonably in sync.

### Software Architecture

#### New Nodes

**Fleet planning node [central Laptop]**

This node knows the position and status of each Duckiebot in the network. It does the actual planning for the fleet. This consists of matching incoming transportation requests with available Duckiebots in such a way that the overall fleet is used in an optimal way.

Subscribed Topics:

- “Location”: To get the location messages from every Duckiebot.
- “Transportation Requests”: Every transportation request posted on this topic should be handled by the fleet planning node.

Published Topics:

- “Transportation status”: A topic that will get messages whenever something is updated within the fleet planning node. Example messages “”Robot” Picked up customer x at y”, “"Robot" Received transportation order from k to m”. This topic will introduce neglectable latency. As soon as the information is acquired from the sources it will post the message to this topic.  
- “Target Location”: here messages are published that contain a robot name and its target location. Based on this the robot will calculate its path to the target location locally. The latency between getting a transportation request and sending out a target location to one of the robots can not be determined offline as it is dependent on the current state of the system. If there is a Duckiebot immediately available, there is no delay. However, it might be that all Duckiebots are busy and therefore no Duckiebot can be assigned to that target location at the moment.
- “Fleet planning active”: Boolean flag indicating that the fleet planning node is active and wants to actively provide instructions at intersections.

**Visualization Node [central Laptop]**

This node is responsible of visualizing n Duckiebots on a map.

Subscribed Topics:

- “Location”: The stream of locations that comes in from all the Duckiebots.
- “Target Location”: Combining this knowledge with the messages from the “Location” topic, the calculated path of each Duckiebot can be visualized. The user can decide if the paths shall be shown. (The shortest path is also calculated locally on the Duckiebot, in order to reduce communication overhead).


Published Topics:

- “Visualization”: The rendered visualization as an image

**Taxi command execution node [local]**

This node will run on the Duckiebot and listen to commands from the central fleet planning node. Whenever it receives a command it starts the appropriate actions.

A Duckiebot can be in one of three fleet-planning states:
- Cruising/Rebalancing
- Picking up Customer
- Transporting Customer

A Duckiebot receives target locations from the central fleet planning node. It then calculates the shortest route to this location. For this the existing A*-path planning node is used. Given the Duckiebot’s current location and the target location, the path planning node can calculate instructions for how to get there. These instructions are then passed (on a per intersection basis) onto lower level navigation nodes (i.e. handled by navigator team).  

Furthermore this node also handles the back right LED which we are allowed to indicate the taxi status of the Duckiebot. Its status is communicated by the central fleet planning node. Additionally, when a customer is picked up a pattern is played on all the LEDs with very low intensity.

Subscribed Topics:

- “Location”: The location of the Duckiebot on the map
- “Stopline” and “April tag”: Whenever we are at a stop line with the according april tag we know that we are in front of an intersection. As a reaction to being at a stop line this node will publish the instruction that tells the Duckiebot what to do at this intersection. This instruction is based on the path it had calculated to reach its target location.
- “Taxi status”: indicates if the taxi is driving to a customer, is carrying a customer or is in idle (cruising/rebalancing) mode.


Published Topics:

- “Crossing_instructions”: Whenever the Duckiebot is at an intersection the node will publish on this topic what direction should be taken. As the path the Duckiebot takes was calculated previously there is no latency introduced. I.e. as soon as the Duckiebot gets the message that it stopped at an intersection it will publish the message with the instruction for this intersection to this topic. The message consists of a single integer. To have backwards compatibility with the current system this is one of the following values:
    - 0: left turn
    - 1: straight
    - 2: right turn
    - 3: random

#### Modified Nodes

Localization is based on last year’s “localization” package. For this purpose the map data was updated to match this year's Duckietown.

The fleet planning package is also based on last year's “navigation” package. It provides software to handle the path planning and a GUI that allows to select start and target nodes and displays the calculated path for a single Duckiebot. Multiple Duckiebot handling does not exist. The Duckiebot is then made to follow these commands. By now, we were not able to reproduce this feature in a stable manner. Also, in this package no location information is taken into consideration, the path planning and execution is executed in an open-loop manner. This shall be closed loop this year.


## Part 2: Demo and evaluation plan

### Demo plan

The demo from last year consisted of a single Duckiebot. It was possible to click on the node in the graph where the Duckiebot currently is and a target now where the Duckiebot should go. A path planning node then calculated a shortest path to the target location and the Duckiebot drives to that location.

For this years demo we envision a system that builds on top of that. A map is presented to the user that contains the current locations of all Duckiebots. The user can generate a transportation request by using a GUI. The system then assigns one of the Duckiebots to that task. That Duckiebots drives to that location, picks the customer up, drives to the target location and drops the customer again. The pickup and dropoff action are visible in the visualization. Further, the pickup and drop off can be visualized using the LEDs by showing a fancy pattern. There is no physical interaction planed. The system will be able to handle multiple of such requests at the same time. Also, the system shall be robust in the face of dying Duckiebots (may they rest in peace), thus a customer shall be assigned to a new Duckiebot if the original one is lost on it’s way. A Duckiebot counts as out of service if he does not publish a new location within a certain time window. No customers waiting forever. Unfortunately we cannot guarantee safety for a customer that is on a lost duckie-taxi.

Setting up this demo is as quick as starting all Duckiebots with the correct mode of operation and putting them on the map.

**Required Hardware**

- Duckietown (At least the same size as ML J44, depending on number of Duckiebots on duty)
- Several Duckiebots
- One laptop to act as the central fleet planning server (provided by fleet-planning group member)

### Formal performance evaluation

This project will introduce metrics that will be used to evaluate the performance of the fleet planning, providing a baseline for future groups working on further optimization. The metrics, introduced below, will be applied to a test-setup, which is described in the “proposed performance evaluation” section. The aim of the setup is to test the system’s reliability on the one hand (first scenario) and the capability to handle multiple requests at very high frequency and at the load limit, that is, at full capacity (number of requests at a given time == number of Duckiebots) on the other hand.

**Customer requests fulfilled per minute for given number of Duckiebots (ie. throughput)**. This metric measures how many customer request are handled by the server and fulfilled by all Duckiebots combined.
This would allow for the evaluation of different path planning algorithms and testing how well the rebalancing works.

**Mean distance of closest Duckiebot to the origin of a request for a set of requests and a given number of Duckiebots**. In the optimal case the Duckiebots will be distributed as homogeneously as possible in the Duckietown, minimizing the expected distance to each customer. This metric will enable the evaluation of how well a given implementation does this.

**[Stretch goal] Lost customers** In certain circumstances Duckiebots will fail (e.g. battery dies, Duckiebots gets stuck, etc.) and if this occurs while a Duckiebot is on the way to pick up a customer the fleet planning system should be able to send another available Duckiebot to fulfill the request instead. For the purpose of evaluation, several Duckiebots can be removed from the Duckietown at random and the system should still be able to fulfill all open requests. (A lost duckiebot can be detected using a timeout on the localization.


#### Proposed performance evaluation
In the Duckietown in ML-J44, that is a 5x6 Duckietown, 4 Duckiebots will be placed at the intersections in the following locations:

    - (0,3)
    - (2,3)
    - (4,3)
    - (2,5)

See picture below for initial locations.

The Duckiebots should operate at a predefined speed which is consistent across all tests for comparability. There should be two scenarios running 10 minutes each with 10 customer requests per scenario.

Scenario 1: 10 requests evenly spaced out across 10 minutes.
Scenario 2: 4 requests within the first minute, then 6 requests at 4,5,6,7,8,9 minutes respectively.

For each scenario, the method is the following:

1. Place Duckiebots in the Duckietown and have them move around in an idle state (i.e moving around randomly).
2. Send a command to move all Duckiebots to predetermined locations to ensure repeatability of the performance evaluation. The locations are
    - (0,1)
    - (2,5)
    - (4,1)
    - (1,5)
3. Start the evaluation by sending the series of customer requests during a 10 minute interval.

<div figure-id="fig:evaluation-1" figure-caption="Initial position">
    <img style="width: 80%" src="evaluation-1.png" alt="Initial positions for Duckies for evaluation"/>
</div>

<div figure-id="fig:evaluation-2" figure-caption="Final position">
    <img style="width: 80%" src="evaluation-2.png" alt="First target positions for Duckies for evaluation"/>
</div>


## Part 3: Data collection, annotation, and analysis

### Collection

We need data to do the formal performance evaluation. As the central fleet planning node has all the information about the Duckiebots (i.e. location at every point in time and taxi status) it is enough to log the information flowing through the topics to and from the central fleet planning node.

These logs will be used for the formal performance evaluation as described in Part 2 of this document.

### Annotation

None needed.


### Analysis

Analysis is done by hand on the acquired logs. As a stretch goal, a set of functions is made available that automates the process such that future teams working on improving this system can use the same evaluation strategy.
