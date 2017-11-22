#  Visual Odometry: preliminary document {#visual-odometry-preliminary-design-doc status=ready}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

We will use unsupervised learning to build a depth estimation system for Duckietown.  
The application could be building a point cloud map for Duckietown for mapping--our overall plan is to have a serviceable deep network that is trained end-to-end with no ground truth data.  We will also be using the Movidius chip, hopefully learning to how integrate it well into the current system for future users.

### Motto

<div class='check' markdown="1">
Carpe diem.
</div>

### Project scope

The scope of the project is to use recent work in unsupervised depth estimation as well as training data we gather in Duckietown to create a fully unsupervised depth estimation system.

#### What is in scope

The deep network and use of Movidius chip.

#### What is out of scope

Localization.  We plan on utilizing the april tags for localization. Also hardware modification--we plan to do fully monocular depth.

#### Stakeholders

All of the SLAM teams could (potentially) benefit from our system, though it might be non-trivial to integrate it.

## Part 2: Definition of the problem

### Problem statement
We need to train a deep neural network to predict depth from monocular video (with no GT depth!)

### Assumptions

We assume that it's possible to collect diverse enough training data in Duckietown to train a deep CNN (outside data like KITTI might not transfer to this task).

### Approach

We will start with the "Unsupervised Learning of Depth and Ego-Motion from Video" paper and modify the framework for duckietown.

### Functionality-resources trade-offs

There's often a tradeoff beteen size of network and performance--the Movidius chip is more limited than our usual NVidia GPUs.

### Functionality provided
We will need to develop some confidence measures for our depth map.  Time permitting, we would like to build a point cloud map of Duckietown.

### Resources required / dependencies / costs

We will need the Movidius chip for best performance--our metrics are the size of the network (including activations) and making sure that inference is real-time.

### Performance measurement
Measuring performance here is tricky since there is no way to obtain ground truth depth data from Duckietown (aside from using a range sensor not available for duckiebots). We will likely develop a surrogate metric based on april tags.

## Part 3: Preliminary design

### Modules
Since it's an end-to-end neural network it's all in one logical module, but we could split up the design of the architecture from the way we use the data (e.g. data augmentation)

### Interfaces

Input: RGB Image
Output: Depth map (potentially relative depth, discretized)

### Preliminary plan of deliverables
The architecture and the data collecation and augmentation schemes need to be designed.
Tensorflow implementation of architecture is what needs to be implemented.
There already exists open source code for unsupervised learning of depth paper (linked below).

### Specifications
Do you need to revise the Duckietown specification? N/A

### Software modules
The software will be a simple end-to-end CNN going from frames to a depth map (likely a node publishing a depth map)

## Part 4: Project planning
Next phase is to start collecting data and experimenting with architectures in Tensorflow.

### Data collection
Video of duckiebot traversing duckietown.

### Data annotation
No data annotation necessary.

#### Relevant Duckietown resources to investigate
All of the camera geometry and computer vision notes.

#### Other relevant resources to investigate
1. https://people.eecs.berkeley.edu/~tinghuiz/projects/SfMLearner/
2. https://github.com/tinghuiz/SfMLearner

### Risk analysis

It's possible that training a deep neural network on only duckietown data will be difficult.  We will also consider using the KITTI dataset as additional training data.
