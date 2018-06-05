# Imitation Learning {#part:imitation-learning}

# Imitation Learning {#imitation_learning status=beta}

This is the description of imitation learning project demo.

<div class='requirements' markdown="1">

Requires: Wheels calibration completed.[wheel calibration](#wheel-calibration)

Requires: Camera calibration completed.[Camera calibration](#camera-calib)

Requires: Joystick demo has been successfully launched.[Joystick demo](#rc-control)

Requires: Movidius Neural Compute Stick.

Requires: PyTorch installed on the local computer, Caffe installed with python2.7 bindings (NCSDK installs python3 bindinds for caffe)
</div>

## Installation Instructions

### 1. NCSDK

* Follow [these](https://developer.movidius.com/start) instructions to install NCSDK on the computer. (This will automatically install caffe as well)

* Follow [these](https://movidius.github.io/blog/ncs-apps-on-rpi/) instructions to install NCSDK API on the Raspberry Pi 3. (This installation should be fast, since it only installs the api)

### 2. PyTorch

    laptop $ conda install -c soumith pytorch==0.2.0 torchvision

### 3. Caffe

Follow [these](http://caffe.berkeleyvision.org/install_apt.html) instructions.

### Training and Compiling instructions

For these instructions, clone [this](https://github.com/ritheshkumar95/imitation_learning/tree/rithesh_branch) repository.

1. Create a hdf5 dataset from the rosbag containing logs of lane following using the `create_dataset_from_bag.py` script

2. Train the PyTorch model using the command:

```shell
    laptop $ python pytorch_train.py
```

3. Convert the PyTorch model into caffe using the command:

```shell
    laptop $ python convert_to_caffe.py
```    

   NOTE: This would require PyTorch 0.2.0 and Caffe to be installed in the python2.7 workspace
   This step creates 2 files, `deploy.prototxt` and `deploy.caffemodel` which are the network and weights files respectively.

4. Compile the Caffe model into NCS format using the command:

```shell
    laptop $ mvNCCompile -w deploy.prototxt -s 12 deploy.caffemodel
```
   This creates a binary file called "graph" which would be used by the inference code.


## Demo instructions

These are the step by step instructions to reproduce the demo.
Clone [this branch](https://github.com/duckietown/Software/tree/ritheshkumar95-project/) of the Duckietown repository.

NOTE: Make sure the robot names are modified accordingly in `src/imitation_learning_node.py`, currently robot name and the NCS graph object locations are hard-coded.

Step 1: On duckiebot, in `/DUCKIERTOWN_ROOT/` directory, run command:

    duckiebot $ source environment.sh
    duckiebot $ source set_vehicle_name.sh duckduckgo
    duckiebot $ roslaunch imitation_learning lane_following.launch

* Wait a while so that everything has been launched.
* Press R1 to start autonomous lane following using the imitation learning algorithm.
* Press L1 to switch to joystick control.

NOTE: Ensure wheel calibration is correct. Parameter tuning is a must. The only two parameters that can be modify are the gain and trim. The parameter pair which makes your bot go straight will unlikely work for the lane following due to the current controller design. Facts show that a gain ranging from 0.5 to 0.9, as long as paired with a suitable trim, will all work on this demo. Start with your parameter pair obtained from wheel calibration. Increase gain for higher speed. Increase trim to horizontally move the bot to the center of the lane. Decrease will do the inverse.

## Troubleshooting

Contact Rithesh Kumar (UdeM) via Slack if any trouble occurs.
