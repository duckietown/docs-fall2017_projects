#  System identification: final report {#sysid-final-report status=draft}



## The final result {#sysid-final-result}

<div figure-id="fig:demo_succeeded-sysid">
    <figcaption>Demo of the calibration procedure
    </figcaption>
    <dtvideo src='vimeo:251027149'/>
</div>

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
 <col2 id='sysid-notations1' figure-id="tab:sysid-notations1" figure-caption="Notations for odometry calibration a differential drive robot">
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
    \tau(t)=\frac{K_i}{R}(V-\frac{K_b}{r}v)  \label{eq:tau}
\end{align}

At steady state $J \ddot{\varphi}=\frac{J}{r} \dot{v}=\tau-\tau_{dist}=0$. If disturbances $\tau_{dist}$ are ignored, $\Rightarrow \tau=0$.
This leads to the following simplification:

\begin{equation}
    v_{l,r}=\frac{r_{l,r}}{K_{b_{l,r}}}V_{l,r}=c_{l,r}V_{l,r}          \label{eq:one}
\end{equation}


From the kinematic model it is known that the following holds true for our assumptions:
\begin{align}
    v_l &= (v_{A}-\omega L)       \label{eq:v_l}\\
    v_r &= (v_{A}+\omega L)        \label{eq:v_r}
\end{align}

By defining trim $t=\frac{c-1}{c+1}$ with $c=\frac{c_{r}}{c_{l}}$ and $g=g_0=1$ the trim and gain model can be defined:
\begin{align}
    V_{l} &= (g+t)(v_{A}-\omega L)  \label{eq:V_l}     \\
    V_{r} &= (g-t)(v_{A}+\omega L)  \label{eq:V_r}
\end{align}


<div markdown="1">

 <col2 id='sysid-notations2' figure-id="tab:sysid-notations2" figure-caption="Notations for odometry calibration a differential drive robot">
    <s>$V_{l,r}$</s>  <s>Voltage to left/right motors</s>
    <s>$g$</s>  <s>Gain</s>
    <s>$t$</s>  <s>Trim</s>
    <s>$v_{A}$</s>  <s>Linear velocity of Duckiebot in bodyframe</s>
    <s>$\theta,\dot{\theta}=\omega$</s>  <s>Vehicle orientation and angular velocity</s>
    <s>$c_{l,r}$</s>  <s>"Wheel" calibration parameters</s>
    <s>$R_{l,r}$</s>  <s>Wheel radii</s>
    <s>$k_{b_{l,r}}$</s>  <s>Motor constants</s>
    <s>$L$</s>  <s>Half of distance between the two wheels</s>
 </col2>
 
</div>



Note that if the gain $g = 1.0$ and trim $t= 0.0$, the wheel’s voltages are exactly the same as the linear velocity + or - angular velocity times half the baseline length $V_{l,r}=v_a \pm \omega L$.
With gain $g > 1.0$ the vehicle goes faster given the same velocity command, and for gain $g < 1.0$ it would go slower.
With trim $t > 0$, the right wheel will turn slightly more than the left wheel given the same velocity command; with trim $t<0$, the left wheel will turn slightly more the right wheel.

#### Existing Calibration Procedure


