# Fleet Planning: Final Report {#fleet-planning-final-report status=beta}

TODO: JT: fix image sizes

## The final result

<div figure-id="fig:demo_video_fleetplanning">
    <figcaption>The Fleet Planning Demo Video</figcaption>
    <dtvideo src="vimeo:257378029"/>
</div>

 See the [operation manual](https://github.com/duckietown/duckuments/blob/9e0df2f522f4e6d7f96d45cd22b9d66d87e554bc/docs/atoms_20_setup_and_demo/30_demos/26_fleet_level_planning.md) to reproduce these results.

## Mission and scope

### Motivation

As cities such as Duckietown grow, their inhabitants can no longer walk from each point of the city to the next; especially after a long night out. With the tremendous growth of Duckietown, the need for a mobility-on-demand (MOD) service became unbearable and was addressed in this project.
Duckietown being a city in which autonomous Duckiebots roam the streets, it makes sense for them to provide the much-needed MOD service. If Duckiebots share their positions as well as their next destinations, with a central dispatcher node they can work together to deliver an efficient service.
The development of a multi-duckiebot MOD service is thus reliant on many other Duckietown components such as localization, mapping, navigation, collision-avoidance and communication.


### Existing solution


The existing codebase was set up to support localization, navigation and visualization for a single Duckiebot in a fixed, hard-coded Duckietown, represented as a tile-map image; see the [specification](http://book.duckietown.org/master/duckiebook/duckietown_specs.html#sec:duckietown-specs) for the available tile-types.



By combining a set of rotated tiles, a map of any given Duckietown can be formed.
In the existing codebase, such a tile-map was supplied as an [image](https://github.com/duckietown/Software/blob/master/catkin_ws/src/20-indefinite-navigation/navigation/src/maps/map.png) and a CSV file of tile-types and orientations; however neither code for composition of a tile-map nor documentation as to how this was generated in the first place were supplied.



The entire code for localization, navigation and visualization ran on the Duckiebot and was linked to the other modules using a Finite State Machine (FSM).



The codebase can be found [here](https://github.com/duckietown/Software/tree/master/catkin_ws/src/20-indefinite-navigation/navigation).



#### Localization


Localization was performed by placing AprilTags at each intersection and having the Duckiebot identify each unique AprilTag through image analysis. The AprilTags are defined in the duckiebook [signage section](#sec:signage).

A Duckiebot could thus compute at which $(x,y)$ coordinate of the map it was and estimate its rotation. The position on the topographic map was not mapped to the corresponding topological graph representation which is required for path planning.


#### Navigation

A graph was computed from the tile-map and given a start and an end node, A* search was performed on that graph to provide the list of nodes along the shortest path. This list of nodes was then converted into a set of instructions the Duckiebot could follow. These instructions were of the form “straight”, ”left”, ”left”, … They were executed in open loop manner, i.e. once the entire path was calculated, the Duckiebot was sent off to execute it without any intermediate checks whether the Duckiebot was doing this correctly. Thus any deviation of the Duckiebot from the path, or noisy FSM-transitions led to failure of the system.
The python package graphviz was used to compute the graph and generate a visualization, i.e. an image of the graph.


#### Visualization/GUI


The GUI consisted of a list, from which the user could select the desired start- and destination nodes for one single Duckiebot as well as an overlay of the tile-map and the graph image. The location was indicated by highlighting the corresponding node in a different color, and the path by highlighting the graph edges. No localization was integrated in the demo, and thus the start node had to be set manually and the subsequent execution of the path was completely open loop.


When the user hit the start button, a request was sent to the navigation ROS node, upon which the start and end graph nodes were highlighted and the computed path was indicated on the image.



### Opportunity


The existing solution could benefit from scalability, adaptability and localization. It has been set-up, and often hard-coded, to work only with a single Duckiebot in a completely open loop manner. Without human interaction the Duckiebot would not move.



For fleet-level behaviour and our MOD service, the solution needed to be scaled up to work with n Duckiebots. Also, in order to being able to implement sophisticated fleet planning algorithms, an API handling and tracking the statuses of the Duckiebots and customer requests appeared highly desirable.



Building this infrastructure was a priority to us, since a good software foundation will make future endeavours in the field more productive and will allow to focus on more complex fleet planning concepts. Nevertheless, inspiration for intelligent fleet-level behavior and automatic rebalancing for more efficient request handling were drawn from [1].


Critical components such as the request-handling were desired to move away from running on a single duckiebot. The previous solution also had no stack and could thus handle only one request at a time. A new request simply overwrote its predecessor, which led to incomplete jobs. These were unacceptable limitations to a true MOD service and could be overcome through the implementation of a central node dealing with coordination, called the taxi central node.


In coordination with the fleet-communications team, a method of sharing information and sending commands to multiple Duckiebots was introduced. The GUI was also changed so as not to run on the Duckiebot, but rather directly on the customer’s machine to allow multiple customers to utilize the service at the same time.


Furthermore, the existing solution had been set up to work with one specific Duckietown layout only. Since a MOD service should be capable of running in every Duckietown, this issue was also addressed. The tile-map and graph are now automatically generated off of a CSV based description, allowing for easy adaptations to an existing Duckietown and the application of our MOD service in new Duckietowns.


The GUI was also rewritten to scale well with varying Duckietowns and number of Duckiebots and now allows the MOD customer to issue requests more easily. The GUI workflow was also improved with respect to intuitive usability and faster issuing of customer requests. It also has the capability to show all available Duckiebots and their locations.
The improved solution is also more flexible with regards to localization. Once this is properly merged with the localization teams’ efforts, the resolution of localization will be dramatically improved.



To verify our work, a test environment and virtual Duckiebots were introduced. The previous solution had no means of demonstrating functionality virtually; i.e. without a physical demo.



## Definition of the problem


### Problem Statement


  Integrate expectations of different stakeholders to achieve planning
   for a fleet of Duckiebots. This can be broken down into the following
   tasks:


 - Receive location information and status from n Duckiebots
 -  Visualize n Duckiebots on the current Duckietown map
 -  Receive and process pick-up and drop-off requests which are issued using a GUI
 -   Assign customer requests and rebalancing targets to Duckiebots
 - Standby distribution (“rebalancing”) of Duckiebots waiting for customer requests
  - Send target location (customer requests or rebalancing targets) to n Duckiebots
  - Indication of each Duckiebot’s status using their LEDs.  A Duckiebot’s status indicates whether it is on its way to a customer, currently transporting a customer or idle.
   - Local path planning and execution on Duckiebots


### Assumptions


Per the preliminary and intermediate design documents, we assume that the Duckietown is large enough to accommodate all Duckiebots and that collision avoidance, line-detection etc. function reasonably well.
We further assume that the system has a CSV map representation of Duckietown conforming to the conventions defined [here](https://docs.google.com/document/d/1VE2v2Yn8d4wzA8DnPuA429gYzFeV_zTX8rDFCZCKIE0/edit)
 This map can be created by hand or with the system implemented by the distributed estimation team.
The communication between Duckiebots and the central planning node relies on the communication team of the distributed estimation project. To exchange messages on a fleet level we need this system to work reliably (i.e. without message loss) and with sufficiently small latencies, i.e. less than roughly a second.


### Contracts {#fleet-planning-final-report-contracts}

#### Distributed estimation and communication team:


The fleet communication team provides a means of transporting arbitrary data as a byte array from one Duckiebot to another Duckiebot or to a computer. Communication channels are set up via a configuration file and messages are sent/received by publishing/listening to messages on a ROS topic. This was integral to the functioning of our system, as we required multiple ROS master nodes running on each Duckiebot locally, nonetheless sharing their information with a separate ROS master on a central computer.


#### The Architects:


Initially the idea was for the Architects to design a Duckietown sufficiently large to accommodate a large number of Duckiebots. Closer to the demo day, it emerged that several smaller Duckietowns would be used so this was no longer needed.


### Performance Evaluation and Metrics
The performance of our project was primarily evaluated qualitatively, since the focus of the project was on providing a working framework to easily implement further fleet planning strategies. The results are discussed in 3.5.



## Contribution / Added functionality

The work on the 2017 fleet planning project was distributed in the following manner: 80% software infrastructure, 10% algorithms and 10% package integration of other teams. The final software can be divided into five parts:


### Path planning and execution


Main component can be found [here](https://github.com/duckietown/Software/blob/devel-fleet-planning/catkin_ws/src/20-indefinite-navigation/fleet_planning/src/actions_dispatcher_node.py).


This nodes listens to location updates from the april tags localization package and target destinations from the taxi central node.



The (x,y) location information is then mapped to the topological graph representation. Using the rotation of the Duckiebot and by finding the red line with the minimum distance to the Duckiebot, the location of the Duckiebot is set to the node that best explains this configuration.


Once the node has received both location and a mission target, it executes A* path planning and publishes the next intersection instruction, i.e. "left", "right", etc., to be received by the intersection navigation package (implemented by the 2016 team). The path is recomputed at each intersection such that deviations from the original plan do not lead to failure of the entire system; this guarantees a certain robustness to errors of other software components.

The calculated path and current location is then reported to the central planning node, called taxi central.

This node runs locally on each Duckiebot.


### Fleet planning aka taxi central node


Core component can be found [here](https://github.com/duckietown/Software/blob/devel-fleet-planning/catkin_ws/src/20-indefinite-navigation/fleet_planning/src/taxi_central_node.py).


Duckiebot and Customer logic is defined [here](https://github.com/duckietown/Software/blob/devel-fleet-planning/catkin_ws/src/20-indefinite-navigation/fleet_planning/include/fleet_planning/duckiebot.py).



  - Runs on a central laptop, handles all the fleet planning logic
  - Maintains a set of active Duckiebots.
  - Receives location updates from
   each Duckiebot and stores them. If a Duckiebot does not update its location within a   certain time window, it is considered dead and removed from the map. Customers onboard are re-assigned to a new Duckiebot, with their pick-up location corresponding to the last known location.  
   - Receives customer requests from GUI, assigns them to
   a nearby Duckiebot based on FIFO breadth first search. For details, see below.
   - Tracks execution status of customer requests, and thus
   tracks whether customer start or target location has been reached and stores the timestamps for each state transition and each request. This allows evaluation of fleet planning metrics such as
   time-to-customer or execution times of the whole request.
   - Duckiebots that are not busy are assigned rebalancing goals. These are currently random locations on the map. This appeared to be a reasonable probabilistic approximation to an optimal strategy, assuming a large
   number of Duckiebots on a uniform map which will then spread rather homogeneously. The software was designed in a way that allows easy extension of the current approach. Also, it is possible to switch between different approaches by setting a single enum variable which can be very useful for testing and comparing rebalancing strategies.



**FIFO breadth first search:** The taxi central stores a list of pending (i.e. not yet assigned) customer requests and idle Duckiebots. Our algorithm then selects the closest Duckiebot to each customer in First-In-First-Out manner:


For each customer in pending_customer_requests:


 - Find closest Duckiebot via breadth-first search on the map graph
 - Assign customer to Duckiebot
 - Remove Duckiebot from list of idle Duckiebots



Although this approach is not fully optimal, it is a reasonable approximation for the 2017 Duckietown setup with a low frequency of new customer requests and a small number of operating Duckiebots. Our software is designed in a way that makes it easy to add more sophisticated approaches. This could include strategies that take into account expected distributions of customers and send Duckiebots to anticipated hotspots ahead of time (e.g. dealing with rush hour customer spikes).


### GUI


The code can be found [here](https://github.com/duckietown/Software/blob/devel-fleet-planning/catkin_ws/src/20-indefinite-navigation/fleet_planning/include/rqt_fleet_planning/rqt_fleet_planning.py).


Since the existing GUI was running directly on the Duckiebot and was laid out for a single Duckiebot system, it had to be completely rewritten. To design the front end, the QtGui module was utilized; the GUI itself runs as an rqt module.



To keep the GUI scalable and extensible along with the rest of our solution, it is able to run on multiple devices at the same time, as long as each device can communicate with the ROS master that the taxi central node is running on. The GUI communicates with other modules through ROS messages and topic listeners/subscribers and runs largely independently of all other components of the fleet planning module.

The source code is located in this following [folder](github:org=duckietown,repo=Software,path=devel-fleet-planning/catkin_ws/src/20-indefinite-navigation/fleet_planning/include).

In this section, the GUI components and their interactions with the other modules are described. The overall layout follows design principles outlined in Galitz’ “The essential guide to user interface design: an introduction to GUI design principles and techniques” [2].

Please note that components (2) through (5) are re-positioned depending on the Duckietown map’s size.

[GUI without customer](#fig:fleet-planning-gui-without-customer).

<div figure-id="fig:fleet-planning-gui-without-customer">
   <img src="gui_1.png" width="25em"/>
   <figcaption>Map of Duckietown in GUI showing Duckiebot _Harpy’s_ current location.</figcaption>
</div>

[GUI with assigned customer](#fig:fleet-planning-map-with-icons).

<div figure-id="fig:fleet-planning-map-with-icons">
   <img src="gui_2.png" width="25em"/>
   <figcaption>Map with icons for a customer at node 7. Duckiebot “Harpy”’s target location is also at node 7 to pickup the customer.</figcaption>
</div>

[GUI with assigned customer](#fig:fleet-planning-happy-travelling).

<div figure-id="fig:fleet-planning-happy-travelling">
   <img src="gui_3.png" width="25em"/>
   <figcaption>Harpy travelling with the customer to the target location.</figcaption>
</div>


#### Duckietown Map



- A map of the current Duckietown as an image, received from the
   drawing node
   - Displays a selected Duckiebot’s location
   - Displays the start and target location of the user’s last issued request once the start button (4) has been hit
   - Displays the calculated route between start and target locations
   - The user can intuitively set start and target location by simply clicking on the map
   - The first click sets the start location
   - The following click sets the target location
   - Both can be cleared by clicking the clear button (5)



#### Display of start and target location


Serves to provide the user with feedback on the state that the GUI is in
Also provides a way to check correctness of the start/target location before issuing a request



#### List of active Duckiebots


The selected Duckiebot’s location is displayed on the map (1)
Used for testing and debugging during development
List received as ROS message from the taxi central node

#### Start button


Triggers a customer request to the taxi central node
Only triggered if start and target location are set

#### Clear button


Clears the start and target locations, which are then removed from the map (1) as well as the numerical display (2)



### Map drawing

Code can be found [here](github:org=duckietown,repo=Software,path=20-indefinite-navigation/fleet_planning/src/map_draw_node.py).



The map drawing node deals with drawing the Duckietown map according to the specifications in the csv file, overlaying the graph on top of the map and drawing the active Duckiebots at the correct locations. As localization only occurs at intersections where Apriltags are located, Duckiebots are only ever drawn at intersections. The Duckiebot is identified by it’s name, displayed below the Duckiebot icon.

When a customer request is assigned to a Duckiebot, a customer icon is drawn at the specified location and an icon is displayed at the target location as well. Once the customer is picked up, his or her icon is drawn alongside the Duckiebot acting as a taxi. See screenshots above for the different states.



### Messaging / Serialization


As described in a previous section, the existing system runs completely on the Duckiebot, including the user GUI. To make the system scalable we needed to have communication between multiple Duckiebots as well as a central planning node. This change in the architecture requires a communication system for reliable communication. We acquired this functionality by setting up a contract with the fleet communication team. See [here](#fleet-planning-final-report-contracts).



The outcome was a system which consists of one ROS node on each participant in the network. Via a configuration file you can define which node communicates with which other node. The interface the fleet communication team provides accepts a byte array and transfers this byte array to the endpoint of the communication channel. For a more detailed explanation of how this is transferred we refer readers to their final report

To send data such as target locations and localization results a way to serialize this data to a byte array was needed. A general framework was setup to serialize the basic data types using python’s pickle [3] module. Based on this we implemented serializer and deserializer classes specifically for the messages we needed to send over the network.



### Virtual Duckiebot / Simulation


Only at the very end of the project all projects we depend on reached a state where we could integrate them all to have a functional system. Therefore we needed a way to test the system, especially the central dispatcher node, without relying on physical Duckiebots. We solved this by implementing a virtual Duckiebot ROS node that acts as if it were a real Duckiebot.

Duckiebots mainly perform two actions, they report their location to the central dispatcher node and they receive commands (customer pickup, target location, …). The virtual Duckiebot node mimics this behaviour by regularly sending a message with the current position. Additionally it prints all the information it receives and sends to the console for easier debugging. To the central dispatcher node it looks as if it were interacting with a real Duckiebot.

The virtual Duckiebot node can be run in one of two ways:


#### Manual mode:


The virtual duckiebot is started with an initial location. The user uses a ROS service call provided by the virtual Duckiebot to tell it to send a location update message. The user can specify the location it should send. This mode gives power to the user to test specific situations.


#### Autonomous mode:


In this mode, the user only adds a Duckiebot at a desired node. The virtual Duckiebot node then takes care of all the things a real Duckiebot would do: it receives target locations from the taxi central, calculates the path it should take, notifies the taxi central of the calculated path, and then every few seconds it publishes a location update, as if it were really following the calculated path. The user can add as many Duckiebots as desired and can remove them as well. This way, the whole fleet planning software can be tested and simulated from a single laptop, without the need of a Duckietown or a Duckiebot.  



## Formal performance evaluation / Results


As mentioned previously, the largest portion of the work that needed to be done involved implementing an operational infrastructure that supports actual fleet planning functionality. 

In summary, this included:



- Generating a Duckietown map from a CSV description of the map
   Tracking n Duckiebots on the map, and displaying their locations,
   their targets and their paths
  - Creating customer requests through the
   GUI by clicking on the image of the map directly, thus removing the
   need to use tedious drop-down menus
   - Tracking customer requests and their assignments through completion (useful classes for easy debugging and clean coding)
   - Visualization of the taxi status directly on the Duckiebots via LEDs
   - A virtual Duckiebot node, that simulates a real Duckiebot in order to test the fleet planning software.



These functionalities can primarily be evaluated in a binary manner, observing whether these components work or not. Furthermore, the overall performance can be analyzed in a qualitative manner.
The components described above all work in the set up used for the 2017 ETH demo day. Our fleet planning system is robust to malfunctions in the execution of the planned path (i.e. the system can recompute paths and is tolerant towards situations where Duckiebots deviate from the optimal trajectory). Furthermore, the loss of a Duckiebot during the transport of a customer is covered by assigning a new, nearby Duckiebot to pick up the customer from where he or she was last seen.


In comparison to the state-of-the-art, where no fleet planning was possible and visualization of only a single Duckiebot on a map was implemented, a large number of new features was added and the infrastructure put in place to develop more advanced re-balancing algorithms.



Our system is relatively high-level in the sense that it requires many other Duckiebot components to work smoothly for successful operation. For example, we need the Duckiebots to reliably find the correct Apriltag at an intersection and the turns at an intersection to work flawlessly. The principal components our project requires are lane following, intersection navigation, collision avoidance, localization and fleet communication.
While our system simply ignores malfunctioning Duckiebots and drops them from the roster of active Duckiebots, it is nonetheless important to ensure most Duckiebots work as expected. On a map the size of the 2017 demo day collisions are inevitable if Duckiebots do not stay in their lane and relatively quickly manual intervention using a joystick becomes necessary. Of course, during manual operation of a Duckiebot the fleet planning system cannot meaningfully assign customers to Duckiebots. Thus, if manual control becomes necessary too often, fleet planning cannot do its job.



At the time of the demo day most other projects of this year that we relied on were not yet ready so we resorted to using last year’s implementations for some components, notably (open-loop) intersection control, lane following as well as last year’s FSM.



What this meant is that very often the system did not function smoothly and manual intervention was often necessary when Duckiebots missed a turn or lane following abruptly stopped working. Coupled with mercurial joysticks this made testing of new features challenging at times. Initially, we had expected to spend far less time on integrating previous features and get the all running in parallel. Instead, we had thought we would need more time advancing the capabilities of the fleet planning package itself. In the end, a basic Duckiebot that could navigate and communicate with reasonable reliability was paramount to doing any development on our package.



Quantitative evaluation of the sort initially planned and described in the PDD did not lead to any sensible results and insights. The metrics suggested were the ‘customer requests fulfilled per minute for given number of Duckiebots’ and the ‘mean distance of closest Duckiebot to the origin of a request for a set of requests and a given number of Duckiebots’. Both require a large map, a large number of Duckiebots and, most importantly, a certain amount of reproducibility in the results. As Duckiebots frequently veered off the side of the road or collided with sign posts, the same experiment could not be repeated in a meaningful way and the proposed metrics did not make a lot of sense. Nonetheless, once this year’s projects have all been merged and a greater overall stability in the system is achieved these metrics would provide a useful method of evaluation.



## Future avenues of development


As seen in the previous sections we were able to provide a framework for a fleet level planning system within Duckietown which can serve as a basis for interesting research questions. However, it goes without saying that the current state of the system has some of room for improvement. This sections lists possible extensions and improvements.



### Integration with other improvements from 2017


As all the teams were working on their projects in the same time frame with unclear finishing dates we made the decision in mid-January to use lane following and intersection maneuvering from the duckietown class of 2016. Therefore the performance of the system is not as good as it could be with the improved implementation of these two components. Integrating these two improved components into the system to replace the old ones would be a low effort, high gain development.



### Distributed fleet planning


The fleet planning system currently requires the taxi central node to run on a computer in the same network as all the Duckiebots. This is a single point of failure. If either communication between the Duckiebots and that computer breaks down or the computer itself fails the whole system fails. This is a very undesirable property.
The fleet communication system not only supports communication with a central node but also point to point communication between the Duckiebots. Therefore it is possible to implement the system in a decentralized way. One possible implementation strategy:

- New customer requests are broadcasted in the network using a flooding algorithm
- Available Duckiebots close to the customer propose that they pick him up
- The duckiebots that broadcasted a proposal reach consensus using an algorithm such as Cheap Paxos [4]
- The decision is again flooded in the network such that every Duckiebot can accordingly update its own knowledge of the system.


Under the assumption of a connected network (i.e. no partitions) such a system is able to achieve the same performance as a system with a centralized node that coordinates all the Duckiebots. However, the implementation of such algorithms is more demanding because of  the increase of complexity in the system which makes it harder to debug.

### Switch to Mesh Network Communication

The system as is requires a router for passing the messages in the network around. This can be a standard wifi router or a mobile phone used as a hotspot. This is additional hardware that is required to run the system and needs to be setup the right way. I.e. further points of a possible error while running the system. The library from the fleet communication team which we already use for communication also supports a mesh network configuration. In this configuration each Duckiebot uses its wifi stick to create the same Wifi network. The duckiebots can then communicate using this network. Therefore, no additional hardware is needed. The change to enable this kind of network communication is rather small. However, it has not yet been tested. So further development could include the inclusion and testing of the mesh network configuration.

### Parking of unused Duckiebots

In a real world scenario of an autonomous taxi system, the demand for vehicles changes over the course of a day, i.e. there are peaks around the morning commute time as well as the evening. One kind of optimality is to only have as many vehicles on the road as needed based on the current demand. Obviously this should be combined with a prediction of the future demand to minimize waiting time for the customer. (The interested reader is referred to [5] and [6] for a more thorough analysis of fleet size and rebalancing strategies).

The current implementation of the system forces all Duckiebots to continuously drive around the city. If they are not fulfilling a customer request they are driving around according to the currently activated rebalancing strategy (random by default). This is not perfectly efficient. Using the outcome from the parking team the two systems can be combined to allow dynamic resizing of the currently active fleet by parking vehicles that are currently not needed.

### Implementation and evaluation of rebalancing strategies

Unused vehicles drive to locations according to a rebalancing strategy. The default rebalancing strategy is “random” and sends the vehicles to random locations. The software architecture allows to easily implement further strategies and use them within the system. As a further development more rebalancing strategies can be implemented and evaluated for their performance in Duckietowns of different sizes. The reader is referred to [6] for a list of possible rebalancing strategies that can be implemented.

### Location estimation and visualization between intersections

The current implementation updates the location of the Duckiebots only at intersections. Using wheel encoder information the locations could be estimated in between intersections and thus deliver a more fluid user interface and allow customer pick up in between intersections. A more high powered version might use SLAM to localize Duckiebots at any given point in time, allowing for more fine-grained localization and consequently better fleet planning.

### Online Map Generation

The current implementation of the MOD system depends on a map of the Duckietown generated a priori. It can be extended with the SLAM functionality implemented by the team in Montreal. Using the map that’s generated while traversing the map would remove the step of manually creating the map and thus make the system more user friendly.


## Conclusion {#fleet-planning-report-conclusion}

In summary, the fleet planning project at current allows for the high-level control of a large number of Duckiebots, the visualization of the duckiebots on the map in a GUI, the assignment of customer requests an the execution of taxi services. The system works but relies heavily on smooth functioning of other components and is only as robust as these components are.
The Duckiebot classes are extensively documented and designed in a way that allows easy extension with different fleet planning and rebalancing algorithms. This paves the way for future updates, some of which were discussed in the previous section.

## References

[1] M. Pavone, K. Treleaven and E. Frazzoli, "Fundamental performance limits and efficient policies for Transportation-On-Demand systems" 49th IEEE Conference on Decision and Control (CDC), Atlanta, GA, 2010, pp. 5622-5629.  

[2] W. Galitz , “The essential guide to user interface design: an introduction to GUI design principles and techniques” John Wiley and Sons, 2007.            

[3] https://docs.python.org/2/library/pickle.html, Accessed: February 2017

[4] L. Lamport and M. Massa, "Cheap Paxos," International Conference on Dependable Systems and Networks, 2004, pp. 307-314.

[5] K. Spieser, K.Treleaven, R. Zhang, E. Frazzoli, D. Morton and M. Pavone, "Toward a Systematic Approach to the Design and Evaluation of Automated Mobility-on-Demand Systems A Case Study in Singapore" Chapter in Road Vehicle Automation, Gereon Meyer, Sven Beiker (Editors). Berlin: Springer, 2014, pp.229-245

[6] Pavone, M., S. L. Smith, E. Frazzoli, and D. Rus, “Robotic load balancing for mobility-on-demand systems.” The International Journal of Robotics Research 31, no. 7
