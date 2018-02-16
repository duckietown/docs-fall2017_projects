#  Supervised Learning: final report {#supervised-learning-final-report status=draft}

This is the final project report for the group of Supervised Learning at ETH Zurich 2017 Fall Semester. The project motivation, implementation and results are shown here. For inquries about Convolutional Neural Network training, please contact Shaohui Yang (), for inquiries about ROS implmemtnation of the project, please contact Tianlu Wang (tiawang@student.ethz.ch).

## The final result {#supervised-learning-final-result}

The final results are shown in the attached video. See the following links. [Recorded video](https://youtu.be/FCP8Ndoxae0) 

## Mission and Scope {#supervised-learning-final-scope}

_Mission:_

To learn policies which match the results from recorded data from agents in the real world, so that the vast volumes of the data in the real world can be made useful.  

_Scope:_

- Verifying whether Deep Learning can be used successfully in Duckietown;

- Motivated by the concept of ‘data processing inequality’, using supervised and imitation learning to control the duckiebot end-to-end with data from a recorded policy;

- Using supervised or unsupervised learning to model specific aspects of the autonomous driving task;
 
- Focus on autonomous lane follwing by learning based tools. 

### Motivation {#supervised-learning-final-result-motivation}

According to the definition of 'data process inequality', essential information is prone to be left out along a long process chain, like the conventional approach for autonomous lane follwiing. To cope with this problem, an end-to-end network work is expected to be implemented, which maps raw input images from camera to vehecles' control command directly.

### Existing solution {#supervised-learning-final-literature}

The similar end-to-end imitation learning neural network has already been implemented by Nvidia for the task of lane following on real roads. The demo details can be seen from the following link. [Nvidia demo](https://youtu.be/-96BEoXJMs0)

In the case of Nvidia's work, researchers did not program any explicit object detection, mapping, path planning or control component into this car. Instead, the vehicle learns on its own to create all necessaty internal representations necessary to steer, simply by observing human drivers. The success of learning to drive in complex environments demomstrates new capacities of deep neural network.

### Opportunity {#template-final-opportunity}

Though the aim is quite same between Nvidia's work and our project, the specific requirements are different. Moreover, our group is the first one to start projects on supervised learning's application on Duckiebots on Zurich's branch. Therefore, the opportunities can be summarized into following aspects: [contribution]

- There wasn't previous implementation of supevised learning's application on lane following for Duckiebots;

- The performance of current approach for lane following is not optimal due to the computation limit of Raspberry Pi;

- For Nvidia's implementation, the network's input is the raw images, and the control command is steering angles, gas and brake, which is different from our case in Duckietown, where the control output is only the Bot's orientation;

Specifically, by implementing the network for lane following, we hope to improve the performance of the conventional approach. This can be assured by two aspects: 

- End-to-end network can have better performance due to 'data process inequality';
- With an extra on-board coputation device Neural Compute Stick, the computation power of Duckiebots can be futher increased.

### Preliminaries (optional) {#template-final-preliminaries}

- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.

Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)

## Definition of the problem {#template-final-problem-def}

_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical definition of the problem you tackled. You can use part of the preliminary design document to fill this section._

Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal

## Contribution / Added functionality {#template-final-contribution}

Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_

## Formal performance evaluation / Results {#template-final-formal}

_Be rigorous!_

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#template-final-next-steps}

_Is there something you think still needs to be done or could be improved? List it here, and be specific!_
