#  Distributed Estimation: intermediate report {#template-int-report status=ready}

_It's time to commit on what you are building, and to make sure that it fits with everything else._

This consists of 3 parts:

- Part 1: System interfaces: Does your piece fit with everything else? You will have to convince both system architect and software architect and they must sign-off on this.

- Part 2: Demo and evaluation plan: Do you have a credible plan for evaluating what you are building? You will have to convince the VPs of Safety and they must sign-off on this.

- Part 3: Data collection, annotation, and analysis: Do you have a credible plan for collecting, annotating and analyzing the data? You will have to convince the data czars and they must sign-off on this.


<div markdown="1">

 <col2 id='checkoff-people-intermediate-report' figure-id="tab:checkoff-people-intermediate-report" figure-caption="Intermediate Report Supervisors">
    <s>System Architects</s>                         <s>Sonja Brits, Andrea Censi</s>
    <s>Software Architects</s>                       <s>Breandan Considine, Liam Paull</s>
    <s>Vice President of Safety</s>                  <s>Miguel de la Iglesia, Jacopo Tani</s>
    <s>Data Czars</s>                                <s>Manfred Diaz, Jonathan Aresenault</s>
 </col2>

</div>

## Part 1: System interfaces

### Logical architecture

Robots can broadcast messages to every other duckiebot in the network (centralized network for sure, and we’ll try to make ad-hoc networking work (as ad-hoc or mesh network)). Robots will post the received message data to the corresponding ros topic.

Ideally the network should scale from 2 to a town of Duckiebots. The network should also be robust to Duckiebots actively leaving and entering the network.

We will measure the latencies and other performance metrics but do not plan on optimizing the performance in regards of it. This can be improved over the next iteration.

Fleet communication:
1. Assume modules that need to use the communication capabilities e.g. multi-robot SLAM, fleet planning are able to send and retrieve ros messages.
2. Assume modules sends data of serializable type and reasonable size 
3. Assume communication is not time critical on sub second scale
4. Assume modules self synchronize  (if applicable)

<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

4. messaging node:
-	subscribed topics: individual outgoing communication (and by outgoing communication, we mean messages we send over wifi) topics published by fleet planning and distributed estimation
-	published topics: individual incoming communication (and by incoming communication, we mean messages we get over wifi) topics subscribed to by fleet planning and distributed estimation, and maybe other groups, since anyone can subscribe to these topics.

Optimizing for latency will be of low priority, since the primary single goal is to pipe through the data. If the data comes in faster that in goes back out from the ROS node, it shall be solved in a next iteration.

We’re going to try and make the node configurable such that the code will not needed to be changed in the future (maybe just optimized, since it is quite complex and we do not have much time to implement it). If not, we will hard code the message conversion (ROS message ←→ ZMQ message).

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


First Header       |        0      |
------------------ | -------------
Message Transport  | Messages cannot be sent or received  |
Network Traffic    | No traffic monitored |
Network Topology (centralized and decentralized) | Network cannot be established |

<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

_Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Collection

- How much data do you need?

- How are the logs to be taken? (Manually, autonomously, etc.)

Describe any other special arrangements.

- Do you need extra help in collecting the data from the other teams?

### Annotation

- Do you need to annotate the data?

- At this point, you should have you tried using [thehive.ai](https://thehive.ai/) to do it. Did you?

- Are you sure they can do the annotations that you want?

### Analysis

- Do you need to write some software to analyze the annotations?

- Are you planning for it?

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
