# PDD - Supervised-learning {#devel-super-learning status=beta}

## Part 1: Mission and scope


### Mission statement

To learn policies which match the results from recorded data from agents in the real world, 
so that the vast volumes of the data in the real world can be made useful.


### Motto

Motto: In Rete Tuo videbimus lumen <br/> (Anything that is said in Latin sounds important)


### Project scope


#### What is in scope

* Verifying whether Deep Learning can be used successfully in duckietown.

* Motivated by the concept of ‘data processing inequality’, using supervised and imitation learning to control the duckiebot end-to-end with data from a recorded policy.

* Using supervised or unsupervised learning to model specific aspects of the autonomous driving task.

* Focusing on indefinite lane navigation by learning based tools. 


#### What is out of scope

* Doing on-policy RL (i.e. running the robot with our learned policy to collect data).

* Collecting our own datasets by running with either our own policy (by hand).


#### Stakeholders

**All current teams**
* Those who wish to use deep learning in the real world could benefit from our pipeline and experiments in using DL in the duckietown.


**For future teams**
* If a future team does on-policy RL in the duckietown, initializing with our imitation learned policy could be smart.


## Part 2: Definition of the problem


### Problem statement

We have recorded data of the lane following algorithm running smoothly in the duckietown. Our goal is to learn a policy which performs as well as, or better than the policy which produced the data. 


### Assumptions

* The policy used to collect the data is reasonably good. 

* The training data can be updated when other groups formulate policy which have better performance. 

* The errors in imitation learning are sufficiently small to allow a straightforward approach to learn a decent policy. 

* Our trained policy can improve the robustness of overall performance.


### Approach

* Initial approach is to take image and control parameter data from the lane navigation checkoff/log data from all schools.

* Start with indefinite navigation and lane following data.

* First DL approach is to take the last k-frames (probably could use a smarter selection strategy which picks some older frames) before the control parameters are recorded, and train a neural network to predict the control parameters from the state.

* We will manually downsample the image frames to find the smallest resolution where the lane is clearly visible by inspection.

* We will use an absolute error to predict the control parameters, and measure relative error on held-out data, to figure out if we can learn a network which generalizes.

* We will train ALI(adversarially learned inference) and VAE(Variational autoencoder) on the images, for the purposes of intellectual curiosity as well as semi-supervised learning (if overfitting is a serious issue).


### Functionality-resources trade-offs

### Functionality provided

1.  Offline metrics
  * Loss for one step ahead prediction on recorded data, with point predictions for the two control parameters.
  
  * Likelihood under a continuous distribution over the predicted control parameters.
  
  * Likelihood under a fixed discrete distribution over the predicted control parameters.

2.  Online metrics: 
  * Actually run the duckiebot in real world duckietown with our learned policy.
  
  * Visual inspection of trajectories. 


### Resources required / dependencies / costs

* Requires neural compute stick on the duckiebot to run. (already got it)
![Plug 1](preliminary_plug1.png)

* GPUs for training models (available through MILA and IDSC in ETH). 

* Data for training the imitation learning algorithm (ideally use as little as possible). 


### Performance measurement

* We can use out-of-sample evaluation for the offline metrics, with care taken so that the train, validation, and test sets cover non-overlapping groups of students.

* Online evaluation will be qualitative, and will be done in the Udem duckietown.


## Part 3: Preliminary design

### Modules

* Data collection: a raw collection of the images (state) and control signals.

* Data alignment: Create sets of actions approximately aligned with states (as they’re recorded at different frequencies).
  
* Data Example Construction: Create tuples of (state[t], action[t], state[t-1], state[t-2]).
  
* Data Train/Valid/Test Split: split the dataset randomly but with different students going into different datasets.
  
* Model trainer: From collected data trains a model to predict actions from states.
  
* Model actuator on duckietown: Runs the duckiebot using actions from the trained model.


### Interfaces

* Data collection: Takes student actions and returns a list of ROS bag files saved to some mila filesystem.

* Data alignment: Performed in-memory, takes the ros-bags as inputs and returns a list of aligned (state[t], action[t]) pairs.

* Data Example Construction: Also performed in-memory, and produces (action[t],state[t],state[t-1],state[t-k]) k-tuples.

* Data Train/Valid/Test Split: Uses a fixed random seed to split the examples into different groups, which are then saved to different files on mila filesystem.

* Model trainer: Takes data as input and returns a saved model binary for running on the intel compute stick (todo: figure out just how small binaries need to be).

* Model actuator on duckietown: Takes a trained model as input and is a ROS module which listens for camera data, and sends control signals to the motors.


### Preliminary plan of deliverables


### Specifications

* No changes to duckietown specification. 

* Duckuments for using neural compute stick. 


### Software modules

* Python script or small collection of python scripts for data processing.

* Python script using Tensorflow for training model.

* ROS node for running a fixed model on the duckiebot.


### Infrastructure modules

* We don’t think any of the modules are infrastructure.


## Part 4: Project planning


### Data collection

* Collect data generated by other navigation policies. 


### Data annotation

* Generally speaking, no data annotation required. 

* Might need to annotate video to remove crashes, stalls. 


#### Relevant Duckietown resources to investigate

* Taiwan group has done imitation learning with 3-camera setup - we may be able to reuse some of their code or at least learn from their experience.


#### Other relevant resources to investigate

* See the following papers. 

[title](https://arxiv.org/abs/1608.01230)
[title](https://selfdrivingcars.mit.edu/deeptesla/)
[title](https://arxiv.org/pdf/1604.07316v1.pdf)


### Risk analysis

* Raw imitation learning may perform badly in practice due to “exposure bias / exploration” issue.

* Simplest solution might be some sort of data augmentation which moves the lane in the training data to create “correction” examples.

* May be smarter ways to improve generalization.

* We may have a hard time figuring out what metrics to trust for offline evaluation of the learned model.

* Workaround might be to report many different metrics and always make sure that the simplest metrics (like per output relative error) are reasonably small.

* Some metrics are hard to interpret: making it difficult to know when to declare success.

* May require some intermediate online evaluation to figure this out.

