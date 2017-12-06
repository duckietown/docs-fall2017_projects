# PDD Neural Slam {#pdd-neural-slam status=beta}


## Part 1: Mission and scope

### Mission statement

Build a map from a duckie driving around the road autonomously that will be used for planning 


### Motto

> À l'idiot sans mémoire tout lui paraît nouveau et miraculeux



### Project scope

**What is in scope**

Train agent to learn to explore an entire map efficiently and leep a representation of the current knowledge of the map.


**What is out of scope**

Need to run live on the duckie.
We do not use real images but assume we have a tile detector

**Stakeholders**

Transfer learning team - tbd



## Part 2: Definition of the problem

### Problem statement

We want the to keep a representation of the current map in memory that could used for downstream task such as planning


### Assumptions
 * We work on the tile level, and do not worry about low-level control. 
 * We assume there are only two types of tiles, road or not road

### Approach

Stage 0: Create an environment where we can a simulated agent 

Stage 1: Train an agent using a reinforcement learning method paired with an external memory

Stage 2: Deploy on the real robot and see how the method perform

Stage 3: Use a decoder that can recover the knowledge of the map

### Functionality provided

 * A exploration policy
 * A map when the exploration is done

### Resources required / dependencies / costs
 * GPUs to train our policy
 * Chip to run the neural network policy
 * A "tile predictor" to infer the tile type from an image
 * A simulator to train the agent

### Performance measurement

 * Check whether or not the agent actually explores the whole map
 * Compare the decoded map with the ground truth map

### Functionality-resources trade-offs

  * Robust obstacle detection (many filters,...) vs. computational efficiency
  * Maximizing speed (e.g. controlles might want to do that) vs. motion blur

## Part 3: Preliminary design

### Modules
 * **A grid map environment to train the agent.**
 * **A policy network with an external memory**
 * **A decoder to decode the external memory into a map**
 * **A tile detector ??**


### Interfaces
 **Simulator**

 * takes the map size as input and generate a environment with agent output

 **Policy network**

 * take the current position of the robot w.r.t its original position, reads/write to the memory and give a control action.


 **Decoder**

 * takes the internal memory as input and give the actual map as output.


### Preliminary plan of deliverables


### Specifications

No need to revise duckietown specifications

### Software modules

 * Pytorch / Tensorflow
 * Grid world simulator


## Part 4: Project planning


### Timeline

 * Build the environment to train the agent
 * Implement the agent to be trained with an external memory for the exploration task
 * Train the agent
 * Check if the map an actually be decoded


### Data collection
 * No need for data collection


### Data annotation

 * No need data annotation

#### Relevant Duckietown resources to investigate
Duckietown simulator


#### Other relevant resources to investigate

 * *Neural SLAM*: Jingwei Zhang, Lei Tai, Joschka Boedecker, Wolfram Burgard, Ming Liu
 * *Learning to Navigate in Complex Environments*: Piotr Mirowski, Razvan Pascanu, Fabio Viola, Hubert Soyer, Andrew J. Ballard, Andrea Banino, Misha Denil, Ross Goroshin, Laurent Sifre, Koray Kavukcuoglu, Dharshan Kumaran, Raia Hadsell
 * *Hindsight experience replay*:  Marcin Andrychowicz, Filip Wolski, Alex Ray, Jonas Schneider, Rachel Fong, Peter Welinder, Bob McGrew, Josh Tobin, Pieter Abbeel, Wojciech Zaremba
 * *Neural Turing Machines*: Alex Graves, Greg Wayne, Ivo Danihelka
Differentiable neural computer Alex Graves et al (DeepMind) 

### Risk analysis
 * Might have issues to scale
 * RL agent might have stability issues paired with an external memory the whole system could be hard to train

