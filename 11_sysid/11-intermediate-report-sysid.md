#  System ID: Intermediate Report {#template-int-report status=ready}
<!--
_It's time to commit on what you are building, and to make sure that it fits with everything else._

This consists of 3 parts:

- Part 1: System interfaces: Does your piece fit with everything else? You will have to convince both system architect and software architect and they must sign-off on this.

- Part 2: Demo and evaluation plan: Do you have a credible plan for evaluating what you are building? You will have to convince the VPs of Safety and they must sign-off on this.

- Part 3: Data collection, annotation, and analysis: Do you have a credible plan for collecting, annotating and analyzing the data? You will have to convince the data czars and they must sign-off on this.
-->

## Part 1: System interfaces

<!--
Please note that for this part it is necessary for the system architect and software architect to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.
-->

### Logical architecture

**Desired functionality**    
The desired functionality is a node that takes the desired linear and angular velocity as input and maps it to the input voltages for the motors.


**Approach**    

In a first step we measure manually the longitudinal and angular velocity for given input voltage.  The aim is to use these measurements to find the $ c_r $, $ c_l $ and $ L $ (or $ c $, trim, $ L $) of our model that was introduced in the preliminary design document.



\begin{align} \label{eq:mod-kin-5}
\left\{  \begin{array}{l} \dot x_A^R &= (c_r V_r+c_l V_l)/2  \\
                          \dot y_A^R &= 0 \\
                          \dot \theta &= \omega = (c_r V_r-c_l V_l)/(2L) \end{array} \right.,
\end{align} 

This  “linear” velocity to voltage function can be used for testing by the controller group.
If we manage the first step, we will move on to a second step. Here we will aim to get a “non-linear” velocity to voltage map. There will be a calibration procedure that creates a custom velocity to voltage map for each Duckiebot that should be independent for different hardware configurations.
The current idea for the calibration procedure is to drive the Duckiebot with different voltage commands to the motor while a checkerboard is in its viewfield. The logs can then be used to extract a pose estimation of the Duckiebot compared to the checkerboard.  The recorded voltage and the pose data will then be fitted in a nonlinear manner, maybe using the acado optimization toolbox.


**Assumptions**

*** Ground truth estimation ***

- Repeatability (currently investigating)

- Can get ground truth also while driving (currently investigating)

- Minimum distance from which each square is visible is sufficient to do calibration trajectories with the Duckiebot (currently investigating)

- The calibration procedure gives very accurate ($\leq 5 mm$) pose estimates to do a decent mapping

*** Performance ***

- Our assumption is that once calibrated the Duckiebot will be able to repeat the same behavior, and the kinematics do not change.


### Software architecture

** Calibration procedure Node **

To achieve the desired functionality, we will create a mapping from velocity states to input motor voltages that will be used by the control. The procedure that will be followed is summarized as follows:

- Drive the Duckiebot with different open loop voltage commands to the motor on a 2x2 tile while a checkerboard is in its view field. The voltage commands should be chosen to allow sufficient exploration of the velocity states to input voltages mapping. Take a log of the motion for offline processing in the next step.

- Process the logs of the images to extract a pose estimation of the Duckiebot using the checkerboard as a reference.  

- Fit The recorded voltage and the pose data in a nonlinear manner potentially using the Acado optimization toolbox.

*** Input ***

- Intrinsic camera calibration

- Extrinsic camera calibration

- Rosbag of raw images and control commands of the Duckiebot driving in front of the checkerboard, the rosbag is recorded by running the calibration node


*** Output ***

- Control commands during calibration procedure (topic: car_cmd, same as control)

- calibration.yaml file containing velocity to voltage map

*** Assumed Latency ***

Offline. The calibration procedure is done on the computer

*** Position estimate ***

In order to to identify a system model, we need the best possible state estimation. This shall be achieved by calculating the camera extrinsics from the checkerboard for each frame. The picture below shows our first experiments with the setup.

<div figure-id="fig:state-est" figure-caption="Setup for state estimation">
  <img src="state_estimation.jpg" style='width: 30em; height:auto'/>
</div>


** Inverse_kinematics_node **
We will edit the existing inverse_kinematics_node. It will get the desired velocity as input and find the corresponding motor speeds using the parameters from the calibration.yaml file generated by the Calibration procedure. For desired velocities that exceed the system's capabilities, the maximum possible velocity will be returned.

*** Subscribed message ***

car_cmd

*** Published message ***

wheels_cmd

*** Assumed Latency ***

Negligible, will run a very lightweight callback directly once it receives a car_cmd message



## Part 2: Demo and evaluation

### Demo plan

The main goal is to demonstrate improved calibration. For this purpose, we will first run the lane following module with a Duckiebot that has the default velocity to voltage mapping on the same test track, see picture.
Afterwards, we will run the improved calibration procedure that will create the custom velocity to voltage mapping. We will then run the lane following module again and see how lane following task is improved.
As the actual calibration procedure will take sometime, we will do it beforehand.


<div figure-id="fig:track" figure-caption="Duckietown test track">
  <img src="duckietown.png" style='width: 30em; height:auto'/>
</div>


## Plan for formal performance evaluation

We will run all of the tests 3x times uncalibrated, and 3x calibrated for the following Duckiebot configuration:

- Normal Duckiebot

- Duckiebot with different right and left wheel diameters
and compare the improvements

** Offset in straight line **  :

We will let the Duckiebot drive straight in open loop and measure its offset after X tiles of straight lane in Duckietown. The performance metric will be the absolute position offset of the expected to the actual terminal position after the run, measured with a ruler.

** Circle test ** : 

We will will drive the Duckiebot with a constant velocity $ v_a $ and constant angular velocity $ \dot \omega $ in open loop on a Duckietown corner tile. We will compare the actual path with the desired path. This is done both clock and counterclockwise. The performance metric will be the absolute position offset of the expected to the actual terminal position after the run, measured with a ruler.

** Integration test ** : 

We want to test how the improved calibration affects the line following mode. Compare different behaviors in line-following mode.

** Material needed to do calibration **

- 4 Black tiles

- Normal Camera Calibration Checkerboard that can be attached vertically


** Material needed to do performance test **

- Duckietown with straights and corners

- Ruler to measure offset of terminal position

## Part 3: Data collection, annotation, and analysis

### Collection

- How much data do you need?

At least one log file (rosbag) of the Duckiebot being driven with various voltage inputs and has a camera calibration checkerboard in view at all times.  The checkerboard will be used to estimate position from images. We’ll write the code for pose estimation (eg. assemble existing libraries).


- How are the logs to be taken? (Manually, autonomously, etc.)

Initially logs will be taken manually, but later will be taken automatically by a calibration node. The Duckiebot, should be on a Duckietown tile and have the checkerboard in view.


- Do you need extra help in collecting the data from the other teams?

We do not need data from other teams and therefore do not need help.


### Annotation

- Do you need to annotate the data?

No, we need to extract pose from images, which is done by geometry from a checkerboard, not annotation

- At this point, you should have you tried using thehive.ai to do it. Did you?

No, because we do not need any annotations.

## Analysis

- Do you need to write some software to analyze the annotations?


 We don’t need data annotation since we can do all the benchmarking by our own. However we are writing the software to estimate the model based on data collection.

We already did a basic analysis of the system by running control commands and measuring the distance or angle by hand. The duration and magnitude of the control commands were extracted from a rosbag. This lets us generate a map from control to velocities, thus basically we have a first guess of the model parameters (average C, C/2L). The results can be found in the two plots below. The estimated speed of the Duckiebot in lane following mode is 0.27 m/s and the yaw rate at maximum control input 3.7 rad/s. The estimate control gains are 0.67 m/s ($ \pm 15\%$) per $ v_{cmd} $ and 0.45 rad/s ($ \pm 30\% $) per $\omega_{cmd} $.


<div figure-id="fig:vel" figure-caption="Estimated linear velocity">
  <img src="velocity_fit.jpg" style='width: 30em; height:auto'/>
</div>


<div figure-id="fig:yaw" figure-caption="Estimated yaw rate">
  <img src="omega_fit.jpg" style='width: 30em; height:auto'/>
</div>







