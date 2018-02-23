#  Parking: final report

## Part 1: The final result

Please see a video of the results [here](https://youtu.be/dS-TWh8cGXk)

Note that the video only includes the simulation results and not the Duckiebot parking autonomously as the control feature of our parking pipeline requires further development. Our parking pipeline works well up until feedback control is required. Please see the demo operation manual for further details: 

`DUCKUMENTS_ROOT/docs/atoms_20_setup_and_demo/30_demos/17_parking.md`


## Part 2: Mission and Scope
* Motivation
    * **Introduction**: A desirable feature of Duckietown is the ability to park Duckiebots in a safe static state.
    * **Relevance**: The parking feature replicates the familiar scenario in real world driving when the driver no longer needs the transportation service offered by the vehicle. Parking allows the vehicle to be stored in a safe static state, without obstructing active Duckietown traffic, until the vehicle is summoned for further transportation services within Duckietown. Additionally, a parking feature allows for a variety of other benefits within Duckietown such as decreased traffic congestion on the roads as well as the potential for the recharging of the Duckiebot batteries while in a parked state. 
* Existing Solution
    * The parking feature was implemented from scratch. 
* Opportunity
    * There was no previous implementation of parking within Duckietown. In order to approach the problem, we first designed a physical parking lot to park the Duckiebots. A specification for the parking lot was therefore determined. In order to actually park the Duckiebots, we split the problem into three main pieces of a parking pipeline:
        * **Duckiebot localization**: Localization was implemented based on the existing AprilTag C++ library found [here](https://april.eecs.umich.edu/software/apriltag.html).
        * **Path planning**: A path was planned using Dubins paths and, during the instance of path obstacles, RRT Star with Dubins paths. A general description of Dubins paths can be found [here](https://gieseanw.wordpress.com/2012/10/21/a-comprehensive-step-by-step-tutorial-to-computing-Dubins-paths/). An existing library for RRT Star with Dubins paths was implemented with help from the library [here](https://github.com/AtsushiSakai/PythonRobotics).
        * **Feedback control to the planned path**: As mentioned in part 1, this is the piece of the pipeline that currently requires development. We found that the localization via AprilTags takes several seconds to compute on the Raspberry Pi. The time lag proved to be insufficient in supplying the lane controller with sufficiently frequent state updates to control to. For further details regarding this issue, please see part 6 of this report (Future avenues of development).
      
        
## Part 3: Definition of the problem

### Problem statement

We need to park N Duckiebots in a designated area in which they are able enter and exit in an efficient manner. 

### Assumptions

* The parking lot is a four tile structure with defined inlet, outlet and colour scheme.
* The specification of the parking lot is known.
* When entering the lot and searching for a parking space, the parking lot is in a static state (there are no actively parking Duckiebots).
* Assume that when leaving the parking space, the parking lot is in a static state.
* The robot is limited in curvature, it exists a minimum curvature radius
* The robot can drive any desired curvature within the minimum curvature radius in forward driving mode
* The only possibility for backwards driving is straight, this a result of the used controller
* The robot must move in a car like behaviour, e.g. no side slip and no turning without forward movement is allowed


### Performance measurement

#### Localization

* Localization involves computing a state estimate of the Duckiebot's position (x, y, theta)
    * **Quantitative performance metric** 
        * accuracy of state estimate in x[mm], y[mm] and theta [degree]

#### Path Planning 

* Path planning consists of planning a collision free path from the current state estimate into or out of a parking space given a static map (no actively parking Duckiebots)
	* **Quantitative performance metric** 
		* Percentage of collision free paths (# of collision free path / # of total paths)[%]

#### Control

* Once a state estimate is computed and a path is planned, the Duckiebot must be controlled to the computed collision free path with a sufficiently high frequency of state updates.
	* **Quantitative performance metric** 
		* Starting at parking lot entrance, measure the number of parking manoeuvres completed within boundaries of designated parking spot (over N attempts)[%]
		* Starting in designated parking space, measure the number of Duckiebots able to arrive at the exit of the parking lot (over N attempts)[%]
		* Average time (for N vehicles) to enter and exit parking lot[seconds]


## Part 4: Contribution / Added functionality

Initially, the theoretical descriptions and implementations of the three main parts of our parking pipeline (localization, path planning and control) are described. As you will notice in the descriptions below, there is no technical description or implementation description for control, as we intended to use the existing lane controller for control. In order to interface with this controller, we developed a state propagation strategy to send high frequency state updates to the lane controller. Therefore, we have included our implementation strategy for control and refer to it as "state propagation". 

Following these descriptions, the logical architecture and software architecture of the pipeline as a whole is described.

### Theoretical Descriptions 

#### Localization

The localization is based on the relative transformation of the Duckiebot to the apriltags within the parking lot and their known position in the world frame.

A rectified images is needed to detect the AprilTags within the image. The used wide angle camera on the Duckiebot provides a distorted barrel image. In a barrel distorted image each pixel is position closer to the optical center as it would be in a rectified image. The distortion is non linear and can be modelled by a polynomial function depending on the pixel distance to the optical center:

r<sup>2</sup>, ... , r<sup>n</sup>
f(r) = 1 + k<sub>1</sub>r + k<sub>2</sub>r + ... + k<sub>n</sub>r
r<sup>2</sup>= (u - u<sub>0</sub>)<sup>2</sup> + (v - v<sub>0</sub>)<sup>2</sup>

The intrinsic camera calibration estimates the distortion parameters k_1 to k_4.
The rectified image can be computed by positioning each pixel of the distorted image at its actual position using the estimated parameters and the distortion model.

The rectified image is first converted to a gray scale image and afterwards thresholded to a binary image. Next the AprilTags in the binary image are detected. 

The relative position of the camera to the each tag can be calculated, after one or multiple AprilTags are detected. The four pixels corresponding to the corners of each AprilTag in the image, as well as the position of the corners in the body frame of each AprilTag are known. Using this information and the intrinsic camera matrix the relative position of the camera and the AprilTag can be computed by using the PnP algorithm.

Once the relative position of the camera to each AprilTag is computed, the absolute position of the Duckiebot in the world frame can be calculated. First the position of the Duckiebot in the world frame can be calculated for each single AprilTag by combining transformation of the AprilTag in the world frame, the relative transformation of the camera and the AprilTag and the relative transformation of the Duckiebot and the camera. Next a more reliable state estimate can be computed by taking the average all estimated Duckiebot transformations.

#### Path Planning (TODO Brett/Nils review, grammar)

We have to find a path in a predefined parking lot with given objects like other Duckiebots, walls or duckies. The area around those non-driveable objects define the obstacles. To be exact, every object is blown up by the distance of the center point of the robot to the most-distant point. This results in a problem of finding a path from start pose (x, y, theta) to end pose within a map where the information about where the robot is allowed to drive is encoded. 

We implemented a two stage algorithm. The first stage is using Dubins curves where the second stage uses rapidly exploring random trees. 

##### Stage 1: Dubins path
Given the assumption mentioned above (forward driving for a car like robot with given minimum curvature radius) the optimal path in an unlimited and obstacle free space on a Dubins path. This path is a combination of driving on a circle with minimum curvature radius and straight lines. This means that the Dubins path from start pose to end pose is calculated in the first stage. A collision checker is applied to the found path afterwards. If the path is completely within the parking space and does not enter the non-driveable region then we are done and we found the optimal path. If not, we switch to stage 2. 

##### Stage 2: RRT*
An alternative way to find a path is the piecewise addition of small path segments with collision check on the fly. A point is randomly sampled within the parking space. The nearest point to the sampled needs to be found. The sampled point is connected to the nearest point using a Dubins curve if and only if the path candidate is collision free. A graph optimization is performed in a local area around the sampled point in the end. This procedure is repeated until a pre-defined number of nodes are sampled. This method is called rapidly exploring random trees with path optimization (RRT*). This method converges to the optimal path with unlimited number of nodes. In practice, we stop earlier and have an approximation to the optimal path. 


#### State Propagation 

As there is no real theory involved in our strategy to interface successfully with the lane controller, please see the implementation section for an implementation strategy for state propagation.

### Implementation

#### Localization

We slightly modified the previously implemented localization pipeline. 

The localization pipeline takes a rectified image as an input. Therefore we need to undistort the barrel distorted images provided by the wide angle camera in a first step. To do so we use the distortion parameters determined in the intrinsic camera calibration. The previously implemented image rectification node undistorts the image in a way that is usable for the lane following pipeline, but unacceptable for a reliable state estimation. It is necessary for the state estimation based on apriltags to work that the undistorted image corresponds to the intrinsic parameters of the camera.

The previously used pipeline uses the image rectification node from the ROS library that is based on openCV. The node works in the following way: 

* The node takes the distorted image (e.g. 480x360 pixels) as an input and undistorts the image using the default "initUndistortRectifyMap" openCV function. 

Undistorting a barrel distorted image using all pixel information will result in an image with black areas along the edges as shown in figure 15.3 in the figure [here](http://book.Duckietown.org/fall2017/duckiebook/camera_calib_jan18.html#sec:camera-calib-jan18). The default openCV function will cut the biggest rectangular section out of the image that contains information for every pixel within the rectangle (red area) and map the image into a new image with the same size as the original image (i.e. 480x360). 

This causes two problems. First of all, the ratio of the cutout section is not the same as the one of the original image. Forcing the selected section back in the original ratio will distort the image by stretching the image more in one direction then in the other causing a rectangle to become oblong. 

Secondly, neglecting the distortion the overall scale of the image does not correspond the focal length anymore.

Both problems cause a miss-match between the image and the intrinsic parameters that could easily be compensated by using scaling the focal length and using a different focal length in x and y direction resulting in a new intrinsic matrix. Instead we came up with the following solution:

* We compute the mapping of every distorted pixel coordinate to the undistorted pixel coordinate for a given intrinsic camera matrix and image size using the openCV "initUndistortRectifyMap" function with non-default parameters. We use the intrinsic camera matrix determined in the camera calibration and size of the original image. This way we overcome both problems that we had with the ROS image rectification node, while not changing the intrinsic camera matrix. 

* For the apriltag detection we use the AprilTags for ROS library from the Robotics and Intelligent Ground Vehicle Research Laboratory. After an update of openCV 3 the algorithm did not work anymore due to a variable type error that we fixed. 

* The apriltag detection node outputs all detected apriltag IDs and the corresponding transformation of the camera to each apriltag. We send these information to the previously implemented apriltag post processing node. The post processing node computes the transformation from the Duckiebot to each apriltag by including the static transformation between the duckebot and the camera. Afterwards, the localization node computes the pose of the Duckiebot in world frame for each apriltag and averages the transformations to a more reliable estimate. We are able to calculate the pose of the Duckiebot in world frame, because the position of each apriltag in world frame is known to the robot. We extended the node by another custom message that includes the x and y position, as well as the orientation of the Duckiebot, because only this information are important for the path planning and control of the Duckiebot.

#### Path Planning (TODO review Brett/Nils)
The path planning algorithm is written in Python. The "PythonRobotics" library from "AtsushiSakai" GitHub account is used as a basic for Dubins Paths and RRT* implementation.

##### Initialization
The parking lot is parameterized as a rectangle with given length `(lot_width, lot_height)`. The path must be completely inside this rectangle.
 
Objects can be defined as rectangles with given properties `(x, y, dx, dy, colour, driveable)`. 

Obstacles are computed automatically based on the non-driveable objects. The rectangles are blown up. The non-driveable region is showed in magenta. 

The parking spaces are numbered from `1` to `6`. The entrance has index `0`, the exit index `7`. The pose of the parking space or entrance / exit can be computed with the function `pose_from_key(key)`. Input value is an integer with the index of the space. The output is a `(x, y, theta)` pose tuple. 

To start the simulation the python script (parking_main.py) can be launched with two arguments. To get a path from the entrance to parking space 2 we type the following command.
`./parking_main 0 2`

The path can either be printed on a (interactive) figure and/or it can be saved in the folder images. 

##### Stage 1: Dubins path
A Dubins path is calculated in stage one. The only argument is the minimum curvature radius. If the path is valid e.g. collision free, it is printed in green, otherwise in magenta. 

##### Stage 2: RRT*
Since RRT* has random character we need to define a stopping criteria. This is done in limiting the number of nodes which is equal in the number of iterations. The design parameter `maxIter` changes this.
For the local graph optimization we need to define the area, this can be changed in changing the parameter `radius_graph_refinement` which holds a distance in mm. 

##### Path variation
The robot can only be controlled on a path when forward driving. When backing up we run the robot in an open loop fashion and only straight backwards driving is allowed. The distance travelled back can be adjusted with the variable `distance_backwards`. 


##### Generate necessary control output

To provide all necessary control signals, the pose of the robot and the path are combined to define an estimated distance from the path (`d_est`) and a reference distance (`d_ref`). The differential heading between the robot pose and the of the point on the path with lowest distance is calculated (`theta_est`). Furthermore, the reference velocity (`v_ref`) and curvature (`c_ref`) are given to the controller.

This can be tested in the file `project_point_to_path.py` with two input parameters (`start_index` and `end_index`). 

#### State Propagation 

In order to control to the planned path, the lane controller is utilized. We developed a path planning node that "fills in" the state update time lag gaps with a feedforward state update. In addition to the feedforward feature, the algorithm allows the Duckiebot to stop for a set period of time in order to plan a new path. The time for which the Duckiebot is stopped is ensured to be sufficient in order to produce an accurate state estimate. As such, the algorithm behaves as follows: 

1) Process AprilTags in view and estimate a state while static (Duckiebot velocity is zero)
2) Plan a path based on this state
3) A "time to plan" threshold has passed
4) Use feedforward state estimates to broadcast inputs to the lane controller
5) Stop the Duckiebot after a "time to control with feedforward" has been passed
6) Return to 1)

