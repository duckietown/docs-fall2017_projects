# Coordination with Implicit Communication 

Implicit coordination of traffic at an intersection comprises the orchestration without any form of explicit communication of the entities involved, such as traffic lights or signs, in-vehicle signalization or vehicle-to-vehicle (to-infrastructure) communication systems. Thus,  the outcome of such mechanism is to produce an accurate inference of when it is safe to progress with a crossing maneuver.

As of today, Duckietown exhibits a less complex environment -compared to real-life situations- where the only mobile entities are duckiebots.  This simplification provides a favorable scenario to explore techniques at different levels of complexity which could be incrementally built to produce algorithms and heuristics applicable to more convoluted scenarios. 

Predicting traffic behavior at an intersection depends on accurately detect and track the position of each object as the preamble of applying prior information (traffic rules) for predicting the sequence of expected actions of each element. Hence, the conception of a mechanism that implicitly coordinates the individual behavior of a duckiebot under such circumstances comprises the research, design, and implementation of components capable of producing the required data for this outcome.

## Detection

## Tracking

The detection algorithm provides the coordinates of the vertices of bounding box in the camera frame, along with a measure of confidence in the detection. In the context of coordination, this information is only meaningful if the bounding boxes can be tracked from frame to frame, ultimately allowing the duckiebot to predict the movement of any duckiebot within its field of view. In the context of this document, the duckiebot performing the tracking operation is called the tracker, whereas the duckiebots being tracked are referred to as the targets. It is assumed in this project that tracker is stationary at an intersection. 

The tracking algorithm requires solving two main problems. First, a data association problem must be solved. This is required to match the bouding boxes in the previous frame with the bounding boxes in the current frame. Second, a state estimation problem must be solved to smooth out the measurements and increase the robustness of the detection algorithm. A Kalman Filter is used to do this. The Kalman Filter will use the kinematics of each target duckiebot to propagate the state estimate through time. Then, using the measurements from the detection algorithm, the estimate will be corrected. 

### Time Propagation

Denoting the state of target $i$ at time $t_k$ as $\boldsymbol{\mu}^i_k = [x_k^i,y_k^i,v_{x_{k}}^i,v_{y_{k}}^i]^\textsf{T}$  the discrete time kinematics of each target are 

\begin{align*}
	x_k^i = x_{k-1}^i + Tv_{x_{k-1}}^i \\
	y_k^i = y_{k-1}^i + Tv_{y_{k-1}}^i \\
	v_{x_k}^i = v_{x_{k-1}}^i + Ta_{x_{k-1}}^i \\
	v_{y_k}^i = v_{y_{k-1}}^i + Ta_{y_{k-1}}^i.
\end{align*}

Here, $\mathbf{p}_k^i = [x_k^i y_k^i]^\textsf{T}$, $\mathbf{v}_k^i = [v_{x_k}^i v_{y_k}^i]^\textsf{T}$ and  $\mathbf{a}_k^i = [a_{x_k}^i a_{y_k}^i]^\textsf{T}$ are the position, velocity and acceleration of target $i$ resolved in the robot frame and $T$ is the integration time step. In state space form, this becomes

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

In order to propagate the state estimate through time, it is assumed that a measurement of the acceleration of each target is available, despite this not being the case. It will be assumed that the acceleration of each target is nominally 0 $m/s^2$ in both the $x$ and $y$ directions. Denoting the measured acceleration as $\mathbf{u}_k^i$, the measurement model is

\begin{align*}
	\mathbf{u}_k^i = \mathbf{a}_k^i + \mathbf{w}_k^i,
\end{align*}

where $\mathbf{w}_k^i$ is defined as a Gaussian random variable with mean $\mathbf{0}$ and covariance $\mathbf{Q}$. From the assumptions, we can then write this as 

\begin{align*}
	\mathbf{u}_k^i = \mathbf{w}_k^i.
\end{align*}

Therefore, denoting $\boldsymbol{\mu}_k^i = [\mathbf{p}_k^i \mathbf{v}_{k-1}^i]^\textsf{T}$, a state-space representation of the kinematics is

\begin{align*}
	\boldsymbol{\mu}_k^i = \mathbf{F}_{k-1}\boldsymbol{\mu}_{k-1}^i + \mathbf{L}_{k-1}\mathbf{w}_{k-1}^i
\end{align*}

where

\begin{align*}
	\mathbf{F}_{k-1} = 
	\begin{bmatrix}%
		 1 & 0 & T & 0 \\ 
		 0 & 1 & 0 & T \\ 
		 0 & 0 & 1 & 0 \\ 
		 0 & 0 & 0 & 1 
	\end{bmatrix}
\end{align*}

and 

\begin{align*}
	\mathbf{L}_{k-1} = 
	\begin{bmatrix}%
		 0 & 0 \\ 
		 0 & 0 \\ 
		 1 & 0 \\ 
		 0 & 1 
	\end{bmatrix}
\end{align*}

### Measurment Model

Assuming a measurement is available at time $t_k$, the measurement $z_k^i$ can be expressed as

\begin{align*}
	\mathbf{z}_k^i = \mathbf{p}_k^i + \boldsymbol{\nu}_k^i,
\end{align*}

