#  Supervised learning: intermediate report {#super-learning-int-report status=beta}

TODO: JT: fix image sizes, bullet points hierarchy

## Part 1: System interfaces

The system architecture is shown below.
![Plug 1](intermediate_plug1.png)

### Logical architecture

- Please describe in detail what the desired functionality will be. What will happen when we click "start"?

* The desired functionality is the deep imitation learning network. How the function works is the following: the node of trained deep imitation learning network subscribes to the image published by compressed image node, does computation,  then publishes control commands(orientation and velocity) to the inverse kinematic node.

- Please describe for each quantity, what are reasonable target values. (The system architect will verify that these need to be coherent with others.)

* The robot does lane following with a learned end-to-end deep imitation learning system. Look at the activations of the layers and try to understand them. The target can also be extended to Indefinite navigation realized by deep imitation learning if the lane following is fulfilled.

- Please describe any assumption you might have about the other modules, that must be verified for you to provide the functionality above.

* The time latency of other modules are within reasonable range.

<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

- Please describe the list of nodes that you are developing or modifying.

* We will develop one node, the trained deep imitation learning model, that maps the compressed images to control commands(orientation and velocity). All other nodes will remain unchanged.

- For each node, list the published and subscribed topics.

* The deep imitation learning node subscribes to /VEHICLE_NAME/camera_node/image/compressed.

* The node publishes /VEHICLE_NAME/car_cmd_switch_node/cmd

- For each subscribed topic, describe the assumption about the latency introduced by the previous modules.

* The latency of image topic can be measured. But during the model training process, we would like to use the map between multiple images and one control command to cover the latency.

- For each published topic, describe the maximum latency that you will introduce.

* The latency that our node introduces will be settled by the computation capability of the Intel Neural Compute Stick and the scale our model. Tests are required before the latency can be finalized.


<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan


### Demo plan

The demo is a short activity that is used to show the desired functionality, and in particular the difference between how it worked before (or not worked) and how it works now after you have done your development.

- How do you envision the demo?

* In the final demo, we hope to implement end-to-end deep imitation learning on Duckiebot and make it work for the aim of lane following.

- What hardware components do you need?

* The Intel Movidius Neural Compute Stick.
![Plug 2](intermediate_plug2.png)


### Plan for formal performance evaluation

- How do you envision the performance evaluation? Is it experiments? Log analysis?

* The performance can be first evaluated manually:

    1)Compare the time of lane following by traditional approach and the end-to-end deep imitation learning approach.

    2)Observe the deviation of lane following by deep imitation learning approach.

* Other formal evaluation approach will be updated.


<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis


### Collection

- How much data do you need?

* A few thousands of image taken by the camera and the corresponding label pairs from images to control commands (orientation, velocity).

- How are the logs to be taken? (Manually, autonomously, etc.)

* The logs will be taken manually. The make log-minimal in branch ‘dev-eth-sup-learning’ will be enough for the data collection.

- Describe any other special arrangements.

* None.

- Do you need extra help in collecting the data from the other teams?

* None.

### Annotation

- Do you need to annotate the data?

* None, The data itself makes enough sense.

- At this point, you should have you tried using [thehive.ai](https://thehive.ai/) to do it. Did you?

* We have collected the data and extracted the things we need already.


### Analysis

- Do you need to write some software to analyze the annotations?

* None. But we did write a python script to write images and data pair from ros bag.

- Are you planning for it?

* None.

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
