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

### Problem statement

We need to park N Duckiebots in a designated area in which they are able enter and exit in an efficient manner. 

### Assumptions

* The parking lot is a four tile structure with defined inlet, outlet and color scheme.
* The specification of the parking lot is known.
* When entering the lot and searching for a parking space, the parking lot is in a static state (there are no actively parking duckiebots).
* Assume that when leaving the parking space, the parking lot is in a static state.

### Performance measurement

#### Localization

* Localization involves computing a state estimate of the duckiebot's position (x, y, theta)
    * **Quantitative performance metric** 
        * accuracy of state estimate in x[mm], y[mm] and theta [degree]

#### Path Planning 

* Path planning consists of planning a collision free path from the current state estimate into or out of a parking space given a static map (no actively parking duckiebots)
	* **Quantitative performance metric** 
		* Percentage of collision free paths (# of collision free path / # of total paths)[%]

#### Control

* Once a state estimate is computed and a path is planned, the duckiebot must be controlled to the computed collision free path with a sufficiently high frequency of state updates.
	* **Quantitative performance metric** 
		* Starting at parking lot entrance, measure the number of parking maneuvers completed within boundaries of designated parking spot (over N attempts)[%]
		* Starting in designated parking space, measure the number of Duckiebots able to arrive at the exit of the parking lot (over N attempts)[%]
		* Average time (for N vehicles) to enter and exit parking lot[seconds]


## Part 4: Contribution / Added functionality

Initially, the theoretical descriptions and implementations of the three main parts of our parking pipeline (localization, path planning and control) are described. As you will notice in the descriptions below, there is no technical description or implementation description for control, as we intended to use the existing lane controller for control. In order to interface with this controller, we developed a state propagation strategy to send high frequency state updates to the lane controller. Therefore, we have included our implementation strategy for control and refer to it as "state propagation". 

Following these descriptions, the logical architecture and software architecture of the pipeline as a whole is described.

### Theoretical Descriptions 

#### Localization

The localization is based on the relative transformation of the duckiebot to the apriltags within the parking lot and their known position in the world frame.

A rectified images is needed to detect the AprilTags within the image. The used wide angle camera on the duckiebot provides a distorted barrel image. In a barrel distorted image each pixel is position closer to the optical center as it would be in a rectified image. The distortion is non linear and can be modeled by a polynomial function depending on the pixel distance to the optical center:

`f(r) = 1 + k_1*r + k_2*r + ... + k_n*r`
`r² = (u-u_0)² + (v-v_0)²`

The intrinsic camera calibration estimates the distortion parameters k_1 to k_4.
The rectified image can be computed by positioning each pixel of the distorted image at its actual position using the estimated parameters and the distortion model.

The rectified image is first converted to a gray scale image and afterwards thresholded to a binary image. Next the AprilTags in the binary image are detected. 

The relative position of the camera to the each tag can be calculated, after one or multiple AprilTags are detected. The four pixels corresponding to the corners of each AprilTag in the image, as well as the position of the corners in the body frame of each AprilTag are known. Using this information and the intrinsic camera matrix the relative position of the camera and the AprilTag can be computed by using the PnP algorithm.

Once the relative position of the camera to each apriltag is computed, the absolut position of the duckiebot in the world frame can be calculated. First the position of the duckiebot in the world frame can be calculated for each single apriltag by combining transformation of the apriltag in the world frame, the relative transformation of the camera and the apriltag and the relative transformation of the duckiebot and the camera. Next a more reliable state estimate can be computed by taking the average all estimated duckiebot transformations.

#### Path Planning (TODO Sam)

* Dubins paths
* RRT Star with Dubins paths

#### State Propagation 

As there is no real theory involved in our strategy to interface successfully with the lane controller, please see the implementation section for an implementation strategy for state propagation.

### Implementation

#### Localization

We slightly modified the previously implemented localization pipeline. 

The localization pipeline takes a rectified image as an input. Therefore we need to undistort the barrel distorted images provided by the wide angle camera in a first step. To do so we use the distortion parameters determined in the intrinsic camera calibration. The previously implemented image rectification node undistorts the image in a way that is usable for the lane following pipeline, but unacceptable for a reliable state estimation. It is necessary for the state estimation based on apriltags to workm that the undistored image corresponds to the intrinsic parametes of the camera.

The previously used pipeline uses the image rectification node from the ROS library that is based on openCV. The node works in the following way: 

* The node takes the distored image (e.g. 480x360 pixels) as an input and undistorts the image using the default "initUndistortRectifyMap" openCV function. 

Undistorting a barrel distorted image using all pixel information will result in an image with black areas along the edges as shown in figure 15.3 in the figure [here](http://book.duckietown.org/fall2017/duckiebook/camera_calib_jan18.html#sec:camera-calib-jan18). The default openCV function will cut the biggest rectangular section out of the image that contains information for every pixel within the rectangle (red area) and map the image into a new image with the same size as the original image (i.e. 480x360). 

This causes two problems. First of all, the ratio of the cutout section is not the same as the one of the original image. Forcing the selected section back in the original ratio will distort the image by stretching the image more in one direction then in the other causing a rectangle to become oblong. 

Secondly, neglecting the distortion the overall scale of the image does not correspond the focal length anymore.

Both problems cause a miss-match between the image and the intrinsic parameters that could easily be compensated by using scaling the focal length and using a different focal length in x and y direction resulting in a new intrinsic matrix. Instead we came up with the following solution:

* We compute the mapping of every distorted pixel coordinate to the undistorted pixel coordinate for a given intrinsic camera matrix and image size using the openCV "initUndistortRectifyMap" function with non default parameters. We use the intrinsic camera matrix determined in the camera calibration and size of the original image. This way we overcome both problems that we had with the ROS image rectification node, while not changing the intrinsic camera matrix. 

* For the apriltag detection we use the AprilTags for ROS library from the Robotics and Intelligent Ground Vehicle Research Laboratory. After an update of openCV 3 the algorithm did not work anymore due to a variable type error that we fixed. 

* The apriltag detection node outputs all detected apriltag IDs and the corresponding transformation of the camera to each apriltag. We send these information to the previously implemented localization node. The localization node computes the pose of the duckiebot in world frame for each apriltag and averages the transformations to a more reliable estimate. We extended the node by another custom message that includes the x and y position, as well as the orientation of the duckiebot, because only this information are importent for the path planning and control of the duckiebot.

#### Path Planning (TODO Sam)

* Add implementation details for dubins path and RRT star with dubins paths.
* Also describe how you generated necessary control outputs (d_est, d_ref, theta_est, v_ref, c_ref)

#### State Propagation 

In order to control to the planned path, the lane controller is utilized. It was soon found, however, that with the given hardware a (x,y,theta) duckiebot state update took several seconds to compute. With the existing lane controller, this time lag proved insufficient: by the time a new state update was calculated and published to the lane controller, the duckiebot had already deviated substantially from this published state. In order to address this issue, we developed a path planning node that "fills in" the state update time lag gaps with a feedforward state update. In addition to the feedforward feature, the algorithm allows the duckiebot to stop for a set period of time in order to plan a new path. The time for which the duckiebot is stopped is ensured to be sufficient in order to produce an accurate state estimate. As such, the algorithm behaves as follows: 

1) Process AprilTags in view and estimate a state while static (duckiebot velocity is zero)
2) Plan a path based on this state
3) A "time to plan" threshold has passed
4) Use feedforward state estimates to broadcast inputs to the lane controller
5) Stop the duckiebot after a "time to control with feedforward" has been passed
6) Return to 1)

