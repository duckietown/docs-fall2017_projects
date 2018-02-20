#  Parking: final report

## Part 1: The final result

Please see a video of the results [here](https://youtu.be/dS-TWh8cGXk)

Note that the video only includes the simulation results and not the duckiebot parking autonomously as the control feature of our parking pipeline requires further development. Our parking pipeline works well up until feedback control is required. Please see the demo operation manual for further details: 

`DUCKUMENTS_ROOT/docs/atoms_20_setup_and_demo/30_demos/17_parking.md`

## Part 2: Mission and Scope
* Motivation
    * **Introduction**: A desirable feature of duckietown is the ability to park duckiebots in a safe static state.
    * **Relevance**: The parking feature replicates the familiar scenario in real world driving when the driver no longer needs the transportation service offered by the vehicle. Parking allows the vehicle to be stored in a safe static state, without obstructing active duckietown traffic, until the vehicle is summoned for further transportation services within duckietown. Additionally, a parking feature allows for a variety of other benefits within duckietown such as decreased traffic congestion on the roads as well as the potential for the recharging of the duckiebot batteries while in a parked state. 
* Existing Solution
    * The parking feature was implemented from scratch. 
* Opportunity
    * There was no previous implementation of parking within duckietown. In order to approach the problem, we first designed a physical space to park the duckiebots. A specification for the space was determined and approved. In order to actually park the duckiebots, we split the problem into three main pieces of a parking pipeline:
        * **Duckiebot localization**: Localization was implemented based on the existing AprilTag C++ library found [here](https://april.eecs.umich.edu/software/apriltag.html).
        * **Path planning**: A path was planned using dubins paths and, during the instance of path obstacles, RRT Star with dubins paths. A general description of dubins paths can be found [here](https://gieseanw.wordpress.com/2012/10/21/a-comprehensive-step-by-step-tutorial-to-computing-dubins-paths/). An existing library for RRT Star with dubins paths was implented with help from the library [here](https://github.com/AtsushiSakai/PythonRobotics).
        * **Feedback control to the planned path**: As mentioned, this is the piece of the pipeline that currently requires development. We found that the localization via AprilTags takes several seconds to compute on the Raspberry Pi. The time lag proved to be insufficient in supplying the lane controller with sufficiently frequent state updates to control to. For further details regarding this issue, please see the "Future avenues of development" portion of this report. 
        
## Part 3: Definition of the problem