# Coordination with Implicit Communication 

The goal of this project is to allow for coordination without any form of explicit communication between the duckiebots, such as LEDs or wireless communication. 

## Detection

## Tracking

The detection algorithm provides the coordinates of the vertices of bounding box in the camera frame, along with a measure of confidence in the detection. In the context of coordination, this information is only meaningful if the bounding boxes can be tracked from frame to frame, ultimately allowing the duckiebot to predict the movement of any duckiebot within its field of view. In the context of this document, the duckiebot performing the tracking operation is called the tracker, whereas the duckiebots being tracked are referred to as the targets. The tracking algorithm requires solving two main problems. First, a data association problem must be solved. This is required to match the bouding boxes in the previous frame with the bounding boxes in the current frame. Second, a state estimation problem must be solved to smooth out the measurements and increase the robustness of the detection algorithm. A Kalman Filter is used to do this. The Kalman Filter will use the kinematics of each target duckiebot to propagate the state estimate through time. Then, using the measurements from the detection algorithm, the estimate will be corrected. 

### Measurement Acquisition

Before elaborating on the details of the Kalman Filter, it is necessary to define how the correcting measurements are obtained. In order to be useful, the coordinates of the bounding boxes must be converted to a position in the robot frame. Then, each measurement must be associated to a measurement in the previous frame, in order to allow for proper tracking. Lastly, the cases of the amount of detections increasing or decreasing from frame to frame must be dealt with. 

In theory, the bottom of each bounding box should lie along the ground plane. Thus, using the homography matrix, the coordinates of two points $mathbf{p}_1^i$ and $mathbf{p}_2^i$ (corresponding to the bottom of bounding box $i$) can be found in the ground plane. The midpoint of this line, defined by 

\begin{align*}
	\mathbf{p} = (\frac{\mathbf{p}_{1_x} - \mathbf{p}_{2_x}}{2}, \frac{\mathbf{p}_{1_y} - \mathbf{p}_{2_y}}{2}),  
\end{align*}

is chosen as an estimate of the position of target $i$. This was found to be robust to changes in pose, providing a smooth estimate of the position. 

Assuming the frame at time $t_{k-1}$ contains $n$ bounding boxes, corresponding to $n$ detected targets, there are three possible cases for the frame at time $t_k$. Either there are $n$ detected targets, there are fewer than $n$ detected targets, or there are more than $n$ detected targets. 

### Time Propagation

Denoting the state of target $i$ at time $t_k$ as $\boldsymbol{\mu}^i_k = [x_k^i,y_k^i,v_{x_{k}}^i,v_{y_{k}}^i]^T$  the discrete time kinematics of each target are 

\begin{align*}
	x_k^i = x_{k-1}^i + Tv_{x_{k-1}}^i \\
	y_k^i = y_{k-1}^i + Tv_{y_{k-1}}^i \\
	v_{x_k}^i = v_{x_{k-1}}^i + Ta_{x_{k-1}}^i \\
	v_{y_k}^i = v_{y_{k-1}}^i + Ta_{y_{k-1}}^i.
\end{align*}

Here, $\mathbf{p}_k^i [x_k^i y_k^i]^T$, $\mathbf{v}_k^i [v_{x_k}^i v_{y_k}^i]^T$ and  $\mathbf{a}_k^i [a_{x_k}^i a_{y_k}^i]^T$ are the position, velocity and acceleration of target $i$ resolved in the robot frame and T is the integration time step. In state space form, this becomes

\begin{align*}
    \begin{bmatrix}%
 	  \mathbf{p}_k^i \\ \mathbf{v}_k^i
    \end{bmatrix}
    =
    \mathbf{1}_{4 \times 4}
    \begin{bmatrix}%
 	   \mathbf{p}_{k-1}^i \\  \mathbf{v}_{k-1}^i
    \end{bmatrix}
    + T\mathbf{1}_{4 \times 4}
    \begin{bmatrix}%
 	  \mathbf{v}_{k-1}^i \\ \mathbf{a}_{k-1}^i
    \end{bmatrix} .    
\end{align*}

In order to propagate the state estimate through time, it is assumed that a measurement of the acceleration of each target is available, despite this not being the case. It will be assumed that the acceleration of each target is nominally 0 m/s^2 in both the $x$ and $y$ directions. Denoting the measured acceleration as $\mathbf{u}_k^i$, the measurement model is

\begin{align*}
	\mathbf{u}_k^i = \mathbf{a}_k^i + \mathbf{w}_k^i,
\end{align*}

where $\mathbf{w}_k^i$ is defined as a Gaussian random variable with mean $\mathbf{0}$ and covariance $\mathbf{Q}_k^i$. From the assumptions, we can then write this as 

\begin{align*}
	\mathbf{u}_k^i = \mathbf{w}_k^i.
\end{align*}

Therefore, denoting $\mu = [\mathbf{p}_k^i \mathbf{v}_{k-1}^i]^\textsf{T}$ a state-space representation of the kinematics is

\begin{align*}
    \begin{bmatrix}%
 	  \mathbf{p}_k^i \\ \mathbf{v}_k^i
    \end{bmatrix}
    =
    \mathbf{1}_{4 \times 4}
    \begin{bmatrix}%
 	   \mathbf{p}_{k-1}^i \\  \mathbf{v}_{k-1}^i
    \end{bmatrix}
    + T\mathbf{1}_{4 \times 4}
    \begin{bmatrix}%
 	  \mathbf{v}_{k-1}^i \\ \mathbf{a}_{k-1}^i
    \end{bmatrix} .    
\end{align*}

## Prediction
