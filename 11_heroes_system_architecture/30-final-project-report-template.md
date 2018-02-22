#  The Heroes - System Architecture: final report {#heroes-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "heroes" in the chapter labels to your group label!
-->

System Architecture refers to the high-level conceptual model that defines the structure and behaviour of a system. There are different ways to get an insight into the architecture of a system, for example functional decomposition diagrams, package composition, or a Finite State Machine diagram.

## The final result {#heroes-final-result}
The System Architecture project helped to ensure that the Fall 2018 projects integrate into the existing system. The role of the System Architect was to identify and solve problems that arose during the project development and integration, as well as influencing the high-level design of the system.

In Duckietown, the Finite State Machine (FSM) diagram plays an important role in determining how the higher-level system behaves in different scenarios. The FSM defines which states the system can be in, and which functionalities must be active in which states. During the project, the need for development on the FSM arose. Below is the resulting updated Finite State Machine (FSM) diagram.

![FSM diagram](fsm_default.png)

## Mission and Scope {#heroes-final-scope}

The System Architecture project was not a clearly defined work package, but rather a responsibility to ensure smooth development and integration of the new projects into the existing system.

### Motivation {#heroes-final-result-motivation}

With many teams working on many different parts of the system, chaos is inevitable (without divine intervention). The role of the System Architect was to identify and address issues between the teams, regarding interfaces, contracts and dependencies.

During the project, the need arose for an updated version of the FSM, therefore the software developement of the FSM package was also undertaken.

### Existing solution {#heroes-final-literature}

Duckietown already had an existing system architecture. As mentioned before, the FSM is closely related to the system architecture, since it defines the high-level behaviour of the system. There was an existing infrastructure for the FSM, which could be modified to include the newly developed functionality. The infrastructure consisted of the `fsm` ROS package, the configuration of the FSM in the form of `.yaml` files, as well as a tool to visualise the FSM structure.

#### ROS `fsm` package
The `fsm` package consists of two nodes, namely the `fsm_node` and the `logic_gate_node`. The `fsm_node` is in charge of determining the current state and computing state transitions, and the `logic_gate_node` acts as a helper node to the `fsm_node`. For more information, see the README of the `fsm` package, found at `20-indefinite-navigation/fsm/README.md`.

#### Configuration of the FSM
While the `fsm` package handles the computation of state transitions, the FSM states and transitions can be configured using the supplied `.yaml` files. The `fsm` package then reads the configuration in order to know which states and transitions are available in the system. This allows for separation of the computation and configuration of the FSM.

#### FSM visualisation tool
There exists a tool to parse and visualise the FSM configuration (in the form of an FSM diagram), found at `00-infrastructure/ros_diagram/parse_fsm.py`. The tool parses the `.yaml` configuration file and outputs a `.dot` format graph, which can be converted to `.png`.

### Opportunity {#heroes-final-opportunity}

During Fall 2018, many new projects were being developed by multiple teams. This created the need for someone to ensure harmony between the projects and the system during the process.

Duckietown was being expanded with new functionalities such as parking and deep learning lane following. The previous FSM was not equipped to deal with the new features, therefore it had to be further developed.

## Definition of the problem {#heroes-final-problem-def}

Ensuring that development of the new projects integrate into the existing system with as little as possible chaos. This implies ensuring that all teams understand their package's objective and their impact on the bigger system. The scope of the project was not precissely defined, as the role required creativity in identifying and solving problems that arose during the project.

## Contribution / Added functionality {#heroes-final-contribution}

The contribution of this project was system integration od project modules, in the form of both organisational activities, as well as software development. The approach can be summarised as follows:

* Understand the greater goals of Duckietown.
* Be familiar with the current system architecture.
* Track changes and identify how they influence the system.
* Keep close communication with project teams.
* Act as middlemand/helper to facilitate negotiation of contracts in and between groups.
* Monitor status of projects to find possible problems.

### Mediation of project interface negotiations

It was important to define how the new projects will interact with the old system, as well as with the other projects. Familiarisation with the existing system architecture was the first step. During the beginning of the project development, the System Architect attended some of the individual groups' meetings to get a better overview of what they plan, as well as how they will affect and be affected by other teams. This process helped to spot problems early on, so that teams can be certain of what is expected of them from other teams and vice versa.

A dedicated System Architecture meeting was held in class to resolve any further conflicts and to conclude discussion on the interfaces between projects.

### Development of updated FSM
The `fsm` package can be found in the Software repository at `20-indefinite-navigation/fsm/`. More information on the `fsm` package can be found in the package README.

### Documentation of FSM package

The previous FSM did not have a README, therefore the documentation was improved greatly. The new documentation contains a detailed description of the `fsm` package, as well as the `fsm_node` and `logic_gate_node`. 

## Formal performance evaluation / Results {#heroes-final-formal}

In the end, all projects were aware of what was expected of them in the scope of the system as a whole, as well as their interfaces to other projects. Many problems were spotted during the initial project team meetings, which could then be resolved before major development work was done.

The new FSM incorporates states for parking and deep learning lane following, and removed the recovery states from the FSM which were never completely developed in the old FSM.

### Testing
Initial testing of the FSM was done by running it alone and manually publishing topics to test correct state transitions. After this was done, the new FSM was integrated and tested by some project teams to incorporate into their demo for demo day.

## Future avenues of development {#heroes-final-next-steps}

The existing framework for the FSM made it relatively easy to update it to include new functionalities (once you've decided on the system architecture). The FSM is configured using `.yaml` files, which are then loaded into the `fsm_node`.

Development of the updated FSM was done in response to a need, and while it has been tested on its own, it has not been tested thoroughly with all other parts of the system yet. Not all demo's on ETH demo day made use of the new FSM, since some projects were not affected by the new functionality added to the system.