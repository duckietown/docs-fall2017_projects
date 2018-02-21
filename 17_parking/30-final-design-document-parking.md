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
    * There was no previous implementation of parking within duckietown. In order to approach the problem, we first designed a physical parking lot to park the duckiebots. A specification for the parking lot was therefore determined. In order to actually park the duckiebots, we split the problem into three main pieces of a parking pipeline:
        * **Duckiebot localization**: Localization was implemented based on the existing AprilTag C++ library found [here](https://april.eecs.umich.edu/software/apriltag.html).
        * **Path planning**: A path was planned using dubins paths and, during the instance of path obstacles, RRT Star with dubins paths. A general description of dubins paths can be found [here](https://gieseanw.wordpress.com/2012/10/21/a-comprehensive-step-by-step-tutorial-to-computing-dubins-paths/). An existing library for RRT Star with dubins paths was implemented with help from the library [here](https://github.com/AtsushiSakai/PythonRobotics).
        * **Feedback control to the planned path**: As mentioned in part 1, this is the piece of the pipeline that currently requires development. We found that the localization via AprilTags takes several seconds to compute on the Raspberry Pi. The time lag proved to be insufficient in supplying the lane controller with sufficiently frequent state updates to control to. For further details regarding this issue, please see part 6 of this report (Future avenues of development).
        
## Part 3: Definition of the problem
<<<<<<< HEAD

### Problem statement

We need to park N Duckiebots in a designated area in which they are able enter and exit in an efficient manner. 

### Assumptions

* The parking lot is a four tile structure with defined inlet, outlet and color scheme.
* The specification of the parking lot is known.
* When entering the lot and searching for a parking space, the parking lot is in a static state (there are no actively parking duckiebots).
* Assume that when leaving the parking space, the parking lot is in a static state.

### Performance measurement

#### Localization

#### Path Planning 

#### Control

* Starting at parking lot entrance, measure the number of parking maneuvers completed within boundaries of designated parking spot (over N attempts)
* Starting in designated parking space, measure the number of Duckiebots able to arrive at the exit of the parking lot (over N attempts)
* Average time (for N vehicles) to enter and exit parking lot


=======
* goal/final objective: park duckiebot in parking space
* state estimation: 	localize duckiebot position (x, y, theta) - quantitativ performance metric - accuracy + precision of state estimate [mm]
* path planning: 		plan collision free path from the current state estimate into a parking space (1-6) given a static map - quantitativ performance metric - is path collision free [# of collision free path / # of path]  
* closed loop control: 	let the duckiebot follow along the path into a set parking space
quantitativ performance metric - is robot able to reach final parking space

## Part 4: Contribution / Added functionality
* Dubins path planning
* Debuggt openCV 3.4 error - (float to double)
* Debuggt image rectification node (remove distortion due to wrong scaling)
* Generate necessary control output (d_est, d_ref, theta_est, v_ref, c_ref)

## Part 5: Formal performance evaluation / Results
* state estimation: quantitativ results - ??? - accuracy + precision (success), speed of algorithm (failure)
* path planning: quantitiativ results - ???
* closed loop control: qunatitativ results - ??? - (failure) most likely due to speed of state estimation

no previous explaination available

explanation / discusstion of results

path planning - as expected
state estimation - accuracy + precision as expected, speed worse then expected

biggest challenge 
speed of state estimation
finding problems in old pipeline

## Part 6: Future avenues of development

* Speed of the state estimation
>>>>>>> 612f2cbb3c22c3d5c8260d8bc300b21572f66e9d

