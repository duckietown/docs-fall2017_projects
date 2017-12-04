# Anti-Instagram


## Part 1: System interfaces

### Logical architecture
It is worth mentioning that our main task will be to improve the color correction. But many groups have declared us to be the experts for line detection. So we'll include that in our document as well even though we don't plan to change anything in that line detection algorithm.

![Image](https://github.com/duckietown/duckuments/blob/devel-anti-instagram/docs/atoms_85_fall2017_projects/27_anti_instagram/images/flow2.svg?raw=true)
*In this schematic you can see the flow of topics in relation to the Anti-Instagram algorithm.*

#### Color correction

To ensure to have the best line detection possible the system uses a so called Anti-Instagram algorithm. This algorithm determines a color transformation such that the input for the line detector has no color variation. This is important because based on color the Duckiebot knows whether it is a middle, a side or a stopping line. The algorithm tries to minimize the influence of external illumination variation (scattered sunlight, different colors of light sources, ...).

##### Online Calculation
Our task is to improve the color correction algorithm. Our algortihm should run in parallel to the whole system. So there will be no more "start" button as there was in the older system. Approximately every 10 seconds a new color transformation should be calculated. The interval time still should be determined but it will be definitively in order of 10 seconds. This is because the calculation needs more than 1.5 seconds on the Raspberry Pi. It has to be investigated whether a proper parallelization can be achieved such that no other process is disturbed through this computationally expensive process.
The computation of this transformation is done using image data. The current algorithm is implemented using the whole picture. This includes potential artifacts!


##### Use geometrical information
To remove potential artifacts it is valuable to use geometrical information. For example since we know that the camera is fixed on the Duckiebot (Same orientation, same field of view, ... over time) we could determine the "road area". Considering only the relevant area we will be able to get a more accurate color transformation.

###### Self improving system

An idea to push the accuracy even more is to determine first a color transformation and afterwards detect the lines. By that we know where the relevant colors for the transformation are. Afterwards we do another color transformation based on the relevant colors detected by the line detector. This should improve the color transformation.

##### Comparison Old vs. New
| Old Anti-Instagram  | New Anti-Instagram  |
| :------------: | :------------: |
| Press a button to start a new color correction  | Do color correction automatically. Around every 10 seconds there should happen a new color correction  |
| No use of old transformation parameters  |  Use the old parameters as input for the new transformation |
| No use of geometrical values | Use geometric information, publish this information for other systems to use |

### Software architecture

#### Anti-Instagram node

We are improving the Anti-Instagram algorithm. So the Anti-Instagram node should stay the same as it was.

##### Topics
| Published topics  | Explanation | Latency |
| :------------: | :------------: |  :------------: |
| ~corrected_image  |  This is the image after the color transformation. It should serve for example as an input for the line detector.  | << 1 s  |
|  ~health | This is a parameter which should indicate how successful the transformation was. Based on that parameter other nodes could decide whether to use the new correction or not. An idea could be to generate a new "decision node"/"decision topic" which decides whether this correction is useful or not.   | << 1 s |
|  ~transform |  This published topic outputs the transformation parameters. For a linear transformation (which is the case up to now) is would be a *shift* and a *scale* parameter.  | 1.5 seconds |

*Regarding the latency from the published topics: Since we are publishing everything at once/the health and the corrected image depend on the transform these times are not considerable. The transforming time is so high that all the topics need "special treatment". This means we have to find a way to run that online. E.g. running the algorithm only all 10 seconds.*

| Subscribed topics  | Explanation  |
| :------------: | :------------: |
|  ~uncorrected_image | The input for the Anti-Instagram node is the uncorrected image directly from the camera in original resolution.  |


#### Image Transformer node
This node transforms the raw image with the parameters from the Anti-Instagram node.
##### Topics
| Published topics  | Explanation | Latency |
| :------------: | :------------: |  :------------: |
| ~transformed_image  |  The transformed image with the color correction parameters.  | TBD  |




| Subscribed topics  | Explanation  |
| :------------: | :------------: |
|  ~uncorrected_image | The input for the Anti-Instagram node is the uncorrected image directly from the camera in original resolution.  |
| ~transform | This published topic outputs the transformation parameters. For a linear transformation (which is the case up to now) is would be a *shift* and a *scale* parameter.  |

#### Considerable area node

During our research and investigation for the Anti-Instagram algorithm we came up with an idea to only consider the relevant areas of the picture. This would improve the accuracy of the color transformation. E.g. only the black area of the street, the white, red and yellow line markings on the street are relevant inputs for the color transformation. Knowing the location of these features would definitively improve the color correction algorithm.
But the feasibility of this is very unclear yet. We don't know either whether we really implement that. It is definitively worth mentioning the idea and seeing if other people are interested in it as well!

##### Topics

| Published topics  | Explanation  |
| :------------: | :------------: |
| ~mask_image  |  A pixel by pixel indication whether the information of the image is relevant or not should be the output. With this binary matrix one could mask the camera picture and do for example line detection only on the relevant area. This would improve the computation speed a lot. The Anti-Instagram algorithm would profit for sure from that as well!  |


| Subscribed topics  | Explanation  |
| :------------: | :------------: |
|  ~uncorrected_image | The input for the Anti-Instagram node is the uncorrected image directly from the camera in original resolution.   |
| ~colorSegment | The color segments would be probably useful. |
| ~segment_list | The list of detected segments could be for use as well. |

#### Line detector 2 node
The line detector node publishes all the relevant data after detecting the dashed yellow line, the white side line and the potential red stopping line.
##### Topics
| Published topics  | Explanation | Latency |
| :------------: | :------------: |  :------------: |
| ~edge  |  Returns Image with edges on it. It's the output of the OpenCV Canny() function converted to a ROS message.  | TBD  |
|  ~color_segment |  Color segmentation for each channel (B,G,R). Same size as image.  | TBD |
|  ~segment_list |  List of line segments with lines sorted by color.  | TBD |
|  ~image_with_lines |  Corrected image with the detected lines drawn on the image.   | TBD |


| Subscribed topics  | Explanation  |
| :------------: | :------------: |
| ~image | This is the compressed image to read.  |
| ~transform | This published topic outputs the transformation parameters. For a linear transformation (which is the case up to now) is would be a *shift* and a *scale* parameter.  |
| ~switch | This is a switch that allows to control the activity of this node. If the message is true, the node becomes active. If false, it switches off. The node starts as active.  |


## Part 2: Demo and evaluation plan

We plan two different demos, one for the general demo and another for the demo day in January:

### General Demo

##### 2 Duckiebots doing the lane-following demo
One of the Duckiebots has the improved Anti-Instagram algorithm on it and the other Duckiebot will run on the old version. With different light conditions in different parts of Duckietown we try to show that the improved version performs better than the old version, i.e. we want to show that the Duckiebot with the new Anti-Instagram algorithm will be able to follow the line without being disturbed by the change of the lightning conditions. On a screen near by the demo, we want to show the output of each of the Duckiebots, so that the viewer can see the result of the Anti-Instagram algorithm visually. The output can be live from the Duckiebots or recorded and played back from a bagfile.[Scenario I]

| **+**  | **-**  |
| :------------: | :------------: |
| One can see the improvement of the whole system.  | It is not obvious what the algorithm exactly does.  |
| It is very easy to understand  | One does not understand why the newer system works better.  |

*Important notes:*
- The two Duckiebots only differ by the Anti-Instagram algorithm. Everything else stays the same!
- If we don't have enough space we could run two similar videos in parallel. One of the old-version-Duckiebot and one of the new-version-Duckiebot. [Scenario II]

*Material needed [Scenario I]:*
1. Full version of Duckietown
2. 2 Full working Duckiebots
3. A table to cover a part of the Duckietown to produce shadows
4. Dimmable light sources, in order to create a various lightning settings.
5. Big video screen and computer attached

*Material needed [Scenario II]:*
1. Big video screen and computer attached

It should take a few minutes maximum for setup and running the demo.

### Demo Day in January

##### Interactive Live-Application of Anti-Instagram filtering.
The visitors can filter a random static image from the Duckiebot interactively with pre-defined color filters and see an instant output image of the Anti-Instagram algorithm. This demo should show how accurate and powerful the Anti-Instagram algorithm is.

| **+**  | **-**  |
| :------------: | :------------: |
| It is very clear what the Anti-Instagram algorithm does  | It doesn't really attract people since it is very static |
| It is easy to understand  | It doesn't show whether the system has improved or not.  |

*Material needed:*
1. Touchscreen and computer unit
2. Program with ability to filter an image
3. Camera which films image
4. Second screen (?) with output image

##### Visualization of the kMeans algorithm

We want to show on a screen a visualization of the kMeans algorithm.

| **+**  | **-**  |
| :------------: | :------------: |
| It described the algorithm very clear in a short way. For people with interest is should be very understandable. | It is not very attractive. |
| It shows all the technical details (# of clusters, # of iterations, ...) | It is difficult to know what happens. Only for experts valuable. |

*Material needed:*
1. Screen with computer attached.
2. Visualization of the kMeans algorithm


By turns we want to show on the same screen the video output of the general demo described above. It should take a few minutes maximum for setup and running the demos.


### Plan for formal performance evaluation

In order to be able to evaluate our algorithm, we need a metric which gives us an estimation of the quality of the color transformation. As a metric we chose the distances between the color centers of every of the four possible colors for the lines in the Duckietown after the transformation and the 'true' colors. In the image shown below you can see a vizualisation of the color distances. To obtain the average color centers for these colors, we need annotated images, which give us the location of the different lines on the street.

![Image](https://github.com/duckietown/duckuments/blob/devel-anti-instagram/docs/atoms_85_fall2017_projects/27_anti_instagram/images/distance.svg?raw=true)
*Color distances: In this schematic you can see the clusters of the four colored lines in the RGB space how they appear after the Anti Instagram algorithm. Each cluster is assigned to the appropriate 'true' color. The distances from the cluster centers to their correspondent 'true' colors (shown as arrows) are defined by the Euclidean distance.*


#### Performance Evaluation

We will deliver a routine which evaluates the color transformation of one or more images. This program takes a directory to the input images and calculates the distances from every cluster to the 'true' colors as shown above. If more than one image is passed, the routine calculates the average distance as well as the variance.


<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis



### Collection


We want to run a broad analysis to be able to calculate average distances and the corresponding variance. Thus, we need as many test images as we can collect. For a number, we aim to have from a few hundred up to one thousand test images to run our analysis. To obtain the test images, we wrote a script to extract single images from a bag file. Thus, we are able to collect a proper amount of data without large effort.


We need the line following demo in order to collect the needed bag files. To obtain a variety of the lightning, we will experiment with different light sources as well as different directions from which we illuminate the Duckietown.

### Annotation

As we have to know where the different lines are in the image, we need annotated images. The images should be annotated with at least one polygon per color, i.e. for the 'street black', the 'white outer line', the 'yellow center line' and the 'red stop line'. We would prefer multiple polygons per color. We have tried to use [thehive.ai](https://thehive.ai/) to obtain the annotaded images.




### Analysis

It seems that they can deliver the polygons around the lines on the street. However, it is not clear yet if it is possible to get the information of which polygon contains which color. If this is not the case, we will implement a simple routine which assigns every polygon to a color, based on the color distance described above.

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
