#  Distributed Estimation: intermediate report

## Part 1: System interfaces

### Logical architecture

Robots can broadcast messages to every other Duckiebot in the network (centralized network for sure, and we’ll try to make ad-hoc networking work (as ad-hoc or mesh network)). Robots will post the received message data to the corresponding ROS topic.

Ideally the network should scale from 2 to a town of Duckiebots. The network should also be robust to Duckiebots actively leaving and entering the network.

We will measure the latencies and other performance metrics but do not plan on optimizing the performance in regards of it. This can be improved over the next iteration.

Fleet communication:
1. Assume modules that need to use the communication capabilities e.g. multi-robot SLAM, fleet planning are able to publish and subscribe to ROS topics.
2. Assume modules sends data of serializable type and reasonable size 
3. Assume communication is not time critical on sub second scale
4. Assume modules self synchronize  (if applicable)

<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

messaging node:
-	subscribed topics: individual outgoing communication (and by outgoing communication, we mean messages we send over wifi) topics published by fleet planning and distributed estimation. These teams publish their data to be sent to these topics.
-	published topics: individual incoming communication (and by incoming communication, we mean messages we get over wifi) topics subscribed to by fleet planning and distributed estimation, and maybe other groups, since anyone can subscribe to these topics.

Outwards (wifi) communication is realized with protobuffs and zmq

Optimizing for latency will be of low priority, since the primary single goal is to pipe through the data. If the data comes in faster that in goes back out from the ROS node, it shall be solved in a next iteration.

We’re going to try and make the node configurable such that the code will not needed to be changed in the future (maybe just optimized, since it is quite complex and we do not have much time to implement it). If not, we will hard code the message conversion (ROS message ←→ ZMQ message).

How does this work?

Once:

- Team A wants to communicate between bots

- Team A tells us their message structure

- We build serialization

- We define ROS topics: teamAout, teamAin

Perpetually (as long as message structure doesn't change:

- Team A bot A posts message to teamAout

- We automatically serialize message with corresponding serialization

- We send on bot A

- We recieve on all other bots

- We deserialize and post to teamAin

- Team A other bots can retrieve message from teamAin


<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan

_Please note that for this part it is necessary for the VPs for Safety to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Demo plan

Multi-Robot SLAM and fleet planning relies on communication to work, therefore the communication demo is implicit in the SLAM and fleet planning demo.
We think a sole communication demo would not be too impressive to the casual demogoer.

At least three Duckiebots (configured for mesh networking i.e. with additional wireless adapters installed, if it works.) Otherwise, a global network (e.g. Duckietown) for the centralized structure.


### Plan for formal performance evaluation

There are three main criterias that have to be evaluated:
1. Message transport:
    1. Centralized Network: test if a simple message e.g. a string can be sent from one duckiebot to another reliably. 
    2. Decentralized Network: test message propagation. In a mesh network two Duckiebots (nodes) may not be directly connected, therefore we must test if a message can be propagated through the network to the correct receiver.
    3. Check of dropped messages
2. Network traffic: accurately monitor network traffic
3. Network topology: visualize nodes entering and leaving the network reliably


| |0|1|2|3|4|
|---|---|---|---|---|---|
|Message Transport  | Messages cannot be sent or received | Strings can be sent and received on tcp | Messages can be serialized and sent and received on tcp | Messages can be serialized and multicast and received on pgm |Messages can be serialized and multicast and received on pgm on a mesh network|
|Network Traffic|No traffic monitored|Can see traffic on the network but no useful information extracted|Able to isolate duckiebot traffic|Able to identify specific packets|Able to visualize routing of specific packages|
|Network Topology (centralized and decentralized)|Network cannot be established|Initial Network can be established, but no new nodes can connect to the network, not robust to connection loses|Initial Network can be established, new nodes can connect but not reliably, not robust to connection loses|Initial Network can be established, new nodes can connect/leave dynamically, but not robust to connection loses|Initial Network can be established, new nodes can connect/leave dynamically, and robust to connection loses|


<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

_Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Collection

Initially a set of compiled dummy messages is used to build the fleet-communication.
For further implementation and evaluation one ROS-bag of broadcasted messages is needed from the fleet-planning team and from the multi-robot-SLAM product each. 

### Annotation

For the fleet-communication no data annotation is needed.

### Analysis

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
