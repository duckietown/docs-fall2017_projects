#  Fleet Communication: final report {fleet-communication-final-report status=draft}

## The final result {fleet-communication-final-result}


![Demo Video](https://github.com/duckietown/duckuments/blob/devel-distribution-est-fleet-wireless-communication/docs/atoms_85_fall2017_projects/16_distrubuted_est/Demo%20Video.ogv)

Add as a caption: see the [operation manual](#demofleet-communication) to reproduce these results.

## Mission and Scope {fleet-communication-final-scope}

With this project we enable Duckiebots to communicate with each other wirelessly without any centralized hardware.

### Motivation {fleet-communication-final-result-motivation}

In the previous state of Duckietown, Duckiebots were individual, autonomous agents, roaming around Duckietown with no way to communicate with each other. With the end goal being an automated taxi system; Duckiebots working together picking up and dropping off customers in the optimal way, the Duckiebots need to be able to communicate with each other.

One important part of this communication setup is that it needs to be decentralized and Duckiebots can join and leave the system without putting the whole network at risk of failing. This also allows for the network to be scaled, given network limitations.

Due to the current state of Duckietown, the communication is needed, but not limited to, multi-SLAM  and fleet planning control to use the entire fleet to build a map of Duckietown without prior knowledge and then to coordinate the fleet in this town.


### Existing solution {fleet-communication-final-literature}
There was no prior work to build a communication system upon. Everything was implemented from scratch.

### Opportunity {fleet-communication-final-opportunity}
Without any existing work on wireless communication, we came up and build a whole new addition to Duckietown. We implemented a fleet-communication package that builds an ad-hoc mesh network and lets other teams define their message types and sends them over the created network.

### Preliminaries (optional) {fleet-communication-final-preliminaries}
We specifically picked libraries and modules that encapsulates their respective functionalities well. Therefore to fully understand what is going on under the hood, you simply need to read up on the documentation of each package used:
- [batman-adv](https://www.open-mesh.org/projects/batman-adv/wiki/Wiki)
- [zeroMQ](http://zeromq.org/)

## Definition of the problem {fleet-communication-final-problem-def}

The final goals of the project were to:
1. Create a robust wireless network that can easily be scaled to a larger fleet size and to a bigger Duckietown.
2. Build a communication framework for the Duckiebots that enables the sending and receiving of messages to and from any Duckiebot, which is connected to the above mentioned network. 
3. Have a communication framework that is adjustable and is not limited to a single message type.

For this we made the following assumptions:
1. Duckiebots can connect to a wifi network.
2. Duckiebots leave network when they are out of range or switched off.
3. If a Duckiebot is not connected to the communication network, it is not in Duckietown.
4. In a first step, the communication network is used by the fleet-planning team and the multi-robot SLAM team. 

To evaluate the new framework we:
1. Compared the messages sent and received between two Duckiebots connected over the network and looked for messages dropped
2. Tested the range of the wifi adapters to see if it is able to cover the size of a demo-sized Duckietown
3. Test the robustness of the network by taking a Duckiebot out of range of the network and back and restarting the Duckiebot in to see if it would reconnect.

## Contribution / Added functionality {fleet-communication-final-contribution}

### Added Functionailities
Added Functionalities are as follow:
- Duckiebots are now able to communicate with one another directly without a central station to relay the messages. 
- The network has no centralized point of failure. If a Duckiebot fails (runs out battery, breaks, disconnects abruptly) for any reason, the rest of the Duckiebots remaining in the system are still able to communicate without any noticeable disruption, given they are in range of each other.

Consequently, we believe we have achieved the gold medal outcome for this project. 

### Package Infrustructure

The package infrastructure is as follows:
1. An ad-hoc mesh network that dynamically connects all of the Duckiebots that are currently in Duckietown.
2. A messaging algorithm that allows the sending and receiving of messages over this network.
3. A message encoder that allows the definition of new message types and serializes the data before it is sent.
4. A framework that allows other Duckietown packages to send and receive messages of their defined types.

These individual parts were implemented as described below.

#### Ad-hoc Mesh Network
Batman-adv is the backbone of the mesh network. In short, it is a specialized linux kernel module that implements a network routing protocol. It emulates a virtual network switch of all nodes participating. Hence, all nodes appears to be linked locally and are unaware of the network's topology and is also unaffected by any network changes. As a result, batman-adv allows for us to develop our communication platform agnostic to the underlying network architecture.

#### Messaging Algorithm
DuckieMQ is based on zeroMQ, it facilitates the sending and receiving of messages. The serialized messages (protobuf) are broadcasted on a specified port into the network. For this to work, we need to know the name of the network interface, the desired port and whether we want to recieve or send on initialization of a messaging socket. Multiple sockets can run on one bot and on also on one port. Receivers can be equipped with filters to only receive messages starting with a specified string.

Once these sockets are created (initialize either sender or receiver) messages can be passed. The receiver node is equipped with a “” filter, meaning it receives all messages. If only messages starting with a or multiple fiter want to be received, these filter strings have to be added and the “” filter string has to be removed with the addfilter and removefilter functions.

#### Message encoder

*TODO*

#### Framework
For easy use of the messaging algorithm a ROS package, with two ROS nodes was implemented. The two nodes, are the reciever_node and the sender_node. 

The sender_node subscribes to the outbox_topic and sends this data to the reciever_node on all other Duckiebots on the network via the messaging algorithm using zeroMQ. The reciever_node then publishes the received data to the inbox_topic.

To use the framework, one simply has to publish to the ROS topic outbox_topic and subscribe to the inbox_topic and listen to the specified message port.

The complete structure of the fleet-messaging package is illustrated below.
![System Infrustructure](https://github.com/duckietown/duckuments/blob/devel-distribution-est-fleet-wireless-communication/docs/atoms_85_fall2017_projects/16_distrubuted_est/Simple%20Fleet%20Messaging%20Flow%20Diagram.png "System Infrustructure")



## Formal performance evaluation / Results {fleet-communication-final-formal}

There are three main criterias that have to be evaluated:
1. Message transport:
    1. Centralized Network: test if a simple message e.g. a string can be sent from one duckiebot to another reliably. 
    2. Decentralized Network: test message propagation. In a mesh network two Duckiebots (nodes) may not be directly connected, therefore we must test if a message can be propagated through the network to the correct receiver.
    3. Check of dropped messages.
2. Network traffic: accurately monitor network traffic.
3. Network topology: visualize nodes entering and leaving the network reliably.


| |0|1|2|3|4|
|---|---|---|---|---|---|
|Message Transport  | Messages cannot be sent or received | Strings can be sent and received on tcp | Messages can be serialized and sent and received on tcp | Messages can be serialized and multicast and received on pgm |Messages can be serialized and multicast and received on pgm on a mesh network|
|Network Traffic|No traffic monitored|Can see traffic on the network but no useful information extracted|Able to isolate duckiebot traffic|Able to identify specific packets|Able to visualize routing of specific packages|
|Network Topology (centralized and decentralized)|Network cannot be established|Initial Network can be established, but no new nodes can connect to the network, not robust to connection loses|Initial Network can be established, new nodes can connect but not reliably, not robust to connection loses|Initial Network can be established, new nodes can connect/leave dynamically, but not robust to connection loses|Initial Network can be established, new nodes can connect/leave dynamically, and robust to connection loses|

## Future avenues of development {fleet-communication-final-next-steps}

<<<<<<< HEAD
### One push solution for package setup/installation

#### Current Issue
Mesh networking can be very finicky because it depends on drivers for the wifi adapters and batman-adv working correctly. From experience, even with the unified Duckiebot hardware this it was very much a case by case basis. This made it difficult to develop a one push solution. Nonetheless we implemented one - see operation manual - which seemed to work on most occasion but we still had to do some on the spot debugging.

#### Possible Solution
Refined the bash script already implemented.

### Improve Developmental setup

#### Current Issue
When developing this package we needed three networks running simultaneously: one connected to the internet (for git purposes), one connected to the local network created by the Duckiebot (for ssh), and lastly the mesh network itself. 

_Side note: Technically you can ssh into the Duckiebot through the mesh network, however this becomes problematic with you want to debug the mesh network itself. If your mesh network doesn’t work then you can ssh into your Duckiebot anymore. That is why it is better - at least when developing - to have three different networks running._

#### Possible solution
There are already two wifi adapters on the current configuration, and there lies the problem.
There are two workarounds in use currently:
1. Use a centralized network created by an access point. This allows you to both ssh and develop your communication platform. However, it is no longer a decentralized mesh network.
2. Use an additional, third (mesh capable) network adapter strictly for mesh networking. There are several drawbacks. Firstly, this means that you need an additional wifi adapter the Duckiebot totalling up to three. Secondly, you also need on for your laptop to connect to mesh network if don’t want to use your in-built adapter.

Both workarounds have their drawbacks, so it would be nice to find a robust solution for laptops connecting to the mesh network.

### Reducing the number of additional hardware

#### Current Issue
Following from the last point, adding an additional wifi adapter is costly

#### Possible Solution
We discover late into the project that the edimax has mesh capabilities but we never explored it properly.

### Improving usability of platform

*TODO*

### Network Visualization

#### Current Issue
With the current implementation there is no way to visual the topology of the network.

#### Possible Solution
A very useful function to implement would be to implement a real time visualization of the network. To visualize the network involves installing batadv-vis. Batadv-vis can be used to visualize the batman-adv mesh network. It reads the neighbor information and local client table and distributes this information via alfred - a user-space daemon for distributing arbitrary local information over the mesh/network in a decentralized fashion - in the network. By gathering this local information, any vis node can get the whole picture of the network. This would have only taken us half the way there as it only gave static snapshots of the network. So to improve on this would be to continuously update the graph so it appears to be live.
=======
One push solution for package setup/installation:

Mesh networking can be very finicky because it depends on drivers for the wifi adapters and batman-adv working correctly. From experience, even with the unified Duckiebot hardware this it was very much a case by case basis. This made it difficult to develop a one push solution. Nonetheless we implemented one - see operation manual - which seemed to work on most occasion but we still had to do some on the spot debugging. If this we refined more it would make using this package almost effortless. 

Development setup:

When developing this package we needed three networks running at the same time: one connected to the internet (for git purposes), one connected to the local network created by the Duckie bot (for ssh), and lastly the mesh network itself.

In principle there are already two wifi adapters on the current configuration, however in order to keep connection to the internet one of those has to be connected to an internet connected network.
There are two workarounds in use currently.
Use a centralized network created by an access point (not decentralized anymore)
Use an additional, third (mesh capable) network adapter strictly for bot-bot and bot-pc communication (higher cost)

Both workarounds have their drawbacks, so it would be nice to find a robust solution for laptops connecting to the mesh network. We discover late into the project that the edimax has mesh capabilities but we never explored it enough to use it reliably.



The configuration files for the different channels are a bit cumbersome to create for bigger groups of bots. It would be nice if the bot would create its own config file according to rules laid down in a central config file where all the different groups specify their communication architecture.
E.g fleet level planning want every bot to listen to port 23334 filter it with their name and publish to /taxi/commands and subscribe to /taxi/location and send it on port 23333.
 As of now the config files do not support filters.

For filtering of serialized messages it would be useful to extend duckieMQ with the capability to preappend a filterstring to serialized messages.



Network Visualization: A very useful function to implement would be to implement a real time visualization of the network. To visualize the network involves installing batadv-vis. Batadv-vis can be used to visualize the batman-adv mesh network. It reads the neighbor information and local client table and distributes this information via alfred - a user space daemon for distributing arbitrary local information over the mesh/network in a decentralized fashion - in the network. By gathering this local information, any vis node can get the whole picture of the network. This would have only taken us half the way there as it only gave static snapshots of the network. So to improve on this would be to continuously update the graph so it appears to be live.

>>>>>>> 31fbf58ec968f955a7c70375d55d56287f16842d