In order to perform the above steps, an alternate node, named devel\_path\_planning\_node.py was constructed. This node can be found in the src folder of the parking package. As mentioned before, this is an experimental node and needs further development to function with the the other parking nodes. The devel\_path\_planning\_node introduces new functions, namely a stopping_callback function and a get_intermediate_pose function. Please see below for a description of each.

* The `get_intermediate_pose` function takes the time since a pose was last calculated as an input. The function then uses that time, along with the Duckiebot's velocity to estimate where along the path the Duckiebot is. The estimate is then broadcasted to the controller. 

* The `stopping_callback` function allows the Duckiebot to stop for a sufficient amount of time for the the camera to estimate a new state via any AprilTags in view and plan a new path. While in the function the Duckiebot velocity is set to zero. 

The following logical architecture describes the parking pipeline as whole. 

### Logical architecture

The logical architecture is a description of the functionality: what happens when we click start?

* As soon as you arrive to the parking lot entrance and see a parking AprilTag, the Duckiebot switches from normal driving mode to parking mode. Parking mode is only allowed in the parking lot. **Note**: the Duckiebot must currently be manually placed at the entrance of the parking lot for the parking feature to engage.

* If the Duckiebot exits the parking lot, it views a "parking exit" AprilTag and it switches from parking mode back to normal driving mode (starting at a four way intersection). **Note**: this feature is not currently implemented in the parking pipeline as it currently stands.

