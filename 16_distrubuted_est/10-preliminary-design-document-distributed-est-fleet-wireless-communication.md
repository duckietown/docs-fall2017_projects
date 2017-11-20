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

<div class="example-usage" markdown="1">

</div>

<div class="example-usage" markdown="1">

</div>

<div class="example-usage" markdown="1">

</div>


### Resources required / dependencies / costs

**Bandwidth definition:**

* Number of duckiebots in the network
* Size of the messages
* Sending frequency
* Latency in message transmission
* Extra computation on duckiebots

<div class="example-usage" markdown="1">

</div>

<div class="example-usage" markdown="1">

</div>

<div class="example-usage" markdown="1">

</div>

<div class="example-usage" markdown="1">

</div>

### Performance measurement

* Visualize the network topology → number of duckies
* Visualize messages (wireshark) → message size, latency
* Visualize HW resources → processor, memory, etc.

<div class="example-usage" markdown="1">

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

# Part 4: Project planning

## Project plan

<col3 figure-id="tab:environment-variables" class='labels-row1'>
    <figcaption>Environment variables used by the software</figcaption>

    <s>variable</s>
    <s>reasonable value</s>
    <s>contains</s>

    <s><code>DUCKIETOWN_ROOT</code></s>
    <s><code>~/duckietown</code></s>
    <s><code>Software</code> repository</s>

    <s><code>DUCKIEFLEET_ROOT</code></s>
    <s><code>~/duckiefleet</code></s>
    <s>Where to look for class-specific information (people DB, robots DB).</s>

    <s><code>DUCKIETOWN_DATA</code></s>
    <s><code>~/duckietown-data</code></s>
    <s>The place where to look for logs.</s>

    <s><code>DUCKIETOWN_TMP</code></s>
    <s></s>
    <s>If set, directory to use for temporary files. If not set, we use the default ( <code>/tmp</code>).</s>

    <s><code>DUCKIETOWN_CONFIG_SEQUENCE</code></s>
    <s><code>defaults:baseline:vehicle:user</code></s>
    <s>The <a href="#easy_node">configuration sequence for EasyNode</a></s>
</col3>

#### Week: 13/11/17
  1. Task:
  * Project kick-off and planning 
  * Deliverables: 

## Task distribution
  * Libraries (incl. Testing): Luca and Antoine
  * ROS node (incl. Testing): Leonie
  * Testing of the framework: Pat and Francesco
  * Redundant centralized network: Pat and Francesco
  * Ad-hoc networking: Leonie, Luca and Francesco

## Data collection
TBD --> other groups

## Relevant Duckietown resources to investigate

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
