#  Fleet Communication: final report {fleet-messaging-final-report status=draft}

## The final result {fleet-messaging-final-result}

![Demo Video](https://github.com/duckietown/duckuments/blob/devel-distribution-est-fleet-wireless-communication/docs/atoms_85_fall2017_projects/16_distrubuted_est/Demo%20Video.ogv)

see the [operation manual](#demofleet-messaging) to reproduce these results.

![README](https://github.com/duckietown/Software/blob/devel-distributed-est-master/catkin_ws/src/30-localization-and-planning/README.md)

## Mission and Scope {fleet-messaging-final-scope}

With this project we enable Duckiebots to communicate with each other on centralized and decentralized wireless network.

### Motivation {fleet-messaging-final-result-motivation}

In the previous state of Duckietown, Duckiebots were individual, autonomous agents, roaming around Duckietown with no way to communicate with each other explicitly (the only method in existence is with the help of LED patterns, resulting in long interpretation times). Since the ultimate goal being an automated taxi system: Duckiebots working together picking up and dropping off customers in the optimal way; the Duckiebots needs to be able to communicate with each other efficiently.

One important part of this communication setup is that it can be decentralized, using a mesh network and Duckiebots can join and leave the system without putting the whole network at risk of failing. This also allows for the network to be scaled, given network limitations.

Due to the current state of Duckietown, the communication is needed, but not limited to, fleet planning control to coordinate the fleet in a town with a predefined map.

### Existing solution {fleet-messaging-final-literature}
There was no prior work to build a communication system upon. Everything was implemented from scratch.

### Opportunity {fleet-messaging-final-opportunity}
Without any existing work on wireless communication, we came up and build a whole new addition to Duckietown. We implemented a fleet-messaging package that builds an ad-hoc mesh network and lets other teams define their message types and sends them over the created network. 

Remark: It is important to know that the  is not stable everywhere yet. There were some driver problems with some WiFi adapters with respect to mesh network capabilities. It works with the edimax, so it might be of advantage to have two edimax adapters: one for the duckiebot and one for the laptop. With this setup, the edimax adapters can be used to create the mesh network (and the connected laptops would be a part of the network as well). It is also important to know that at this moment, it is not possible to get a connection to the internet through the duckiebot via the mesh network.

### Preliminaries (optional) {fleet-messaging-final-preliminaries}
We specifically picked libraries and modules that encapsulates their respective functionalities well. Therefore to fully understand what is going on under the hood, you simply need to read up on the documentation of each package used:
- [batman-adv](https://www.open-mesh.org/projects/batman-adv/wiki/Wiki)
- [zeroMQ](http://zeromq.org/)
- [protobuf](https://developers.google.com/protocol-buffers/)

## Definition of the problem {fleet-messaging-final-problem-def}

The final goals of the project were to:
1. Create a robust wireless network that can easily be scaled to a larger fleet size and to a bigger Duckietown.
2. Build a communication framework for the Duckiebots that enables the sending and receiving of messages to and from any Duckiebot, which is connected to the above mentioned network. 
3. Have a communication framework that is adjustable and is not limited to a single message type.

For this we made the following assumptions:
1. Duckiebots can connect to a wifi network.
2. Duckiebots leave network when they are out of range or switched off.
3. If a Duckiebot is not connected to the communication network, it is not in Duckietown.
4. In a first step, the communication network is used by the fleet-planning team. 

To evaluate the new framework we:
1. Compared the messages sent and received between two Duckiebots connected over the network and looked for messages dropped
2. Tested the range of the wifi adapters to see if it is able to cover the size of a demo-sized Duckietown
3. Test the robustness of the network by taking a Duckiebot out of range of the network and back and restarting the Duckiebot in to see if it would reconnect.

## Contribution / Added functionality {fleet-messaging-final-contribution}

### Added Functionailities
Added Functionalities are as follow:
- Duckiebots are now able to communicate with one another directly without a central station to relay the messages. 
- The network has no centralized point of failure. If a Duckiebot fails (runs out battery, breaks, disconnects abruptly) for any reason, the rest of the Duckiebots remaining in the system are still able to communicate without any noticeable disruption, given they are in range of each other.

Consequently, we believe we have achieved the gold medal outcome for this project. 

### Package Infrustructure

The package infrastructure is as follows:
1. A mesh network that dynamically connects all of the Duckiebots that are currently in Duckietown.
2. A messaging algorithm that allows the sending and receiving of messages over this network.
3. A message encoder that packs the data before it is sent.
4. A message decoder that unpacks the data after it is received.
5. A framework that allows other Duckietown packages to send and receive messages of their defined types.

These individual parts were implemented as described below.

#### Mesh Network
Batman-adv is the backbone of the mesh network. In short, it is a specialized linux kernel module that implements a network routing protocol. It emulates a virtual network switch of all nodes participating. Hence, all nodes appears to be linked locally and are unaware of the network's topology and is also unaffected by any network changes. Once setup properly, batman-adv manages the mesh network for us.

#### Messaging Algorithm
DuckieMQ is based on zeroMQ, a framework used to send messages over sockets (&FEF: small technical correction). The serialized messages (protobuf) are broadcasted on a specified port into the network.  For this to work, we need to know the name of the network interface, the desired port initialization of a messaging socket, which then can either be used as receiver or sender. Multiple sockets can and usually will run on one bot. Also because we use a multicast protocol (epgm) multiple sender and recieiver sockets can run on one port. 

Moreover, messaging features of the platform is decoupled from the implementation of the network achitecture.

#### Message encoder
In order for the messages to be sent and received, they have to be serialized. Therefore, a serialization library was implemented to serialize [ByteMultiArrays](http://docs.ros.org/jade/api/std_msgs/html/msg/ByteMultiArray.html). After the message is sent, the data is then parsed back into a ROS message and published to the correct inbox_topic specified by the package that sent the message.
We chose to use ByteMultiArray for its flexibility and because it is a `std_msg` of ROS. This means that other packages must only publish ByteMultiArray to fleet messaging.

#### Framework
For easy use of the messaging algorithm a ROS package, with two ROS nodes was implemented. The two nodes are the receiver_node and the sender_node. 

The sender_node subscribes to the outbox_topic and sends this data to the receiver_node on all other Duckiebots on the network via the messaging algorithm using zeroMQ. The receiver_node then publishes the received data to the inbox_topic .

To use the framework, one simply has to publish to the ROS topic outbox_topic (specified by the config file) and subscribe to the inbox_topic (also specified by the config file) and listen to the specified message port.

The complete structure of the fleet-messaging package is illustrated below.
![System Infrustructure](https://github.com/duckietown/duckuments/blob/devel-distribution-est-fleet-wireless-communication/docs/atoms_85_fall2017_projects/16_distrubuted_est/Simple%20Fleet%20Messaging%20Flow%20Diagram.png "System Infrustructure")



## Formal performance evaluation / Results {fleet-messaging-final-formal}

There are three main criterion that have to be evaluated:
1. Message transport:
    1. Centralized Network: test if a simple message e.g. a string can be sent from one duckiebot to another reliably. 
    2. Decentralized Network: test message propagation. In a mesh network two Duckiebots (nodes) may not be directly connected, therefore we must test if a message can be propagated through the network to the correct receiver.
    3. Check of dropped messages.
2. Network traffic: accurately monitor network traffic.
3. Network topology: visualize nodes entering and leaving the network reliably.

To test the first criteria: 
- Compared the messages sent and received between two Duckiebots connected over the network and looked for messages dropped

To test the second criteria:
- Analyse packets sent on wireshark

To test the third criteria:
- Tested the range of the wifi adapters to see if it is able to cover the size of a demo-sized Duckietown
- Test the robustness of the network by taking a Duckiebot out of range of the network and back and restarting the Duckiebot in to see if it would reconnect.
 
Our conclusions are summarized in the following table

| |0|1|2|3|4|
|---|---|---|---|---|---|
|Message Transport  | Messages cannot be sent or received | Strings can be sent and received on tcp | Messages can be serialized and sent and received on tcp | Messages can be serialized and multicast and received on pgm |**Messages can be serialized and multicast and received on pgm on a mesh network**|
|Network Traffic|No traffic monitored|**Can see traffic on the network but no useful information extracted**|Able to isolate duckiebot traffic|Able to identify specific packets|Able to visualize routing of specific packages|
|Network Topology (centralized and decentralized)|Network cannot be established|Initial Network can be established, but no new nodes can connect to the network, not robust to connection loses|Initial Network can be established, new nodes can connect but not reliably, not robust to connection loses|Initial Network can be established, new nodes can connect/leave dynamically, but not robust to connection loses|**Initial Network can be established, new nodes can connect/leave dynamically, and robust to connection loses**|

## Future avenues of development {fleet-messaging-final-next-steps}

### One push solution for package setup/installation

#### Current Issue
Mesh networking can be very finicky because it depends on drivers for the wifi adapters and batman-adv working correctly. From experience, even with the unified Duckiebot hardware this it was very much a case by case basis. This made it difficult to develop a one push solution. Nonetheless we implemented one - see operation manual - which seemed to work on most occasion but we still had to do some on the spot debugging.

#### Possible Solution
Use wireless adapters where the mesh network is currently working (edimax). A refined the bash script already implemented.

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
We discover late into the project that the edimax has mesh capabilities. We tried it and knows that it worked but never fully tested to a point that we were confident with its viability. 

### Improving usability of platform

ZeroMQ allows to filter messages according to strings (eg. botname) at the start of messages. Our duckieMQ implementation already supports filtering it has however not been used yet by the teams as the whole set up gets a bit cumbersome if it has to be implemented on every bot. Also serialized messages need to be manually prepended by the filterstring. It would be useful to extend duckieMQ with the capability to prepend a filterstring to serialized messages automatically.

The configuration files for the different channels are a bit cumbersome to create for bigger groups of bots. It would be nice if the bot would create its own config file according to rules laid down in a central config file where all the different groups specify their communication architecture. 

E.g fleet level planning want every bot to listen to port 23334 filter it with their name and publish to /taxi/commands and subscribe to /taxi/location and send it on port 23333.

As a final step we could let the config file generator handle ports by itself, so no one needs to keep track of used ports

### Network Visualization

#### Current Issue
With the current implementation there is no way to visualize the topology of the network.

#### Possible Solution
A very useful function to implement would be to implement a real time visualization of the network. To visualize the network involves installing batadv-vis. Batadv-vis can be used to visualize the batman-adv mesh network. It reads the neighbor information and local client table and distributes this information via alfred - a user-space daemon for distributing arbitrary local information over the mesh/network in a decentralized fashion - in the network. By gathering this local information, any vis node can get the whole picture of the network. But this would have only taken us half the way there as it only gave static snapshots of the network. So to improve on this would be to continuously update/generate the graph so it appears to be live.