* Once at the parking lot entrance, the Duckiebot estimates her pose (x, y and theta) using as many AprilTags as possible within view. The localization of course requires the camera nodes, AprilTag detector node, Apriltag Postprocessing node and localization node to be launched. There is at least one (potentially more) AprilTags per parking space and possibly some additional tags placed at the entrance and exit. To estimate a pose, the state estimation algorithm has been extended (see localization description above) using the library 'AprilTags C++'. It estimates the relative position of the robot with respect to the april tag. The location of the april tag is encoded in the QR code. As soon as you see one (better two) tags, the pose can be calculated. We assume that we always see at least one tag.

* Given a prior information about the parking lot (where the parking spaces are located, where the robot can drive, etc) and real time vision information, the robot chooses a parking space. Currently, the parking space is chosen as a "hardcoded" value in the launch file, please see the demo operation manual for more information. We assume that the parking lot is empty or that other Duckiebots are static (do not move) and this is encoded in the parking map (places where the robot is not allowed to drive).

* We use Dubins paths to generate a path given the pose of the robot, the pose of the assigned parking space and the parking map. If there is an obstacle in place, we use RRT star with Dubins paths to generate a path (this feature is coded, but not currently implemented within ROS). The above features are launched in the path planning node.

* We control the robot to the optimal path using a sufficient controller using visual feedback. The control is performed in the lane controller node.

