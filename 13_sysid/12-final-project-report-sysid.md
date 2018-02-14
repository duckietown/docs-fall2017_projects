#  System identification: final report {#sysid-final-report status=draft}



## The final result {#sysid-final-result}

INSERT VIDEO OF FINAL OUTCOME

To reproduce the results see the  [operation manual](#demo-sysid) which includes detailed instructions for a demo.


## Mission and Scope {#sysid-final-scope}

Define what is your mission here.


### Motivation {#sysid-final-result-motivation}


The mission is to make the controller more robust to different configurations of the robot. One approach to do this is obtaining a mathematical model of the Duckiebot in order to understand its behavior. The mathematical model can then be used to design a controller to obtain robust desired behaviors and performances.

The Duckiebot is in a differential-drive configuration. It actuates each wheel with a separate DC Motor. By applying the same torque on both wheels one can go straight, and by applying different torques the Duckiebot turns. A schematic overview of the model can be seen in Figure [](#fig:model).

<div figure-id="fig:model" figure-caption="Model of differential drive robot [](#bib:Modeling)">
  <img src="model.PNG" style='width: 30em; height:auto'/>
</div>

A model for a differential drive robot is derived in section in chapter [#duckiebot-modeling](#duckiebot-modeling). This model can be used to provide a simple method of maintaining an individual’s position or velocity estimate in the absence of computationally expensive position updates from external sources such as the mounted camera.

The derived non-linear model describes the expected output of the pose (e.g. position, velocity) w.r.t. a fixed inertial frame for a certain voltage input. The model makes several assumptions, such as rigid body motion, symmetry, pure rolling and no lateral slipping.

Most important of all, the model assumes the knowledge of certain constants that characterize the DC motors as well as the robot’s geometry.

However, there will never be two duckiebots that show exactly the same behavior. This can be very problematic. 
You might have noticed that your vehicle doesn’t really go in a straight line when you command it to. 
For example, when the same voltage is supplied to each motor, the Duckiebot will not go straight as might expected. 
Also, the vehicle might not go at the velocity you are commanding it to drive at.
Therefore, these constants needs to be identified individually for each single robot. The determination process to do so is called system identification. 
Odometry calibration is the process of determining model parameter to “match” predicted motion and measurements.  For the duckiebot odometry calibration is the process aimed at identifying the kinematic parameters used to reconstruct the robot’s absolute configuration from the given voltage input. 

Increasing the accuracy of the Duckiebot’s odometry will result in reduced operation cost as the robot requires fewer absolute positioning updates with the camera.

### General odometry formulation {#sysid-final-odometry}

The general problem definition for the odometry is to find the most likely calibration parameters given the duckiebot model [#duckiebot-modeling](#duckiebot-modeling) and a set of discrete measurement from which the output can be estimated. [](#bib:OdometryCalibration) 
The model of the system [](#bib:OdometryCalibration) with the notations explained in Table [](#tab:sysid-notations) can be described as :

\begin{align}
    \dot{x} &= f(p;x,u)       \label{eq:model1} \\
      y & = g(x)       \label{eq:model2} \\
      \mathcal{M}_{n} & = \{ m_k=m(t_k), t_1 < \dots < t_k < \dots < t_n)\}     \label{eq:measurements} \\
      \hat{\mathcal{Y}}_{n} & = \{ \hat{y}_{k}=h(m_k),k=1, \dots ,n \}             \label{eq:outputestimates}
\end{align}


<div markdown="1">
 <col2 id='sysid-notations' figure-id="tab:sysid-notations" figure-caption="Notations for odometry calibration a differential drive robot">
    <s>$p$</s>  <s>Calibration Parameters</s>
    <s>$f(\cdot)$</s>  <s>Model </s>
    <s>$g(\cdot)$</s>  <s>Pose </s>
    <s>$\mathcal{M}_n$</s>  <s>Set of discrete measurements</s>
    <s>$m_k$</s>  <s>Measurements (not necessarily evenly space in time)</s>
    <s>$\hat{\mathcal{Y}}_{n}$</s>  <s>Set of output estimates</s>

 </col2>
</div>


The model $f(\cdot)$ can be a kinematic model, constrained dynamic model or more general dynamic model.
The pose $g(\cdot)$ can be the robot pose or the sensor pose.
The measurements $m_k$ can be from "internal" sensors e.g. wheel encoders, IMUs etc. or from "external" sensors such as Lidar, Infrared or camera.

### Typical solution: Odometry calibration with wheel encoders {#sysid-final-encoders}

The most spread odometry calibration is done with $f(\cdot)$ being a kinematic model as well as using measurements from $m_k$ wheel encoders.
A detailed describtion can be found in 
However, the Duckiebot does not have wheel encoders. Therefore, this approach is unsuitable.

### Existing solution {#sysid-final-literature}


#### Forward Kinematics

<div figure-id="fig:mod-kin" figure-caption="Schematics of differential drive robot [](#bib:Modeling)">
  <img src="mod-kin.png" style='width: 30em; height:auto'/>
</div>

We make the following assumptions to gain a simplified relationship between input voltage and resulting left and right wheel velocities. 
A more thorough derivation can be found in [#duckiebot-modeling](#duckiebot-modeling).

\begin{align}
    \tau(t)=\frac{K_i}{R}(V-\frac{K_b}{r}v) 
\end{align}

At steady state $J \ddot{\varphi}=\frac{J}{r} \dot{v}=\tau-\tau_{dist}=0$. If disturbances $\tau_{dist}$ are ignored, $\Rightarrow \tau=0$.
This leads to the following simplification:

\begin{align}
    v_{l,r}=\frac{r_{l,r}}{K_{b_{l,r}}}V_{l,r}=c_{l,r}V_{l,r}
\end{align}


From the kinematic model it is known that the following holds true for our assumptions:
\begin{align}
    v_l &= (v_{A}-\omega L)       \\
    v_r &= (v_{A}+\omega L)
\end{align}

By defining trim $t=\frac{c-1}{c+1}$ with $c=\frac{c_{r}}{c_{l}}$ and $g=g_0=1$ the trim and gain model can be defined:
\begin{align}
    V_{l} &= (g+t)(v_{A}-\omega L)       \\
    V_{r} &= (g-t)(v_{A}+\omega L)
\end{align}


<div markdown="1">
 <col2 id='sysid-notations' figure-id="tab:sysid-notations" figure-caption="Notations for odometry calibration a differential drive robot">
    <s>$V_{l,r}$</s>  <s>Voltage to left/right motors</s>
    <s>$g$</s>  <s>Gain</s>
    <s>$t$</s>  <s>Trim</s>
    <s>$v_{A}$</s>  <s>Linear velocity of Duckiebot in bodyframe</s>
    <s>$\omega$</s>  <s>Angular velocity of Duckiebot in bodyframe</s>
    <s>$c_{l,r}$</s>  <s>"Wheel" calibration parameters</s>
    <s>$R_{l,r}$</s>  <s>Wheel radii</s>
    <s>$k_{b}_{l,r}$</s>  <s>Motor constants</s>
    <s>$L$</s>  <s>Half of distance between the two wheels</s>
 </col2>
</div>

Note that if the gain $g = 1.0$ and trim $t= 0.0$, the wheel’s voltages are exactly the same as the linear velocity + or - angular velocity times half the baseline length $V_{l,r}=v_a \pm \omega L$.
With gain $g > 1.0$ the vehicle goes faster given the same velocity command, and for gain $g < 1.0$ it would go slower.
With trim $t > 0$, the right wheel will turn slightly more than the left wheel given the same velocity command; with trim $t<0$, the left wheel will turn slightly more the right wheel.

#### Existing Calibration Procedure

The current implementation launches the joystick demo:

duckiebot: $ roslaunch duckietown_demos joystick.launch veh:=${VEHICLE_NAME}

If the Duckiebot drifted to the left side of the tape, decrease the value of $t$, for example:


    duckiebot: $ rosservice call /${VEHICLE_NAME}/inverse_kinematics_node/set_trim -- 0.01

Or Changing the trim in a negative way, e.g. to -0.01:

    duckiebot: $ rosservice call /${VEHICLE_NAME}/inverse_kinematics_node/set_trim -- -0.01

This procedure is repeated until there is less than around $10 cm$ drift for two meters distance.
The speed of the duckiebot can be adjusted by setting the gain:

    duckiebot: $ rosservice call /${VEHICLE_NAME}/inverse_kinematics_node/set_gain -- 1.1

The parameters of the Duckiebot are saved in the duckietown/config/baseline/calibration/kinematics/{VEHICLE_NAME}.yaml file.

### Opportunity {#sysid-final-opportunity}

#### Current shortcomings
* Human in the loop
    * The car is not able to calibrate itself without human input
* Item2
    * ...
* Item3
    * ...



#### Possible approaches

A crucial step should be to take the human out of the loop.
This means that the car will calibrate itself, without any human input.

There were several possible approaches discussed to overcome the shortcomings of the current calibration:

* Localization based calibration
    * E.g. determine relative pose w.r.t. Chessboard from successive images
* Closed loop calibration
    * Modify the trim while Duckiebot is following a loop until satisfactory
* Motion blur based calibration
    * Reconstruct dynamics from blurred images




Identify motors steady-state parameters. Mapping between voltage and angular rate, i.e. mapping between voltage and velocity at once based on localization based calibration.

Assumptions: wheel radius is known and no slipping hypothesis is made.



### Preliminaries (optional) {#sysid-final-preliminaries}



## Definition of the problem {#sysid-final-problem-def}



## Contribution / Added functionality {#sysid-final-contribution}



## Formal performance evaluation / Results {#sysid-final-formal}



## Future avenues of development {#sysid-final-next-steps}

* Simultaneous odometry and camera calibration

