# Deep Visual Odometry ROS Package
## devel-visual-odometry

This package contains a ROS node `dt_visual_odometry` that produces monocular depth estimates on duckiebot. It also contains another node `apriltags_ros_center`, which is slightly modified from `apriltags_ros` to publish pixel locations, in order to benchmark the result on April tags. You need to have Tensorflow installed on your local machine.

To launch the node, clone the repository into `catkin` workspace and download the Tensorflow model checkpoint file from Duckietown Dropbox to `checkpoint_dir`. This Tensorflow model is obtained through taking a pretrained model on KITTI and further train it on Duckietown for 200K iterations with smoothness penalty set as 0.1. Use the command:

`roslaunch dt_visual_odometry deepvo.launch ckpt_file:=checkpoint_dir robot_name:=robot_name`

This publishes the depth heatmap the into `robot_name/VO/image/compressed` as well as prints the predicted(from CNN) and actual(generated from April tags node, unit in meters) depth on any detected April tags. The scaling factor is calculated as the average predicted/actual depth for all detected April tags, and published under the topic `robot_name/VO/scale`. To supress April tags detections for a higher refresh rate of depth heatmap, use `apriltags_scaling:=0`.