* For driving to the exit, we generate a path and control our robot to this path which includes driving backwards to leave the parking space and turn to get to the exit in a forward motion. **Note**: this feature is currently not implemented within ROS. 

Target values:

* accuracy: the error is a combination of localization accuracy and the offset due to the maximum allowable controller error. To park two Duckiebots next to each other within the space boundaries, the path planning accuracy has to be less (or equal) than 5 cm (which is the distance from the robot edge to the parking lane)

* the point of the robot which is the furthest away from the parking mid line should be less than half of the parking space width while the heading of the robot must be less than a constant (20 degrees) relative to the parking space boundary lines.

### Software architecture (TODO update with correct values)

**rosnode list** (note that topic names are often remapped in launch files. Please refer to specific launch files for details): 

* image\_proc\_proportional\_node.py
	* subscribes: /camera_node/image/raw, /camera_node/raw_camera_info
	* publishes: image\_rect

* apriltag\_detector.cpp
	* subscribes: image\_rect
	* publishes: tag\_detections\_image, tag\_detections, tag\_detections\_pose
	
* apriltags\_postprocessing\_node.py
	* subscribes: apriltags\_in
	* publishes: apriltags\_out , tag\_pose, apriltags\_parking, apriltags\_intersection

* localization\_node
	* subscribes: apriltags
	* publishes: /tf, pose\_Duckiebot