where $\boldsymbol{\nu}_k^i$ is defined as a Gaussian random variable with mean $\mathbf{0}$ and covariance $\mathbf{R}$. The measurements provided by the detection correspond to noise corrupted position measurements. This can also be written as

\begin{align*}
	\mathbf{z}_k^i = \mathbf{H}_{k-1}\boldsymbol{\mu}_{k-1}^i + \boldsymbol{\nu}_k^i
\end{align*}

where

\begin{align*}
	\mathbf{H}_{k-1}= 
	\begin{bmatrix}%
		 1 & 0 & 0 & 0 \\ 
		 0 & 1 & 0 & 0 \\ 
	\end{bmatrix}
\end{align*}

### Measurement Acquisition

Until this point, it was simply assumed that a measured position is available. However, obtaining this measurement is not trivial. In order to be useful, the coordinates of the bounding boxes must be converted to a position in the robot frame. Then, each measurement must be associated to a measurement in the previous frame, in order to allow for proper tracking. Lastly, the cases of the amount of detections increasing or decreasing from frame to frame must be dealt with. 

In theory, the bottom of each bounding box should lie along the ground plane. Thus, using the homography matrix, the coordinates of two points $\mathbf{p}_1^i$ and $\mathbf{p}_2^i$ (corresponding to the bottom of bounding box $i$) can be found in the ground plane. The midpoint of this line, defined by 

\begin{align*}
	\mathbf{p} = (x,y) = (\frac{\mathbf{p}_{1_x} + \mathbf{p}_{2_x}}{2}, \frac{\mathbf{p}_{1_y} + \mathbf{p}_{2_y}}{2}),  
\end{align*}

is chosen as an estimate of the position of target $i$. This was found to be robust to changes in pose, providing a smooth estimate of the position. 

The core task of this tracking algorithm is to associate bounding boxes in the current frame with a previous estimate of the state of the targets. Assuming a state estimate $\hat{\boldsymbol{\mu}}_k^i$, where $i = 1, \ldots,  n$, and a measurement $\mathbf{z}_k^j$, where $j = 1, \ldots, m$, there are three possible cases. Either $n = m$, meaning there are the same number of tracked targets as detected targets, $n < m$, meaning there are fewer tracked targets than detected targets, $n > m$, meaning there are more tracked targets than detected targets. In the first case, each state estimate is corrected based on its corresponding measurement. In the second case, a new target has been detected and a Kalman Filter must be initialized to track it. All other targets are treated as in the first case. The final case occurs when a target is no longer detected. In this case, the correction step is simply skipped and the estimate is simply propagated through time until TODO.

In each of these cases, the measurement must be associated with its corresponding state estimate. Assuming a position estimate of $\hat{\mathbf{p}}_k^i$, it is assumed that this corresponds to measurement $j$ if

\begin{align*}
	|| \hat{\mathbf{p}}_k^i - \mathbf{z}_k^j || < d_{min},
\end{align*}

where $d_{min}$ is a tunable parameter. In other words, if the norm of the vector from a state estimate to a measurement is less than a threshold distance, they are associated. Following this procedure, along with the described method to deal with new targets and targets that no longer detected, the output of the Kalman Filter described below should be a consistent state estimate of each target from the moment they enter the tracker's field of view until they leave it, thus yielding an estimate of the trajectory of each target in the ground plane. 

### Kalman Filter Implementation

When a new target is detected, a new Kalman Filter is initialized to track it. The state is initialized to $\hat{\boldsymbol{\mu}}_k^i = [\mathbf{z}_k^i \ \mathbf{0}]^\textsf{T}$. The covariance matrix $\boldsymbol{\Sigma}_k^i$ is initialized to TODO. At a constant rate of 20Hz, the kinematics are integrated and covariance is propagated. The equations governing these steps are 

\begin{align*}
	\hat{\boldsymbol{\mu}}_k^i = \mathbf{F}_{k-1}\boldsymbol{\mu}_{k-1}^i
\end{align*}

and 

\begin{align*}
	\boldsymbol{\Sigma}_k^i = \mathbf{F}_{k-1} \boldsymbol{\Sigma}_{k-1}^i \mathbf{F}_{k-1}^\textsf{T} +  \mathbf{L}_{k-1} \mathbf{Q} \mathbf{L}_{k-1}^\textsf{T}.
\end{align*}

When a measurement is available, the state estimate is corrected. The Kalman gain is 

\begin{align*}
	\mathbf{K}_k = \boldsymbol{\Sigma}_k^i  \mathbf{H}_{k}^\textsf{T} (\mathbf{H}_{k}\boldsymbol{\Sigma}_k^i\mathbf{H}_{k}^\textsf{T} + \mathbf{R})^{-1}.
\end{align*}

The state estimate and covariance update equations are

\begin{align*}
	\hat{\boldsymbol{\mu}}_k^i = \hat{\boldsymbol{\mu}}_k^i +  \mathbf{K}_k(\mathbf{z}_k - \hat{\mathbf{p}}_k^i)
\end{align*}

and

\begin{align*}
	\boldsymbol{\Sigma}_k^i = \boldsymbol{\Sigma}_{k}^i -   \mathbf{K}_k \mathbf{H}_k \boldsymbol{\Sigma}_{k}^i .
\end{align*}


## Prediction
