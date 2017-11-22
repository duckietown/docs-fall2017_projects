#  Distributed Estimation: preliminary design document {#fleet-wireless-communication-preliminary-design-doc status=ready}

## Part 1: Mission and scope

### Mission statement

Enable Duckiebots to communicate with each other wirelessly

### Motto

BI UNUM IBI OMNES (Where there is one, there is everybody)

### Project scope

#### What is in scope

**Communication in a centralized network:**
* Communication between Duckiebots in one Duckietown
* Define interfaces for communication
* Performance testing on the communication system
* One network for one Duckietown
* TBD: does anything need to be synchronized?

<div figure-id="fig:figure 1">
    <img src="https://github.com/duckietown/duckuments/blob/devel-distribution-est-fleet-wireless-communication/docs/atoms_85_fall2017_projects/16_distrubuted_est/Duckietown_Project_Image.png" />
    <figcaption id='fig:figure 1:caption'>
        System Layout
    </figcaption>
</div>


**Option 1:**
Communication in a centralized network with a redundant centralized component (multiple routers)

**Option 2:**
Communication in a de-centralized network (ad-hoc)

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
    * Live visualization of bandwidth (and/or latency, etc. → Network performance measures) of network
    * Live visualization of network topology 
    * Are any messages being dropped without arriving at their destination?
    * Etc.

**Step 2 (Optional):** Test the communication framework using a redundant centralized network

* Same as step 1, except for the wifi network → redundant centralized network

**Step 2 (Optional):** Test the communication framework using a de-centralized network

* Same as step 1, except for the wifi network → AD-HOC network

### Functionality-resources trade-offs

### Functionality provided

* Enable duckiebots to join the network.
* Enable duckiebots to pack data and send it to other duckiebots in the same network.
* Enable duckiebots to receive and unpack data such that the data can be used in other software modules.
* Other functionality TBD

### Resources required / dependencies / costs

**Bandwidth definition:**

* Number of duckiebots in the network
* Size of the messages
* Sending frequency
* Latency in message transmission
* Extra computation on duckiebots

### Performance measurement

* Visualize the network topology → number of duckies
* Visualize messages (wireshark) → message size, latency
* Visualize HW resources → processor, memory, etc.

## Part 3: Preliminary design

### Modules

**Libraries:**
* Serialization
* Messaging

**Integration:**
* Libraries integration into component 
* Communication framework into Duckietown (repository, duckumentation, etc. → contact Integration Heroes)
* Creation of the WiFi network (centralized vs. decentralized)

**Testing:**
* Measuring and visualization of performance metrics


### Interfaces

**Fleet planning**
* Send data (SLAM, etc.)
* Distribute data through the network 

**Distributed Estimation**
* Receive data (local maps built from each Duckiebots)
* Send data (local maps and/or global map)

**Integration Heroes**
* Duckumentation 
* Contract negotiation 
* External libraries (Protocol buffers, ZMQ, etc.)


### Preliminary plan of deliverables

### Specifications

* WiFi specs

### Software modules

* Library for Serialization
   * Inputs:
      * Data to be serialized
      * Data to be de-serialized
      * Type definitions for serialization → Contract with distributed-estimation and fleet-planning teams
   * Outputs:
      * Serialized data
      * De-serialized data
   * Functionality:
      * Serialize data
      * De-serialize data
* Library for Messaging
   * Inputs:
      * Destination
      * Port
      * Type of socket
      * Serialized data
      * Optional: priority
   * Outputs:
      * Serialized data
* ROS node for messaging
   * Inputs:
      * Messages from ROS topics
      * Messages from other duckiebots
   * Outputs:
      * Messages to ROS topics
      * Messages to other duckiebots

### Infrastructure modules

* Wifi router for Duckietown
   * Network configuration (centralized/decentralized)
   
* Testing
   * Visualize the network topology → number of duckies
   * Visualize messages (wireshark) → message size, latency
   * Visualize HW resources → processor, memory, etc.
   * ...



## Part 4: Project planning

### Project plan

#### Week 9: 13/11/2017
* **Tasks:**
    * Project kick-off and planning
  
* **Deliverables:** 
    * Preliminary Design Document

#### Week 10: 20/11/2017
* **Tasks:**
    * Contract negotiation with the relevant groups
    * Research on testing, redundant centralized and ad-hoc networking
    * Design and implementation of Libraries
    * Design and implementation of ROS node
    * Configuration of centralized network
  
* **Deliverables:** 
    * Contracts

#### Week 11: 27/11/2017
* **Tasks:**
    * Find suitable tools for testing and test the ROS node (incl. libraries) using these tools
    * Configuration of redundant centralized network
    * Code reviews
    * Duckumentation

* **Deliverables:** 
    * Network configuration working
    * Libraries (tested)
    * ROS node (tested)
    * First test results

#### Week 12: 04/12/2017
* **Tasks:**
    * Ad-hoc networking
    * Duckumentation

* **Deliverables:** 
    * Redundant centralized network working

#### Week 13: 11/12/2017
* **Tasks:**
    * Ad-hoc networking
    * Duckumentation
  
* **Deliverables:** 
    * Testing “framework” complete

#### Week 14: 18/12/2017
* **Tasks:**
    * Ad-hoc networking
    * Duckumentation
    * Solve networking problems
    * Testing

* **Deliverables:** 
    * None

#### Week "15" : 25/12/2017
* **Tasks:**
    * Ad-hoc networking
    * Duckumentation
    * Solve networking problems
    * Testing
  
* **Deliverables:** 
    * Communicating Duckiebots over ad-hoc network
  
#### Week "16" : 01/01/2018
* **Tasks:**
    * Duckumentation and Buffer
   
* **Deliverables:** 
    * Duckumentation

### Task distribution
* Libraries (incl. Testing): Luca and Antoine
* ROS node (incl. Testing): Leonie
* Testing of the framework: Pat and Francesco
* Redundant centralized network: Pat and Francesco
* Ad-hoc networking: Leonie, Luca and Francesco

### Data collection
TBD --> other groups

### Data annotation

No data to be annotated.

### Relevant Duckietown resources to investigate

WiFi specs (duckumentation)

### Other relevant resources to investigate

**Suggestions on Slack channel:**
1. [MAVLink (born for UAVs also used for other robots)](http://qgroundcontrol.org/mavlink/start)
2. [ROMANO (not so good...)](https://arxiv.org/abs/1709.07555) 
3. [DDS (standard for ROS 2.0)](http://portals.omg.org/dds/)

**Computer Networks:**

* Introduction to computer networks: http://intronetworks.cs.luc.edu/

**Serialization and Messaging:**

* Protocol Buffers: https://developers.google.com/protocol-buffers/
* ZeroMQ: http://zeromq.org/ 

**Google python coding style guide:**

* https://google.github.io/styleguide/pyguide.html 

**Redundant routers (rollover, cascading):**

* http://www.tomshardware.co.uk/forum/21591-43-design-redundant-wireless-network 
* https://www.linksys.com/ca/support-article?articleNum=132275 (to be verified)

**Static code analysis in python:**

* https://www.pylint.org/ 

**Pylint configuration:**

* https://stackoverflow.com/questions/29597618/is-there-a-tool-to-lint-python-based-on-the-google-style-guide 

### Risk analysis

**Possible Risks?**
* Network problems (ad-hoc: unstable network, low bandwidth, high latency, ...)
* No ad-hoc network solution
* Sizable amount of redundant data sent over wifi (chaos)
* Synchronization

**How to mitigate the risks?**
* Synchronization not part of networking → contract
* Contracts to prevent redundancy