In order to perform the above steps, an alternate node, named devel_path_planning_node.py was constructed. This node can be found in the src folder of the parking package. As mentioned before, this is an experimental node and needs further development to function with the the other parking nodes. The devel_path_planning_node introduces new functions, namely a stopping_callback function and a get_intermediate_pose function. Please see below for a description of each.

* The `get_intermediate_pose` function takes the time since a pose was last calculated as an input. The function then uses that time, along with the duckiebot's velocity to estimate where along the path the duckiebot is. The estimate is then broadcasted to the controller. 

* The `stopping_callback` function allows the duckiebot to stop for a sufficient amount of time for the the camera to estimate a new state via any AprilTags in view and plan a new path. While in the function the duckiebot velocity is set to zero. 

The following logical architecture describes the parking pipeline as whole. 

### Logical architecture

The logical architecture is a description of the functionality: what happens when we click start?

* As soon as you arrive to the parking lot entrance and see a parking AprilTag, the duckiebot switches from normal driving mode to parking mode. Parking mode is only allowed in the parking lot. **Note**: the duckiebot must currently be manually placed at the entrance of the parking lot for the parking feature to engage.

* If the duckiebot exits the parking lot, it views a "parking exit" AprilTag and it switches from parking mode back to normal driving mode (starting at a four way intersection). **Note**: this feature is not currently implemented in the parking pipeline as it currently stands.

* Once at the parking lot entrance, the duckiebot estimates her pose (x, y and theta) using as many AprilTags as possible within view. The localization of course requires the camera nodes, AprilTag detector node, Apriltag Postprocessing node and localization node to be launched. There is at least one (potentially more) AprilTags per parking space and possibly some additional tags placed at the entrance and exit. To estimate a pose, the state estimation algorithm has been extended (see localization description above) using the library 'AprilTags C++'. It estimates the relative position of the robot with respect to the april tag. The location of the april tag is encoded in the QR code. As soon as you see one (better two) tags, the pose can be calculated. We assume that we always see at least one tag.

