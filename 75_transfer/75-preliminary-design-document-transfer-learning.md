#  Transfer learning: preliminary report {#transfer-pdd status=beta}

## Part 1: Mission and scope

### Mission statement

The goal is to test transfer learning algorithms trained on simulator and test on the real duckietown.

Solving control problems in reality is hard due to sparse reward signals, expensive data acquisition and the danger of breaking the robot during exploration. It is comparatively easier to train the policy in a simulator as we can “speed up” the reality, and there are no inherent dangers of running arbitrary policies. But policies trained on simulator do not necessarily transfer directly onto the real world, and our goal is try and bridge this gap

### Motto

Motto: IPSA SCIENTIA POTESTAS EST, <br/>UT TRANSIRE CALLIDUS EST <br/>(Knowledge is power, transferring that is clever)

### Project scope

Final goal: Implement [DARLA](https://arxiv.org/abs/1707.08475) or [https://arxiv.org/pdf/1703.06907.pdf](https://arxiv.org/pdf/1703.06907.pdf) or [https://arxiv.org/pdf/1710.06537.pdf](https://arxiv.org/pdf/1710.06537.pdf)

#### What is in scope

Testing out different RL algorithms, (or unsupervised feature learning algorithms, if any)

Methods for transferring from simulator to duckietown

Baselining (?)

#### What is out of scope

Hardware modifications

Design modifications that might be needed

Anything not involving the control of the duckiebot

#### Stakeholders

Simulator (Maxime and Florian)

Supervised Learning (Rithesh and Alex)

Running NN models on the bot (Neural stick ?) who is in charge?

File with all projects [click](https://docs.google.com/presentation/d/1AAtXIOBTlAJajn7d2Zrbv9CEAnz8YidB-FHQ3SO3Sec/edit)

## Part 2: Definition of the problem

### Problem statement

We will replace the part of the pipeline that uses raw images and give motor controls by using a model trained in a simulator. The model policy will be able to do lane following and navigation. The model can hopefully generalize to imperfect camera calibration, motor calibration and different light conditions.

Mission: Efficient transfer of algos trained on simulators


Problem statement:

- Implement [DARLA](https://arxiv.org/abs/1707.08475) or
- [Domain Randomization Paper](https://arxiv.org/pdf/1703.06907.pdf) or
- [Dynamics Randomization Paper](https://arxiv.org/pdf/1710.06537.pdf )
- Apply the algorithms for tasks like, navigation.


### Assumptions

We need the simulation to be close enough to real world so as to be transferable.

We need to run inference on the robot at test time.

Maybe allow for fine-tuning in the real world (on a small dataset)

### Approach

- Get the simulator up and running
- Start with the most basic lane following environment: A straight lane…
- Train a model to control the duckiebot in the simulator
- Transfer the policy to the real world
- Move onto slightly more complicated scenario … repeat (curriculum learning setting)

### Functionality-resources trade-offs

### Functionality provided

Lane following or route following (i.e. follow a given route through duckietown and obey traffic laws)

Metrics:

- Quality of navigation (as defined by a reward/loss function)
- Quality of transfer defined by the above reward function, computed automatically or by hand
- Finetuning needed for transfer?

Reward function description (probable):

- Deviation from center (-ve)
- Time taken (-ve)
- Finish position (+ve)
- Collision (-ve)
- Violating traffic rules or conventions (-ve)
- See [Socially Aware Motion planning](https://arxiv.org/abs/1703.08862)

### Resources required / dependencies / costs

- \# of cpu/gpu-hours for training
- Test time computation costs
- Duckiebots/ duckietown

### Performance measurement

- In simulation, use access to the true state to compute the reward function
- In duckietown, compute the reward function by hand (or develop a heuristic for computing it)
- Qualitative comparison to current control pipeline
- Baseline wrt other RL algos
- Compare with #devel-super-learning for performance wrt imitation learning policies


## Part 3: Preliminary design

### Modules

Module 1: Policy mapping raw images ⟶ actions
(or Module 1a: policy ⟶ disentangled representation ⟶ actions)

Module 2: Actions ⟶ motor voltages (Joy Mapper node)

Module 3: Training module  (Also introduces domain/dynamics randomization in the simulator)


### Interfaces

Module 1: subscribe to raw camera images and publish actions (forward/backward/turn left/turn right = float values)

Module 2: already provided in duckietown stack

Module 3:

- Input: simulator parameters, model
- Output: a trained policy to be used in Module 1

### Preliminary plan of deliverables

### Specifications

See Modules and Interfaces above.

### Software modules

During training, the modules will be written in Python.

At test time, modules 1,2 will be deployed as a ROS node during test.

### Infrastructure modules

Simulator

## Part 4: Project planning

### Data collection

- Real world cam pictures corresponding to simulator states (e.g in front of a straight lane/at the beginning of a left/right turn/...) : can be useful if we want to assess the quality of the disentangled representation without the need to run it on the robot.
- Use the simulator to generate training data for RL algorithms

### Data annotation

The simulator will annotate data automatically by providing ground truth information about the duckiebot and the environment

Comment: To be confirmed… We probably need to implement a module that does that.

If needed, then semantically segmented images would useful

#### Relevant Duckietown resources to investigate

Simulator

#### Other relevant resources to investigate

PyTorch rl codebase, OpenAI Gym, OpenAI Baselines

DARLA: https://arxiv.org/abs/1707.08475

Transfer by randomization/generalization:

- https://arxiv.org/abs/1703.06907
- https://arxiv.org/abs/1710.06537

UNREAL: https://arxiv.org/abs/1611.05397

Socially Aware Motion planning: https://arxiv.org/abs/1703.08862

### Risk analysis

Deploying the policy trained in simulator will break the bots in real duckietown.

Risk mitigation:

- Train it properly
- Curriculum learning setting to gradually increase difficulty
- Safety on/off switch
- Cushion the sides of the lane
