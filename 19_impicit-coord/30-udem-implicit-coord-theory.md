# Coordination with Implicit Communication {#implicit-coord-final-report-udem status=beta}

Implicit coordination of traffic at an intersection comprises the orchestration without any form of explicit communication of the entities involved, such as traffic lights or signs, in-vehicle signalization or vehicle-to-vehicle (to-infrastructure) communication systems. Thus,  the outcome of such mechanism is to produce an accurate inference of when it is safe to progress with a crossing maneuver.

As of today, Duckietown exhibits a less complex environment -compared to real-life situations- where the only mobile entities are Duckiebots.  This simplification provides a favorable scenario to explore techniques at different levels of complexity which could be incrementally built to produce algorithms and heuristics applicable to more convoluted scenarios.

Predicting traffic behavior at an intersection depends on accurately detect and track the position of each object as the preamble of applying prior information (traffic rules) for predicting the sequence of expected actions of each element. Hence, the conception of a mechanism that implicitly coordinates the individual behavior of a Duckiebot under such circumstances comprises the research, design, and implementation of components capable of producing the required data for this outcome.

## Detection

Object localization and detection tasks haven been extensively reviewed in the area of Computer Vision for more than 30 years [REF]. With the creation of large corpora of semantically annotated images and a surge of devices' computing power, machine learning techniques have achieved notable results in most visual perception tasks in detriment of purely computer vision-based methods. Most, if not all, state-of-the-art results on these large datasets (ImageNet, PASCAL VOC, MSCOCO, KITTI, and others) have been obtained with the implementation of neural network architectures (Convolutional Neural Networks) trained under a supervised learning regime.

Therefore,  the examination of deep learning-based object detectors deems appropriate to obtain highly reliable information of the entities present in the field of view of a Duckiebot. A state of the art CNN-based object detector is regularly composed two elements: a CNN for pre-trained on ImageNet object classification challenge (usually named "backbone") and a meta-architecture that transforms the features extracted from the CNN into proposed object regions (bounding boxes).

<!--
![Object Detection](dl_object_detection.png)
-->
<style>
  * {
    padding: 0;
    margin: 0;
  }
  .fit {
    max-width: 100%;
  }
  .center {
    display: block;
    margin: auto;
  }
  .group-photo {
      max-width: 80%;
  }
</style>

<div figure-id="fig:implicit-coord-object-detection-udem">
   <img src="dl_object_detection.png" class='center fit'/>
   <figcaption>Implicit coordination object detection</figcaption>
</div>

### Object Detection in Duckietown

