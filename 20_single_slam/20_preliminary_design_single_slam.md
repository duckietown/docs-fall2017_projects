# Single SLAM Project {status=beta}


## Part 1: Mission and scope

### Mission statement

Enable a bot to map duckietown and know its position in the town


### Project scope


#### What is in scope
* Map visualization
* SLAM super-class
    * EKF SLAM
    * Rao-Blackwellized


#### What is out of scope

* SLAM with lane segments
* April tag detector
* Controlling the bot / finding path to do SLAM with

#### Stakeholders

- April Tags
- Control

## Part 2: Definition of the problem

### Problem statement

Every Duckiebot is different in configuration.


> Mission = we need to make control robust to different configuration

> Problem statement = we need to identify kinematic model to make control robust enough

### Assumptions

* Robot will move only in horizontal plane
* No lateral slipping of robot
* The body fixed longitudinal velocity and angular velocity will be provided as well as the timestamp of each measurement
* April tags don’t move
* Gaussian errors for EKF slam (relaxed for Rao-Blackwellized)

### Functionality provided

* A ROS node that subscribes to image feed and controls
* Publishes estimate of robot pose and uncertainty relative to starting point
* Publishes estimates of april tag poses and associated uncertainties relative to starting point 


### Resources required / dependencies / costs

* Requires camera calibration
* Process model
* April tags
* A town


### Performance measurement

* We will qualitatively evaluate the generated map (Visualization) against town
* For finer tuning we may consider pairwise distance between april tags and compare our estimate to the actual town

## Part 3: Preliminary design


### Modules and interfaces

* State data-structure holding poses of robot/features and associated uncertainties
* `Visualize :: State -> Image of map`
* `ProcessImage :: Image -> Poses` (in the map frame)
    * Get relative feature pose given robot pose
    * Add new feature to state
* `Predict :: velocity -> new robot pose`
* `Update :: list of poses, old distribution -> new distribution`

### Subclasses and specific methods
* Kalman Filter
    * 3 sigma circles around feature/robot mean
* Particle Filter
    * Distribution of robot/features heatmap


### Preliminary plan of deliverables

### Specifications

Duckiebots with different hardware configurations for testing


### Software modules

- Parameter estimation:
    - runs calibration protocol
- Velocity translation: (Node)
    - get velocity as input and translate it to voltage as output


### Infrastructure modules

None

## Part 4: Project planning

What data do you need to collect?

### Data annotation

Performances of the current implementation

#### Relevant Duckietown resources to investigate

- Current State Estimation
- Calibration files


#### Other relevant resources to investigate

[Probabilistic Robotics](https://docs.ufpr.br/~danielsantos/ProbabilisticRobotics.pdf) Chapter 3, 10



### Risk analysis

What could go wrong?

* We may not complete the project in the alloted time
* Uncertainty in map may be so high that it is useless


Mitigation strategy:

- Focus on EKF SLAM early