* path\_planning\_node
	* subscribes: pose\_Duckiebot
	* publishes: parking\_pose, parking\_active

* lane\_controller\_node
   	* **note**: we copy this node from 'the controllers'
   	* subscribes: parking\_pose
   	* publishes: car\_cmd, actuator\_limits\_received, radius\_limit

**rostopic list** (TODO: Add a note about the latency):

* image\_rect
	* from sensor_msgs.msg, type: Image
	
* tag\_detections
	* from Duckietown_msgs.msg, type: AprilTagDetectionArray
	* **note**: this is the topic that is published at a frequency of ~ 1 signal/ 2-3 seconds. As such, this topic is the bottle neck of the algorithm. Please see Part 6 for potential remedies for this issue. 
	
* apriltags\_in
	* from Duckietown_msgs.msg, type: AprilTagDetectionArray

* apriltags\_out
	* from Duckietown_msgs.msg, type: AprilTagsWithInfos

* apriltags
	* from Duckietown_msgs.msg, type: AprilTagsWithInfos

* pose\_Duckiebot
	* from Duckietown_msgs.msg, type: Pose2DStamped

* parking\_pose
	* from Duckietown_msgs.msg, type: LanePose


## Part 5: Formal performance evaluation / Results (TODO: complete)
* state estimation: quantitativ results - ??? - accuracy + precision (success), speed of algorithm (failure)
* path planning: quantitiativ results - ???
* closed loop control: qunatitativ results - ??? - (failure) most likely due to speed of state estimation

no previous explaination available

explanation / discusstion of results

path planning - as expected
state estimation - accuracy + precision as expected, speed worse then expected

biggest challenge 
speed of state estimation
It was soon found, however, that with the given hardware a (x,y,theta) Duckiebot state update took several seconds to compute. With the existing lane controller, this time lag proved insufficient: by the time a new state update was calculated and published to the lane controller, the Duckiebot had already deviated substantially from this published state. 
finding problems in old pipeline

## Part 6: Future avenues of development

As mentioned before, the main area of work needed to get the parking pipeline working is to successfully implement some sort of control in order to autonomously park a Duckiebot. Currently, there are three main options for doing this, described in the following sections. Each avenue of development may be explored individually or a combination of multiple could prove to be the best way forward.

1) Increase the speed of the state estimation

* As seen in the part 5 of this report, there is a considerable time lag in the state estimation via AprilTags. An avenue of investigation should be to look deeper into the `extractTags` method from the AprilTag C++ library. The node where this method is called is found here, on line 67:

`DUCKIETOWN_ROOT/catkin_ws/src/20-indefinite-navigation/apriltags_ros/apriltags_ros/src/apriltag_detector.cpp`

* Any development which can increase the speed of this 'extractTags' method, which takes a grayscale image and detects tag number(s) in view, would be very beneficial for an increased state estimate frequency. 

* Another avenue of development may be to increase the computing power of the Duckiebot. The parking pipeline was currently run on a Raspberry Pi. A more powerful computer may improve the time lag issue.

2) Successful integration of state propagation 

* More development could be made on the `devel_path_planning_node` node that propagates the state estimate at a high frequency for use with the lane controller. Please see the "State Propagation" section of part 4 for how this algorithm is intended to work. 
* As of the writing of this report, the parking group was unable to successfully integrate the state propagation. More work is needed to allow the algorithm to work as designed. 

3) Development of a dedicated parking controller

* 1) and 2) above rely on the use of the lane controller while parking. It may be beneficial, however, to develop a dedicated parking controller which can better handle the parking feature pipeline.