* Given a prior information about the parking lot (where the parking spaces are located, where the robot can drive, etc) and real time vision information, the robot chooses a parking space. Currently, the parking space is chosen as a "hardcoded" value in the launch file, please see the demo operation manual for more information. At first we assume that the parking lot is empty or that other duckiebots are static (do not move) and this is encoded in the parking map (places where the robot is not allowed to drive).

* We use dubins paths to generate a path given the pose of the robot, the pose of the assigned parking space and the parking map. If there is an obstacle in place, we use RRT star with dubins paths to generate a path (this feature is coded, but not currently implemented within ROS). The above features are launched in the path planning node.

* We control the robot to the optimal path using a sufficient controller using visual feedback. The control is performed in the lane controller node.

* For driving to the exit, we generate a path and control our robot to this path which includes driving backwards to leave the parking space and turn to get to the exit in a forward motion. **Note**: this feature is currently not implemented within ROS. 

Target values:

* accuracy: the error is a combination of localization accuracy and the offset due to the maximum allowable controller error. To park two duckiebots next to each other within the space boundaries, the path planning accuracy has to be less (or equal) than 5 cm (which is the distance from the robot edge to the parking lane)

* the point of the robot which is the furthest away from the parking mid line should be less than half of the parking space width while the heading of the robot must be less than a constant (20 degrees) relative to the parking space boundary lines.

Assumptions about other modules:
* we assume that the robot finds itself at the entrance of the parking lot whenever it wants to get a parking space.

* once in the parking lot: parking is decoupled from the rest of duckietown. 

### Software architecture (TODO update with correct values)

**rosnode list**:

* localization_node
    - subscribes: driving\_mode, camera\_image,
    - publishes: parking\_mode, space\_status, pose\_duckiebot, ,

* path_planning_node
    - subscribes: parking\_mode, pose\_duckiebot,  space\_status
    - publishes: reference\_for\_control, (path)

* /vehicle/parking\_control
   	* we copy this node from 'the controllers'
   	* subscribes: reference\_for\_control
   	* publishes: motor\_voltage

**rostopic list**:

* /vehicle/driving\_mode
   	* values = {driving, parking}
   	* frequency: ~ 1 Hz

* /vehicle/parking\_mode
    * values = {parking, staying, leaving, observing}
    * frequency: ~ 1 Hz

* /vehicle/space\_status
    * 1xN array, N = number of parking space
    * values = {taken, free, not\_detectable, my\_parking\_space}
    * frequency: ~ 1 Hz

* /vehicle/pose\_duckiebot
    * x,y,theta
    * frequency: inherit from camera\_image (~30 Hz)

* /vehicle/path
    * x,y,theta array
    * frequency: very low - only updated once (if my\_parking\_space = {1:3}) or twice (for my\_parking\_space = {4:6}, first path is to go to the middle and observe which parking spaces are free, second path is to go go the associated parking space)
    - computation time ~ 10 s

* /vehicle/reference\_for\_control
    * d (orthogonal distance to path), c (curvature), phi (differential heading path and Duckiebot)
    * frequency: first step (path generation) uses a lot of time ~ 10 s, afterwards fast (~ 30 Hz) 

* /vehicle/motor\_voltage
    * two values for the two motors
    * frequency: fast (~ 30 Hz)

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

As mentioned before, the main area of work needed to get the parking pipeline working is to successfully implement some sort of control in order to autonomously park a duckiebot. Currently, there are three main options for doing this, described in the following sections. Each avenue of development may be explored individually or a combination of multiple could prove to be the best way forward.

1) Increase the speed of the state estimation

* As seen in the part 5 of this report, there is a considerable time lag in the state estimation via AprilTags. An avenue of investigation should be to look deeper into the `extractTags` method from the AprilTag C++ library. The node where this method is called is found here, on line 67:

`DUCKIETOWN_ROOT/catkin_ws/src/20-indefinite-navigation/apriltags_ros/apriltags_ros/src/apriltag_detector.cpp`

* Any development which can increase the speed of this 'extractTags' method, which takes a grayscale image and detects tag number(s) in view, would be very beneficial for an increased state estimate frequency. 

* Another avenue of development may be to increase the computing power of the duckiebot. The parking pipeline was currently run on a Raspberry Pi. A more powerful computer may improve the time lag issue.

2) Successful integration of state propagation 

* More development could be made on the `devel_path_planning_node` node that propagates the state estimate at a high frequency for use with the lane controller. Please see the "State Propagation" section of part 4 for how this algorithm is intended to work. 
* As of the writing of this report, the parking group was unable to successfully integrate the state propagation. More work is needed to allow the algorithm to work as designed. 

3) Development of a dedicated parking controller

* 1) and 2) above rely on the use of the lane controller while parking. It may be beneficial, however, to develop a dedicated parking controller which can better handle the parking feature pipeline.