A recent paper produced by a group at [Google Research](https://arxiv.org/pdf/1611.10012.pdf) reproduces an apples-to-apples comparison of the speed/memory/accuracy trade-off among the object detection algorithms that have currently obtained the best result in different visual challenges. Figure below  depicts the speed vs. accuracy the result of their research, and represents the basis for a selection of an object detection algorithm in the context of the implicit coordination mechanism.

<!--
![Speed vs Accuracy Trade-Off on Object Detectors](tradeoff.png)
-->

<div figure-id="fig:implicit-coord-speed-vs-accuracy-obj-det-udem">
   <img src="tradeoff.png" class='center fit'/>
   <figcaption>Speed vs Accuracy Trade-Off on Object Detectors</figcaption>
</div>

As explained before, Duckietown presents a diminished environment when contrasted with real-life scenarios while the only entities that constitute the traffic at its intersections are Duckiebots. Consequently, the object detection task is reduced to the retrieval of the bounding box of Duckiebots.  Also, the current Duckiebot platform based on a Raspberry Pi 3 board poses a challenging constraint regarding the computational resources available. This low performant environment creates a requirement for the careful selection of a model that balances speed, memory consumption, and accuracy.  

The analysis of the trade-off presented on this comparison leads to a combination of Residual Networks + Faster-R-CNN as the most balanced solution. However, it must be taken into account that the GPU Time reported was obtained on a machine with 32GB RAM, an Intel Xeon E5-1650 v2 processor, and a Nvidia GeForce GTX Titan X discrete GPU. This specification is several times superior to a Raspberry Pi 3 capabilities, even with the presence of external processing devices (e.g.: Movidius Neural Compute Stick).

Worth noting that while conducting similar research on the constraints imposed by a Duckiebot design would be relevant,  an architecture based on  [MobileNets](https://arxiv.org/pdf/1704.04861.pdf) and [Single-Shot Detector (SSD)](https://arxiv.org/pdf/1512.02325.pdf) seemed the most appropriate -best accuracy among the fastest solutions. Also, as these models are constructed to deal with hundreds of object classes and to have our problem reduced to only Duckiebots, accuracy it is not as significant as it would be in other applications.

### Duckiebot Detection Dataset

All the aforementioned deep learning models obtain their results while training in a supervised learning regime. Thus, as part of the effort of bringing deep learning-based Duckiebot detection to the Duckietown infrastructure, there exists today a dataset composed of approximately 6000 images collected at University of Montreal duckietown instance.  This non-rectified images (640x480 pixels) collected from `~/image_node/camera/compressed/` topic mainly depict the scenario of a Duckiebot parked at a 4-way intersection and capture different traffic patterns and behaviors. Also, in an attempt to make this detection procedure invariant to Duckiebots' aspect, the dataset includes Duckiebots in different poses, sizes, distances and portraying Duckiebots wearing a shell with color variations.

Unfortunately, due to time and human resources constraints, there are currently only 612 densely annotated instances with bounding box coordinates for all Duckiebots in the frame.

### Object Detector Training

Another interesting result of the Google Research paper mentioned above is the creation of the [Tensorflow Object Detection API](https://github.com/tensorflow/models/tree/master/research/object_detection), which makes available a configurable Tensorflow-based library to train from scratch or to fine-tune object detection models. Commonly, training deep neural networks from scratch requires an amount of annotated data for which 612 frames is notably insufficient. In this case, the training of the object detector fine-tuned a MobileNets+SSD architecture pre-trained on the Microsoft COCO dataset, thus relying on the notion of transfer learning.

Below, is the configuration of the Tensorflow Object Detection API used during training. Although it reflects conventional hyper-parameters values selection for these tasks, some assumptions made were explicitly tailored for Duckiebots detection. Firstly, as discussed before, only Duckiebots are detected `num_classes: 1`. Also, the dataset images were downsized to a dimension of 300x300 pixels `fixed_shape_resizer { height: 300 width: 300 }`, as a data augmentation technique each image produced a synthetic frame when applied random horizontal flip `data_augmentation_options {    random_horizontal_flip { } }`.  Another significant factor in object detection meta-architectures is the number of maximum detections per class, and the maximum number of detection per image assumed to be no than 15 `max_detections_per_class: 15   max_total_detections: 15`.

```
model {
  ssd {
    num_classes: 1
    box_coder {
      faster_rcnn_box_coder {
        y_scale: 10.0
        x_scale: 10.0
        height_scale: 5.0
        width_scale: 5.0
      }
    }
    matcher {
      argmax_matcher {
        matched_threshold: 0.5
        unmatched_threshold: 0.5
        ignore_thresholds: false
        negatives_lower_than_unmatched: true
        force_match_for_each_row: true
      }
    }
    similarity_calculator {
      iou_similarity {
      }
    }
    anchor_generator {
      ssd_anchor_generator {
        num_layers: 6
        min_scale: 0.2
        max_scale: 0.95
        aspect_ratios: 1.0
        aspect_ratios: 2.0
        aspect_ratios: 0.5
        aspect_ratios: 3.0
        aspect_ratios: 0.3333
      }
    }
    image_resizer {
      fixed_shape_resizer {
        height: 300
        width: 300
      }
    }
    box_predictor {
      convolutional_box_predictor {
        min_depth: 0
        max_depth: 0
        num_layers_before_predictor: 0
        use_dropout: false
        dropout_keep_probability: 0.8
        kernel_size: 1
        box_code_size: 4
        apply_sigmoid_to_scores: false
        conv_hyperparams {
          activation: RELU_6,
          regularizer {
            l2_regularizer {
              weight: 0.00004
            }
          }
          initializer {
            truncated_normal_initializer {
              stddev: 0.03
              mean: 0.0
            }
          }
          batch_norm {
            train: true,
            scale: true,
            center: true,
            decay: 0.9997,
            epsilon: 0.001,
          }
        }
      }
    }
    feature_extractor {
      type: 'ssd_mobilenet_v1'
      min_depth: 16
      depth_multiplier: 1.0
      conv_hyperparams {
        activation: RELU_6,
        regularizer {
          l2_regularizer {
            weight: 0.00004
          }
        }
        initializer {
          truncated_normal_initializer {
            stddev: 0.03
            mean: 0.0
          }
        }
        batch_norm {
          train: true,
          scale: true,
          center: true,
          decay: 0.9997,
          epsilon: 0.001,
        }
      }
    }
    loss {
      classification_loss {
        weighted_sigmoid {
          anchorwise_output: true
        }
      }
      localization_loss {
        weighted_smooth_l1 {
          anchorwise_output: true
        }
      }
      hard_example_miner {
        num_hard_examples: 3000
        iou_threshold: 0.99
        loss_type: CLASSIFICATION
        max_negatives_per_positive: 3
        min_negatives_per_image: 0
      }
      classification_weight: 1.0
      localization_weight: 1.0
    }
    normalize_loss_by_num_matches: true
    post_processing {
      batch_non_max_suppression {
        score_threshold: 1e-8
        iou_threshold: 0.6
        max_detections_per_class: 15
        max_total_detections: 15
      }
      score_converter: SIGMOID
    }
  }
}

train_config: {
  batch_size: 32
  optimizer {
    rms_prop_optimizer: {
      learning_rate: {
        exponential_decay_learning_rate {
          initial_learning_rate: 0.004
          decay_steps: 1500
          decay_factor: 0.95
        }
      }
      momentum_optimizer_value: 0.9
      decay: 0.9
      epsilon: 1.0
    }
  }
  fine_tune_checkpoint: "object_detection/duckietown_data/ssd_mobilenet_v1_coco_2017_11_17/model.ckpt"
  from_detection_checkpoint: true
  # Note: The below line limits the training process to 200K steps, which we
  # empirically found to be sufficient enough to train the pets dataset. This
  # effectively bypasses the learning rate schedule (the learning rate will
  # never decay). Remove the below line to train indefinitely.
  num_steps: 10000
  data_augmentation_options {
    random_horizontal_flip {
    }
  }
  data_augmentation_options {
    ssd_random_crop {
    }
  }
}

train_input_reader: {
  tf_record_input_reader {
    input_path: "object_detection/duckietown_data/duckietown.tfrecord"
  }
  label_map_path: "object_detection/duckietown_data/duckietown_label_map.pbtxt"
}

eval_config: {
  num_examples: 612
  # Note: The below line limits the evaluation process to 10 evaluations.
  # Remove the below line to evaluate indefinitely.
  max_evals: 10
}

eval_input_reader: {
  tf_record_input_reader {
    input_path: "object_detection/duckietown_data/duckietown.tfrecord"
  }
  label_map_path: "object_detection/duckietown_data/duckietown_label_map.pbtxt"
  shuffle: true
  num_readers: 1
  num_epochs: 1
}

```


## Tracking

The detection algorithm provides the coordinates of the vertices of bounding box in the camera frame, along with a measure of confidence in the detection. In the context of coordination, this information is only meaningful if the bounding boxes can be tracked from frame to frame, ultimately allowing the Duckiebot to predict the movement of any Duckiebot within its field of view. In the context of this document, the Duckiebot performing the tracking operation is called the tracker, whereas the Duckiebots being tracked are referred to as the targets. It is assumed in this project that tracker is stationary at an intersection.

The tracking algorithm requires solving two main problems. First, a data association problem must be solved. This is required to match the bouding boxes in the previous frame with the bounding boxes in the current frame. Second, a state estimation problem must be solved to smooth out the measurements and increase the robustness of the detection algorithm. A Kalman Filter is used to do this. The Kalman Filter will use the kinematics of each target Duckiebot to propagate the state estimate through time. Then, using the measurements from the detection algorithm, the estimate will be corrected.

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

### Measurement Model

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


Question: Incomplete?
