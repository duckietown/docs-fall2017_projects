# Demo Instructions

As part of our work in the Implicit Coordination project, we have added three new nodes for the Duckietown infrastructure and their relationship is shown in the figure below.

![](nodes.png)

## Object Detection Node

This node source code can be find on the `devel_implicit_coord` branch of `duckietown/Software` repo in the `catkin` workspace at `80-deep-learning/object_detection`

### Prerequisites

For this node, the following libraries have now been add to the Duckietown infrastructure as a dependencies:

- Tensorflow 1.4.0 (GPU/CPU, installed from pip)

### Configuration

The root directoy of the node contains a `default.yaml` configuration file.

```
object_detector:
  inference_graph_path: models/mobilenets_ssd.pb
  score_threshold: 0.2
  denormalize_boundingbox: True

```

### Messages
