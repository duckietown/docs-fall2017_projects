#  Supervised Learning: final report {#supervised-learning-final-report status=draft}

This is the final project report for the group of Supervised Learning at ETH Zurich 2017 Fall Semester. The project motivation, implementation and results are shown here. For inquries about Convolutional Neural Network training, please contact Shaohui Yang (), for inquiries about ROS implmemtnation of the project, please contact Tianlu Wang (tiawang@student.ethz.ch).

## The final result {#supervised-learning-final-result}

The final results are shown in the attached video. See the following link. [Recorded video](https://youtu.be/FCP8Ndoxae0) 

## Mission and Scope {#supervised-learning-final-scope}

_Mission:_

To learn policies which match the results from recorded data from agents in the real world, so that the vast volumes of the data in the real world can be made useful.

_Scope:_

- Verifying whether Deep Learning can be used successfully in Duckietown;

- Motivated by the concept of ‘data processing inequality’, using supervised and imitation learning to control the duckiebot end-to-end with data from a recorded policy;

- Using supervised or unsupervised learning to model specific aspects of the autonomous driving task;
 
- Focus on autonomous lane follwing by learning based tools. 

### Motivation {#supervised-learning-final-result-motivation}

According to the definition of 'data process inequality', essential information is prone to be left out along a long process chain, like the conventional approach for autonomous lane following. To cope with this problem, an end-to-end network work is expected to be implemented, which maps raw input images from camera to vehecles' control command directly.

### Existing solution {#supervised-learning-final-literature}

The similar end-to-end imitation learning neural network has already been implemented by Nvidia for the task of lane following on real roads. The demo details can be seen from the following link. [Nvidia demo](https://youtu.be/-96BEoXJMs0)

In the case of Nvidia's work, researchers did not program any explicit object detection, mapping, path planning or control component into this car. Instead, the vehicle learns on its own to create all necessaty internal representations necessary to steer, simply by observing human drivers. The success of learning to drive in complex environments demomstrates new capacities of deep neural network.

### Opportunity {#supervised-learning-final-opportunity}

Though the aim is quite same between Nvidia's work and our project, the specific requirements are different. Moreover, our group is the first one to start projects on supervised learning's application on Duckiebots on Zurich's branch. Therefore, the opportunities can be summarized into following aspects:

- There wasn't previous implementation of supevised learning's application on lane following for Duckiebots;

- The performance of current approach for lane following is not optimal due to the computation limit of Raspberry Pi;

- For Nvidia's implementation, the network's input is the raw images, and the control command is steering angles, gas and brake, which is different from our case in Duckietown, where the control output is only the Bot's orientation; CNN is expected to be used in our case to realize the end-to-end control;

Specifically, by implementing the network for lane following, we hope to improve the performance of the conventional approach. This can be assured by two aspects: 

- End-to-end network can have better performance due to 'data process inequality';

- With an extra on-board coputation device Neural Compute Stick, the computation power of Duckiebots can be futher increased.

### Preliminaries (optional) {#supervised-learning-final-preliminaries}

There are three parts of preliminaries that are important to the implementation:

- Understand basic knowledge and differences between machine learning, deep learning, supervised learning and unsupervised learning;

- Train an effective Convotional Neural Network which can maps raw image to orientation of Duckiebots for lane following;

- Implement a ROS node which subscribes to input images, communicates with Neural Compute Stick for computation, and publishes the computed orientation angle to the car control node. 

Concerning learning related knowledge, the relation between machine learning and deep learning is shown in the following figure. Moreover, machine learning can be catagorized into three groups: supervised learning, unsupervised learning and reinforcement learning.

![Plug 0](machine-deep.png)

To know more about Machine Learning, readers can refer to [ETH Machine Learning Course](https://ml2.inf.ethz.ch/courses/ml/) and [Andrew's Course on Deep Learning](https://www.deeplearning.ai/), to be familiar with CNN, readers can refer to [Stanford University CS231n](http://cs231n.stanford.edu/) for further information; get familiar with Neural Compute Stick, please refer to [Movidius NCS Information](https://developer.movidius.com/); to know how to implement ROS in our project, please refer to our code directly.

## Definition of the problem {#supervised-learning-final-problem-def}

_Final objective:_

We have recorded data of the lane following algorithm exploring the duckietown. Our goal is to learn a policy which performs as well as, or better than the policy which produced the data.

_Assumptions:_

- The policy used to collect the data is reasonably good;

- The errors in imitation learning are sufficiently small to allow a straightforward approach to learn a decent policy;

- Our trained policy can improve the robustness of overall performance. 

_Evaluation:_

- Publishing rate of the computed orientation by the neural network, compared with the conventional approach;

- Observation of the overall lane following performance, compared with the conventional approach.

## Contribution / Added functionality {#supervised-learning-final-contribution}

As mentioned above, our group initiated the learning based approaches for Duckietown. Contributions can be categorized into two groups, successful training of a CNN for lane following and  its relevant ROS implementation. The details are demonstrated below.

_Logical Architecture:_

The logical architecture can be seen in the following picture. We will develop one node, the trained deep imitation learning model, that maps the compressed images to control command(orientation). All other nodes will remain unchanged.

![Plug 2](intermediate_plug1.png)

_Software Architecture:_

There are three main steps for software part:

- Offline training with logged data;

- NCS thing works on the laptop; 

- It’s funny on the Pi.

_Model Training:_

_ROS Implementation:_

When implementing the ROS node, the different speed of subscription to images and computation speed of NCS should be paid attention to. To make sure that each of the input image can be processed properly, we start a daemon thread to process them. For details, please refer to our code. More proper approaches can also be exploited.

## Formal performance evaluation / Results {#supervised-learning-final-formal}

_Results:_

The overall results of the project can be seen from the demo video: [Recorded video](https://youtu.be/FCP8Ndoxae0). Because we are the first group starting work on supervised learning for Duckietown, it is not possible to compare our results with former groups on the same topic. Threfore, we compare the performance our implemented lane following based on 

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#supervised-learning-final-next-steps}

_Is there something you think still needs to be done or could be improved? List it here, and be specific!_