The current implementation of the calibration procedure can be found in the [#wheel-calibration](#wheel-calibration) section.

Hereby, the Duckiebot is placed on a line(e.g. tape). Afterwards the joystick demo is launched with the following command:

    duckiebot: $ roslaunch duckietown_demos joystick.launch veh:=${VEHICLE_NAME}

Now the human operator commands the Duckiebot to go straight for around 2m.
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

* Differential-drive model [#duckiebot-modeling](#duckiebot-modeling)
    
* Pinhole-camera model [#camera-geometry](#camera-geometry)



## Definition of the problem {#sysid-final-problem-def}

### Kinematics
The previously defined equations \eqref{eq:V_l} and \eqref{eq:V_r} can be rewritten:

\begin{align}
    v_{a} &= \frac{c_r V_r+c_l V_l}{2}  \label{eq:v_a}     \\
    \dot{\theta} &= \frac{c_r V_r-c_l V_l}{2L}  \label{eq:omega}
\end{align}

This can be rewritten by defining $c=c_r$ and $c_l=c+\Delta c$ and we get from \eqref{eq:v_a2} and \eqref{eq:omega2}:


\begin{align}
    v_{a} &= \frac{c \cdot (V_r+V_l)+\Delta c \cdot V_l}{2}  \label{eq:v_a2}     \\
    \dot{\theta} &= \frac{c \cdot (V_r-V_l)-\Delta c \cdot V_l}{2L}  \label{eq:omega2}
\end{align}

With this last equation, we see that it we set $V_r=V_l $ , then $ θ̇ = -\frac{\Delta c}{2L} \cdot V_l$

### Calibration from velocity estimation

A first way of estimating the parameters is directly by trying to approximate the linear and angular velocities $v_{a_{i}}$ and $\omega_i$ (in inertial frame)of the duckiebot from the position ($d_i,\omega_i$) we get at every time $t_i$.
By the finite difference formula:

\begin{align}
    v_{a_{i}} & = \frac{d_{i+1}-d_{i}}{T_{i+1}-T_{i}}  \label{eq:v_a_i}     \\
    \dot{\theta}_{i}   & = \frac{\theta_{i+1}-\theta_{i}}{T_{i+1}-T_{i}}  \label{eq:omega_i}
\end{align}

Once the velocities $v_{a_{i}}$,$\dot{\theta}_{i}$ have been measured for each voltage commands $V_{l,i}$,$V_{r,i}$, we try to fit the measurements with our model, namely the equations \eqref{eq:v_a2} and \eqref{eq:omega2} at every time $t_i$.

For a "straight" line $V = V_l = V_r$, and neglecting $\Delta c$ in \eqref{eq:v_a}, \eqref{eq:v_a2} and \eqref{eq:omega2} become

\begin{align}
    v_{a} &= c \cdot V  \label{eq:v_a3}     \\
    \dot{\theta} &= \frac{-\Delta c \cdot V}{2L}  \label{eq:omega3}
\end{align}

Or:

\begin{align}
    c  &=\frac{v_{a}}{V}  \label{eq:v_a4}     \\
   \Delta c  &= \frac{\dot{\theta}2L}{ V}  \label{eq:omega4}
\end{align}

By manually calculating the semi axis length $L$, we can directly estimate all the parameters, and therefore calibrate the Duckiebot.

### Calibration from position estimation

Alternatively, rather than estimating the velocity, we can estimate the position of the Duckiebot ($x_i,y_i$) recursively with the formula:

\begin{align}
    x_{k+1} & = x_k+v_A \cdot cos(\theta)  \label{eq:1}     \\
    y_{k+1} & = y_k+v_A \cdot sin(\theta)  \label{eq:2}
\end{align}

Because $v_A=\frac{c_r \cdot V_r+c_l \cdot V_l}{2}$ we can express every position $x_i,y_i$ of the Duckiebot with help of the parameters:

\begin{align}
    x_{k+1} & = x_k+\frac{c_r \cdot V_r+c_l \cdot V_l}{2} \cdot cos(\theta)  \label{eq:xk}     \\
    y_{k+1} & = y_k+\frac{c_r \cdot V_r+c_l \cdot V_l}{2} \cdot sin(\theta)  \label{eq:yk}
\end{align}

By minimizing the position of the Duckiebot $x_i,y_i$ and its theoretical position given by our model $x_i,y_i$ at every time $t_i$ , we can estimate the parameters $c_r , c_l$ and $L$.

We can as example use the L2-norm :

\begin{equation}
    \begin{pmatrix}c_{l}^{\star}\\c_{r}^{\star}\\L^{\star}\end{pmatrix}= \underset{c_l,c_r,L}{\mathrm{argmin}}   \begin{pmatrix} x_1-\tilde{x}_1\\y_1-\tilde{y}_1\\\vdots\\x_n-\tilde{x}_n\\y_n-\tilde{y}_n\end{pmatrix}^T Q   \begin{pmatrix} x_1-\tilde{x}_1\\y_1-\tilde{y}_1\\\vdots\\x_n-\tilde{x}_n\\y_n-\tilde{y}_n\end{pmatrix}
    
\end{equation}

With a matrix $Q$ that weights the importance of each measurement (it can be the identity matrix, if all measurements are equally weighted).


### Dealing with uncertainty

Because our model does not take into account the dynamics of the system, and many assumptions as been made, the results we will get won’t perfectly match with the reality.
Assuming that the states estimation $v_{a_{i}}$ and $\dot{\theta}_{i}$ is accurate enough (an hypothesis that will have to be tested)
and a Gaussian distribution of the noise, we can quantify this nose by estimating its variance :


\begin{align}
    \sigma_v^2 & = \frac{1}{n}  \sum_{k=1}^n (v_{A_{i}}-\tilde{v}_{A_{i}})^2  \label{eq:sigmav}     \\
    \sigma_{\dot{\theta}}^2 & = \frac{1}{n}  \sum_{k=1}^n (\dot{\theta}_{A_{i}}-\tilde{\dot{\theta}}_{A_{i}})^2   \label{eq:sigmatheta}
\end{align}

with 

\begin{align}
    \tilde{v}_{A_{i}} & = \frac{c_{r}^{\star} V_{r_i}+c_{l}^{\star} V_{l_i}{2} \label{eq:vtilde}     \\
    \tilde{\dot{\theta}}_{A_{i}} & = \frac{c_{r}^{\star} V_{r_i}-c_{l}^{\star} V_{l_i}}{2L^{\star}}   \label{eq:thetatilde}
\end{align}


## Contribution / Added functionality {#sysid-final-contribution}


### Pose estimation w.r.t. chessboard from successive image


#### Step 1 - Find 2D image points of chessboard in picture

To to find pattern in chess board, we use the function, [findChessboardCorners](https://docs.opencv.org/3.0-beta/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#drawchessboardcorners "OpenCV findChessboardCorners")
It gives us the image points locations where two black squares touch each other in chess boards)

We also need to pass what kind of pattern we are looking, like 8x8 grid, 5x5 grid etc. In this example, we use 7x6 grid. (Normally a chess board has 8x8 squares and 7x7 internal corners). It returns the corner points and retval which will be True if pattern is obtained. These corners will be placed in an order (from left-to-right, top-to-bottom)

If the chessboard is found in the image, the corners position are refined with the cv2.cornerSubPix() function.

 
#### Step 2 - Find the rotation and translation vectors of chessboard w.r.t camera 

The next steps is to find the relation between the found image points and the objects points, the so called extrensic parameters.
Extrinsic parameters corresponds to rotation and translation vectors which translates a coordinates of a 3D point to a coordinate system.
3D points are called object points and 2D image points are called image points.

The chessboard was kept stationary at XY plane, (so $Z=0$ always) and camera was moved accordingly. This consideration helps us to find only $X$,$Y$ values. Now for $X$,$Y$ values, we can simply pass the points as $(0,0), (1,0), (2,0), ...$ which denotes the location of points. In this case, the results we get will be in the scale of size of chess board square. But if we know the square size, (Duckie chessboard $32 mm$), and we can pass in our case the values as $(0,0),(32,0),(64,0),...,$ we get the results in mm. 

For the pose estimation we need to know the extrinsic and extrinsic parameters of the camera.
They can be loaded with the implemented load_camera_info() function.
Intrinsic parameters are specific to a camera. It includes information like focal length ($f_x$,$f_y$), optical centers ($c_x$,$c_y$) etc. It is also called camera matrix. It is expressed as a 3x3 matrix:


\begin{equation}
    camera \; matrix = \left [ \begin{matrix} f_x & 0 & c_x \\ 0 & f_y & c_y \\ 0 & 0 & 1 \end{matrix} \right ]
\end{equation}


We are using the function cv.solvePnPRansac() to calculate the rotation and translation. [solvePnPRansac](https://docs.opencv.org/3.0-beta/modules/calib3d/doc/camera_calibration_and_3d_reconstruction.html#solvepnpransac "OpenCV solvePnPRansac") finds an object pose from 3D-2D point correspondences using the RANSAC scheme.
It uses the object points, the chessboard corners and the camera matix as an input and gives the translation $t_{CamChess}$ and rotation $R^{Cam}_{Chess}$

#### Step 3 - Find the rotation and translation vectors of duckiebot w.r.t chessboard

We are interested to find the vehicle pose with respect to world frame. 
This is done by using the homography relation:


\begin{equation}
                       T^{I}_{cam}=T^{I}_{chess} T^{chess}_{cam}
\end{equation}

where as we use the following equations:


\begin{align}
    T^{world}_{veh} & = \begin{bmatrix}R^{I}_{veh}&t_{I  Veh}\\0&1\end{bmatrix}     \label{eq:worldveh}     \\
    T^{chess}_{cam} & = \begin{bmatrix}R^{chess}_{cam}&t_{ChessCam}\\0&1\end{bmatrix} = \begin{bmatrix}R^{chess}_{cam}^T&-R^{chess}_{cam}^T t_{CamChess}\\0&1\end{bmatrix} \label{eq:chesscam}
\end{align}

The yaw $\theta$ can then be extracted with the implemented rot2euler() function.






### Fit

### Integration



## Formal performance evaluation / Results {#sysid-final-formal}

### Plan for Performance evaluation

** Pose estimation w.r.t. chessboard from successive images**  :

Accuracy

Precision



** Offset in straight line **  :

We will let the Duckiebot drive straight in open loop and measure its offset after X tiles of straight lane in Duckietown. The performance metric will be the absolute position offset of the expected to the actual terminal position after the run, measured with a ruler.

** Circle test ** : 

We will will drive the Duckiebot with a constant velocity $ v_a $ and constant angular velocity $ \dot \omega $ in open loop on a Duckietown corner tile. We will compare the actual path with the desired path. This is done both clock and counterclockwise. The performance metric will be the absolute position offset of the expected to the actual terminal position after the run, measured with a ruler.

** User friendliness ** : 






## Future avenues of development {#sysid-final-next-steps}

* Simultaneous odometry and camera calibration

* Position estimation based on april tags

* Caster wheel identification
    * Initial aim was to include the kinematics of the caster wheel, however due to time constraint, we sticked to the roller wheel

