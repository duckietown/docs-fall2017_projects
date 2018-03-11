#  The Saviors: Final Report {#saviors-final-report status=beta}

TODO: JT: fix math formatting, video aspect ratio, standardize apperance 

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

This is the final report of the fall 2017 Saviors group from ETH Zurich, namely Fabio Meier (fmeier@ethz.ch), Julian Nubert (nubertj@ethz.ch), Fabrice Oehler (foehler@ethz.ch) and Niklas Funk (nfunk@ethz.ch). We enjoyed contributing to this great project and in case there are any open questions left after having read this report, do not hesitate to contact us.

## Structure  
<!--
unfortunately cannot be generated on own but thought would be helpful to have one in such huge report!
-->  
[The Final Result](#saviors-final-result)

[Mission and Scope](#saviors-final-scope)

[Definition of the Problem](#saviors-final-problem-def)

[Contribution / Added Functionality](#saviors-final-contribution)

[Formal Performance Evaluation / Results](#saviors-final-formal)

[Future Avenues of Development](#saviors-final-next-steps)

[Theory Chapter](#saviors-theory-chapter)

## The Final Result {#saviors-final-result}

The Saviors Teaser: 

[![Vimeo](vimeo_screenshot.png){:height="50%" width="50%"}](https://player.vimeo.com/video/251523150 "The Saviors Teaser - Click to Watch!")

Note: See the [operation manual](#demo-saviors) to reproduce these results. 

The code description can be found here in the [Readme](https://github.com/duckietown/Software/blob/devel-saviors-23feb/catkin_ws/src/25-devel-saviors/obst_avoid/README.md).

## Mission and Scope {#saviors-final-scope}
<!--Define what is your mission here.-->

**_“URBEM ANATUM TUTIOS FACIENDA (EST) - MAKE DUCKIETOWN A SAFER PLACE”_**

The goal of Duckietown is to provide a relatively simple platform to explore, tackle and solve many problems linked to autonomous driving. “Duckietown” is simple in the basics, but an infinitely expandable environment. From controlling single driving Duckiebots until complete fleet management, every scenario is possible and can be put into practice. Due to the previous classes and also the great work of many volunteers, many software packages were already developed and provided a solid basis. But something was still missing.   

### Motivation {#saviors-final-result-motivation}
<!--_Now step back and tell us how you got to that mission._
- What are we talking about? [Brief introduction / problem in general terms]
- Why is it important? [Relevance]-->

So far, none of the mentioned modules was capable of reliably detecting obstacles and reacting to them in real time. We were not the only ones who saw a problem in the situation at that time: _“Ensuring safety is a paramount concern for the citizens of Duckietown. The city has therefore commissioned the implementation of protocols for guaranteed obstacle detection and avoidance.”_[](#bib:Tani2017). Therefore the foundation of our complete module lies in the disposal of this shortcoming. 
Finding a good solution for this safety related and very important topic helped us to stay motivated every day we were trying to improve our solution.

The goal of our module is to detect obstacles and react accordingly. Due to the limited amount of time, we focused the scope of our module to two points: 

**1.** In terms of detection, on the one hand we focused to reliably detect yellow duckies and therefore to saving the little duckies that want to cross the road. On the other hand we had to detect orange cones to not crash into any construction site in Duckietown.

**2.** In terms of reacting to the detected obstacles we were mainly restricted by the constraint given by the controllers of our Duckiebots, who do not allow us to cross the middle of the road. This eliminated the need of also having to implement a Duckiebot detection algorithm. So we focused on writing software which tries to avoid obstacles within our own lane if it is possible (e.g. for avoiding cones on the side of the lane) and to stop otherwise. 

Besides aforementioned restrictions and simplifications we faced the general problem of detecting obstacles given images from a monocular RGB camera mounted at the front of our Duckiebot and reacting to them properly without crashing or erroneously stopping the Duckiebot. Both processes above have to be implemented and have to run on the Raspberry Pis in real time. Due to the strong hardware limitations, we decided to not use any learning algorithms for the obstacle detection part. 
As it later transpired, a working "hard coded" software needs thorough analysis and understanding of the given problem.
However, in the future, considering additional hardware like e.g. [Tung, "Google offers Raspberry Pi owners this new AI vision kit" (2017)](http://www.zdnet.com/article/google-offers-raspberry-pi-owners-this-new-ai-vision-kit-to-spot-cats-people-emotions/), this decision might have to be reevaluated. 

In practice a well working obstacle detection is one of the most important parts of an autonomous system to improve the reliability of the outcome even in unexpected situations. Therefore the relevance of an obstacle detection in a framework like "Duckietown" is very important. Especially because the aim of "Duckietown" is to simulate the real world as realistic as possible and also in other topics such as fleet planning, a system with obstacle detection behaves completely different than a system without. 

### Existing Solution {#saviors-final-literature}
<!--- Was there a baseline implementation in Duckietown which you improved upon, or did you implemented from scratch? Describe the "prior work"-->

There was a previous implementation from the MIT classes in 2016. Of course we had a look into the old software and found out that one step of them was quite similar to ours: They based their obstacle detection on the colors of the obstacles. Therefore they also did their processing in the HSV color space as we did. Further information on why filtering colors in the HSV space is advantageous can be found in the [Theory Chapter](#saviors-HSV).

Nevertheless, we implemted our solution from scratch and didn't base ours on any further concepts found in their software. That is why you won't find any further similarites between the two implementations. The reasons for implementing our own code from scratch can be found in the next section [Opprtunity](#saviors-final-opportunity). In short, last year's solution considered the image given the original camera's perspective and tried to classify the objects based on their contour. We are using a very different approach concerning those two crucial parts as you can see in the [Contribution](#saviors-final-contribution) section.

### Opportunity {#saviors-final-opportunity}
<!--at didn't work out with the existing solution? Why did it need improvement?
Examples:
- there wasn't a previous implementation
- the previous performance, evaluated according to some specific metrics, was not satisfactory
- it was not robust / reliable
- somebody told me to do so (/s)
* How did you go about improving the existing solution / approaching the problem? [contribution]
- We used method / algorithm xyz to fix the gap in knowledge (don't go in the details here)
- Make sure to reference papers you used / took inspiration from-->

From the beginning it was quite clear that the old software was not working reliable enough. The information we have been given was that it was far off detecting all obstacles and that there were quite a few false positives: It detected yellow line segments in the middle of the road as obstacles (color and size are quite similar to the ones of typical duckies) which led to a stopping of the car. Furthermore, extracting the contour of every potential obstacle is highly computationally expensive. As mentioned, we had a look into the software and tried to understand it as well as possible but because it was not documented at all we couldn't go much into detail. On top of that, from the very beginning we had a completely different idea of how we wanted to tackle these challenges. 

We also tried to start their software but we couldn't make it run after a significant amount of time. The readme file didn't contain any information and the rest of the software was not documented as well. This also reinforced us in our decision to write our own implementation from scratch. 

### Preliminaries {#saviors-final-preliminaries}
<!--- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.
Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)-->

Since our task was to reliably detect obstacles using a monocular camera only, we mainly dealt with processing the camera image, extracting the needed information, visualizing the results and to act accordingly in the real world. 

For understanding our approach we tried to explain and summarize the needed concepts in the theory chapter (see section [Theory Chapter](#saviors-theory-chapter)). There you will find all the references to the relevant sources. 

## Definition of the Problem {#saviors-final-problem-def}
<!--_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical definition of the problem you tackled. You can use part of the preliminary design document to fill this section._
Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal -->

In this chapter we try to explain our problem in a more scientific way and want to show all needed steps to fullfill the superordinate functionality of **"avoiding obstacles"**.

The only input is a RGB colored image, taken by a monocular camera (only one camera). The input image could look as [](#fig:image_start).

<center><img figure-id="fig:image_start" figure-caption="Sample image including some obstacles" src="image_start.jpg" style="width: 200px;"/></center>

With this information given, we want to find out whether an obstacle is in our way or not. If so, we want to either stop or adapt the trajectory to pass without crashing into the obstacle. This information is then forwarded as an output to the controllers who will process out commands and try to act accordingly. 

Therefore one of the first very important decisions was to separate the _detection_ and _reaction_ parts of our **saviors pipeline**. This decision allowed us to divide our work efficiently, to start right away and is supposed to ensure a wide range of flexibility in the future by making it possible to easily replace, optimize or work on one of the parts separately (either the obstacle avoidance strategies or obstacle detection algorithms). 
Of course it also includes having to define a clear, reasonable interface in between the two modules, which will later be explained in detail. 

You can have a look in our [Preliminary Design Document](#saviors-pdd) and [Intermediate Report](#saviors-int-report) to see how we defined the following topics in the beginning: The problem statement, our final objective, the underlying assumptions we lean on and the performance measurement to quantitatively check the perfomance of our algorithms. For the most part, it worked out to adhere to this document but for sake of completeness we will shortly repeat them again in the following for each of the two submodules. 

### Part 1: Computer Vision - Description {#saviors-definition-computer-vision}

In principle we wanted to use the camera image only to ***reach the following***:

1. **Detect** the obstacles in the camera image
2. **Viusalize** them in the camera image for tuning parameters and optimizing the code
3. Give the **3D coordinates** of every detected obstacle in the real world
4. Give the **size** of every detected obstacle in the form of a radius around the 3D coordinate
5. Label each obstacle if it's **inside or outside the lane boundaries** (e.g. for the purpose of not stopping in a curve)
6. **Visualize** them as markers in the 3D world (rviz)

Since every algorithm has its limitations, we made the following ***assumptions***: 

* Obstacles are only yellow duckies and orange cones
* Calibrated camera including intrinsics and extrinsics

Those assumptions changed slightly since the _Preliminary Design Document_ because we are now also able to detect duckies on the middle line and in intersections.

It was our aim to reach the maximum within these specified limits. Therefore our goal was not only the detection and visualization in general but we also wanted to reach a ***maximum in robustness*** with respect to changes in:

* Obstacle size
* Obstacle color (within orange, and yellow to detect different traffic cones and duckies)
* Illumination

For evaluating the ***performance***, we used the following metrics, evaluated under different light conditions and different velocities (static and in motion):

* Percentage of correctly classified obstacles on our picture datasets
* Percentage of false positives
* Percentage of missed obstacles

An evaluation of our goals and the reached performance can be found in the [Performance Evaluation](#saviors-final-formal) section. 

Our ***approach*** is simply based on analysing incoming pictures for obstacles and trying to track them to make the algorithm more robust against outliers. Since we only rely on the monocular camera, we do not have any direct depth information given. In theory, it would be possible to estimate the depth of each pixel through some monocular visual odometry algorithm considering multiple consecutive images. However this would be extremely computationally expensive. The large amount of motion blur in our setup, a missing IMU (for estimating the absolute scale) further argue against such an approach. 
In our approach we use the extrinsic calibration to estimate the position of the given obstacles. The intuition behind that is that it is possible to assume that all pixels seen from the camera belong to the ground plane (except for obstacles which stand out of it) and that the Duckikebot's relative position to this ground plane stays constant. Therefore you can assign a real world 3D coordinate to every pixel seen with the camera. For more details refer to the [section below](#saviors-functionality-computer-vision).

The final output is supposed to look as [](#fig:part_1_image_final).

<center><img figure-id="fig:part_1_image_final" figure-caption="Final output image including visualization of detected obstacles" src="part_1_image_final.jpg" style="width: 200px;"/></center>

### Part 2: Avoidance in Real World - Description {#saviors-definition-avoidance} 

With the from [Part 1](#saviors-definition-computer-vision) given 3D position, size and the labelling whether the object is inside the lane boundaries or not, we wanted to reach the final objectives:

1. Plan path around obstacle if possile (we have to stay within our lane)
2. If this is not possible, simply stop

The assumptions for correctly reacting to the previously detected obstacles are:

* Heading and position relative to track given
* "The Controllers" are responsible for following our trajectory
* Possibility to influence vehicle speed (slow down, stop)

As we now know, the first assumption is normally not fulfilled. We describe in the [functionality section](#saviors-functionality-avoidance) why this comes out to be a problem. 

For measuring the performance we used:

* Avoid/hit ratio
* Also performed during changing light conditions

## Contribution / Added Functionality {#saviors-final-contribution}
<!--Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them
_Feel free to create subsections when useful to ease the flow_-->
### Software Architecture {#saviors-software-architecture}

In general we have four interfaces which had to be created throughout the implementation of our software:

**1.** At first, we need to recieve an incoming picture which we want to analyse. As our chosen approach includes filtering for specific colors, we are obviously dependent on the lighting conditions. In a first stage of our project, we nevertheless simply subscribed to the raw camera image because of the considerable expense of integrating the _Anti Instagram Color Transformation_ and since the _Anti Instagram_ team also first had to further develop their algorithms. During our tests we quickly recognized that our color filtering based approach would always have some troubles if we don't compensate for the lighting change. Therefore, in the second part of the project we closely collaborated with the _Anti Instagram team_ and are now subscribing to a color corrected image provided by them. Currently, to keep computational power on our Raspberry Pi low, the corrected image is published at 4Hz only and the color transformation needs at most 0.2 seconds.

**2.** The second part of our System Integration is the internal interface between the object detection and avoidance part. The interface is defined as a _PoseArray_ which has the same timestamp as the picture from which the obstacles have been extracted. This Array, as the name already describes, is made up of single poses. The meaning of those are the following: 

The position _x and y_ describe the real world position of the obstacle which is in our case the center front coordinate of the obstacle. Since we assume planarity, the _z coordinate_ of the position is not needed. That is why we are using this _z coordinate_ to describe the radius of the obstacle. 

Furthermore a negative _z coordinate_ shows that there is a white line in between us and the obstacle which indicates that it is not dangerous to us since we assume to always having to stay in the lane boundaries. Therefore this information allows us to not stop if there is an obstacle behind a turn. 

As for the scope of our project, the orientation of the obstacles is not really important, we use the _remaining four elements_ of the Pose Message to pass the pixel coordinates of the bounding box of the obstacle seen in the bird view. This is not needed for our “Reaction” module but allows us to implement an efficient way of visualisation which will be later described in detail. Furthermore, we expect our obstacle detection module to add an additional delay of about max. 0.3s.

**3.** The third part is the interface between our obstacle avoidance node and the _Controllers_. The obstacle avoidance node generates an obstacle avoidance _pose array_ and obstacle avoidance _active flag_. 

The obstacle avoidance pose array is the main interface between the Saviors and the group doing lane control. We use the pose array to transmit _d_ref_ (target distance to middle of the lane) and _v_ref_ (target robot speed). The _d_ref_ is our main control output which enables us to position the robot inside the lane and therefore to avoid objects which are placed close the laneline on the track. Furthermore _v_ref_ is used to stop the robot when there is an unavoidable object by setting the target speed to zero. 

The _flag_ is used to communicate to the lane control nodes when the obstacle avoidance is activated which then triggers _d_ref_ and _v_ref_ tracking.

**4.** The fourth part is an optional interface between the Duckiebot and the user's personal Laptop. Especially for the needs of debugging and infering what is going on, we decided to implement a visualisation node which can visualize on the one hand the input image including bounding boxes around all the objects which were classified as obstacles and furthermore this node can output the obstacles as markers which can be displayed in rviz. 

In the following ([](#fig:overview)) you find a graph which summarises our software packages and gives a brief overview. 

<center><img figure-id="fig:overview" figure-caption="Module overview 'The Saviors'" src="overview.png" style="width: 500px;"/></center>


### Part 1: Computer Vision - Functionality {#saviors-functionality-computer-vision}

Let's again have a look on the usual incoming camera picture in [](#fig:image_start). 

In the very beginning of the project, like the previous implementation in 2016, we tried to do the detection in the normal camera image but we tried to optimize for more efficient and general obstacle descriptors. Due to the specifications of a normal camera, lines which are parallel in the real world are in general not parallel any longer and so the size and shape of the obstacles are disturbed (elements of the same size appear also larger in the front than in the back). This made it very difficult to reliably differentiate between yellow ducks and line segments. We tried several different approaches to overcome this problem, namely: 

* Patch matching of duckies viewed from different directions
* Patch matching with some kind of an ellipse (because line segments are supposed to be square)
* Measuring the maximal diameter
* Comparing the height and the width of the objects
* Taking the pixel volume of the duckies

Unfortunately none of the described approaches provided a sufficient performance. Also a combination of them didn't make the desired impact. All metrices which are somehow associated with the size of the object just won't work because duckies further away from the duckiebot are simply a lot smaller than the one very close to the Duckiebot. All metrices associated with the "squareness" of the lines were strongly disturbed by the ocurring motion blur. This makes finding a general criterion very difficult and made us think about changing the approach entirely.

Therefore we developed and came up with the following new approach!
 

#### **Theoretical Description** {#saviors-computer-vision-theoretical}

In our setup, through the extrinsic camera calibration, we are given a mapping from each pixel in the camera frame to a corresponding real world coordinate. It is important to mention that this transformation assumes all seen pixels in the camera frame to lie in one plane which is in our case the ground plane/street. Our approach exactly exploits this fact by transforming the given camera image into a new, bird's view perspective which basically shows one and the same scene from above. Therefore the information provided by the extrinsic calibration is essential for our algorithm to work properly. In [](#fig:bird_view) you can see the newly warped image seen from the _bird's view perspective_. This is one of the most important steps in our algorithm. 

<center><img figure-id="fig:bird_view" figure-caption="Image now seen from the bird's view perspective" src="bird_view.png" style="width: 200px;"/></center>

This approach has already been shown by Prof. Davide Scaramuzza (UZH) and some other papers and is referred as **Inverse Perspective Mapping Algorithm**. (see: [](#bib:Scaramuzz2005),[](#bib:GangYiJiang2000),[](#bib:MassimoBertozzi1997)) 

What stands out, is that the lines which are parallel in the real world are also parallel in this view. Generally in this “bird's” view, all objects which really belong to the ground plane are represented by their real shape (e.g. the line segments are exact rectangles) while all the objects which are not on the ground plane (namely our obstacles) are heavily disturbed in this top view. This top view is roughly keeping the size of the elements on the ground whereas the obstacles are displayed a lot larger. 

_The theory behind the calculations and why the objects are so heavily distorted can be found in the [Theory Chapter](#saviors-transformations)._

Either way, we take advantage of this property. Given this bird's view perspective, we still have to extract the obstacles from it. To achieve this extraction, we first filter out everything except for orange and yellow elements, since we assumed that we only want to detect yellow duckies and orange cones. To simplify this step significantly, we transform the obtained color corrected images (provided by the Anti Instagram module) to the **_HSV color space_**. We use this HSV color space and not the RGB space because it is much easier to account for slightly different illuminations - which of course still exist since the performance of the color correction is logically not perfect - in the HSV room compared to RGB. For the theory behind the HSV space, please refer to our appropriate [Theory Chapter](#saviors-HSV).

After this first color filtering process, there are only objects remaining which have approximately the colors of the expected obstacles. For the purpose of filtering out the real obstacles from the bunch of all the remaining objects which passed the color filter, we decided to do the following: We segment the image of the remaining objects, i.e. all connected pixels in the filtered image are getting the same label such that you can later analyse the objects one by one. Each number then represents an obstacle. For the process of segmentation, we used the following algorithm. (see [](#bib:Wu2005))

Given the isolated objects, the task remains to finally decide which objects are considered obstacles and which not. In a first stage, there is a filter criterion based on a rotation invariant feature, namely the two eigenvalues of the inertia_tensor of the segmented region when rotating around its center of mass. (see [](#bib:Fitzpatrick2011))

In a second stage, we apply a tracking algorithm to reject the remaining outliers and decrease the likelihood for misclassifications. The tracker especially aims for objects which passed the first stage's criterion by a small margin.  

For further informations and details about how we perform the needed operations, please refer to the next chapter.

The final output of the detection module is the one we showed in [](#fig:part_1_image_final).

#### **Actual Implementation** {#saviors-comp-vis-imp}

Now we want to go more into detail how we implemented the described steps. 

In the beginning we again start from the picture you can see in [](#fig:image_start). In our case this is now the corrected image coming out form the _image_transformer_node_ and was implemented by the _anti instagram_ group. We then perform the follwing steps:

**1.** In a first step we crop this picture to make our algorithm a little bit more efficient and due to our limited velocities, it makes no sense to detect obstacles which are not needed to be taken into consideration by our obstacle avoidance module. However, we do not simply crop the picture by a fixed amount of pixels, but we use the extrinsic calibration to neglect all the pixels which are farther away than a user defined threshold, which is at the moment at 1.7 meters. So the amount of pixels which are neglected are different for every Duckiebot and depend on the extrinsic calibration. The resulting image can be seen in [](#fig:cropped). The calculations to find out where You have to cut the image are quite simple (note that it still bargains for homogeneous coordinates): 

\[
    p_{camera} = H^{-1}P_{world}
\] 

<center><img figure-id="fig:cropped" figure-caption="Cropped image" src="image_cropped.png" style="width: 200px;"/></center>

**2.** Directly detecting the obstacles from this cropped input image failed for us due to the reasons descibed [above](#saviors-functionality-computer-vision). That is why the second step is to perform the transformation to the bird’s view perspective. For transforming the image, we first use the corners of the cropped image and transorm it to the real world. Then we scale the real world coordinates to pixel coordinates, so that it will have a width of 640 pixels afterwards. For warping all of the remaining pixels with low artifacts we use the function _cv2.getPerspectiveTransform()_. The obtained image can be seen in [](#fig:bird_view). 

**3.** Then we transform the given RGB picture into the HSV colorspace and apply the yellow and orange filter. While a HSV image is hardly readable for humans, it is way better to filter for specific colors. The obtained pictures can be seen in [](#fig:yellow_filtered) and [](#fig:orange_filtered). The color filter operation is performed by the cv2 function _cv2.inRange(im_test, self.lower_yellow, self.upper_yellow)_ where lower_yellow and upper_yellow are the thresholds for yellow in the HSV color space. 

<center><img figure-id="fig:yellow_filtered" figure-caption="Yellow filtered image" src="yellow_filtered.png" style="width: 200px;"/></center>

<center><img figure-id="fig:orange_filtered" figure-caption="Orange filtered image" src="orange_filtered.png" style="width: 200px;"/></center>

**4.** Now there is the task of segmenting/isolating the objects which remained after the color filtering process. At the beginning of the project we therefore implemented our own segmentation algorithm which was however more inefficient and led to an overall computational load of 200% CPU usage and a maximum frequency of our whole module of about 0.5 Hz only. By using the scikit-image module which provides a very efficient [label function](http://scikit-image.org/docs/dev/api/skimage.measure.html#skimage.measure.label), the computational efficiency could be shrunk considerably to about 70% CPU usage and allows the whole module to run at up to 3 Hz. It is important to remember that in our implementation the segmentation process is the one which consumes the most power. The output after the segmentation is the one in [](#fig:segmented), where the different colors represent the different segmented objects.

<center><img figure-id="fig:segmented" figure-caption="Segmented image" src="segmented.png" style="width: 200px;"/></center>

**5.** After the segmentation, we analyse each of the objects separately. At first there is a general filter which ensures that we are neglecting all the objects which contain less than a user influenced threshold of pixels. Since as mentioned above, the homographies of all the users are different, the exact amount of pixels, an object is required to have, is again scaled by the individual homography. This is followed by a more in detail analysis which is color dependent. On the one hand there is the challenge to detect the orange cones reliably. Speaking about cones, the only other object that might be erroneously detected as orange are the stop lines. Of course, in general the goal should be to very reliably detect orange but as the light is about to change during the drive, we prepared to also detect the stop lines and being able to cope with them when they are erroneously detected. The other general challenge was that all objects that we have to detect can appear in all different orientations. Simply inferring the height and width of the segmented box, as we did it in the beginning, is obviously not a very good measure (e.g. in [](#fig:bird_view_box) in the lower left the segmented box is square while the cone itself is not quadratic at all). 

<center><img figure-id="fig:bird_view_box" figure-caption="Bird's view with displayed obstacle boxes" src="bird_view_box.png" style="width: 200px;"/></center>

That is why it is best to use a rotation invariant feature to classify the segmented object. In our final implementation we came up with using the two eigenvalues of the inertia tensor, which are obviously rotation invariant (when being ordered by their size). Being more specific about the detection of cones, when extracting the cone from [](#fig:segmented) it is looking like in [](#fig:segmented_cone), while an erroneous detection of a stop line is looking like in [](#fig:segmented_stop_line).

<center><img figure-id="fig:segmented_cone" figure-caption="Segmented cone" src="segmented_cone.png" style="width: 200px;"/></center>

<center><img figure-id="fig:segmented_stop_line" figure-caption="Segmented stop line" src="segmented_stop_line.png" style="width: 200px;"/></center>

Our filter criteria is now the ratio between the eigenvalues of the inertia tensor. This ratio is always by a factor of about 100 greater in case the object is a cone, compared to when we erroneously segment a red stop line. This criteria is very stable that is why there is no additional filtering needed to detect the cones.

If the segmented object is yellowish, things get a little more tricky as there are always many yellow objects in the picture, namely the middle lines. Line elements can be again observed under every possible orientation. Therefore the eigenvalues of the inertia tensor, which are as mentioned above rotation invariant, are again the way to go. In [](#fig:segmented_middle_line) you can see a segmented line element and in [](#fig:segmented_duck) again a segmented duckie. 

<center><img figure-id="fig:segmented_middle_line" figure-caption="Segmented middle line" src="segmented_middle_line.png" style="width: 200px;"/></center>

<center><img figure-id="fig:segmented_duck" figure-caption="Segmented duckie" src="segmented_duck.png" style="width: 200px;"/></center>

As the labelled axis already reveal, they are of a different scale, but as we also got very small duckies, we had to choose a very small threshold. To detect the yellow duckies, the initial condition is that the first eigenvalue has to be greater than 20. This criteria alone however includes to sometimes erroneously detecting the lines as obstacles, that is why we implemented an additional tracking algorithm which works as follows:
If an object’s first eigenvalue is greater than 100 pixels and it is detected twice - meaning in two consecutive images there is a object detected at roughly the same place - it is labelled as an obstacle. However, if an object is smaller or changed the size by more than 50% in the consecutive frames, then a more restrictive criteria is enforced. This more restrictive criterion states that we must have tracked this object for at least for 3 consecutive frames before being labelled as an obstacle. This criteria is working pretty well and a more thorough evaluation will be provided in the next [section](#saviors-final-formal). In general those criteria help that the obstacles can be detected in any orientation. The only danger to the yellow detecting algorithm is motion blur, namely when the single lines are not separated but connected together by “blur”. 

**6.** After analysing each of the potential obstacle objects, we decide whether it is an obstacle or not. If so, we continue to steps **7.** and **8.**. 

**7.** Afterwards, we calculate the position and radius of all of the obstacles. After segmenting the object we calculate the 4 corners (which are connected in [](#fig:position_size) to form the green rectangle). We defined the obstacle's position as the midpoint of the lower line (this point surely lies on the ground plane). For the radius, we use the distance in the real world between this point and the lower right corner. This turned out to be a good approximation of the radius. For an illustration you can have a look at [](#fig:position_size).

<center><img figure-id="fig:position_size" figure-caption="Position and radius of the obstacle" src="position_size.jpg" style="width: 200px;"/></center>

**8.** Towards the end of the project we came up with one additional last step based on the idea that only obstacles inside the white lane boundaries are of interest to us. That is why for each obstacle, we look whether there is something white in between us and the obstacle. In [](#fig:classification) you can see an example situation where the obstacle inside the lane is marked as dangerous (red) while the other one is marked as not of interest to us since it is outside the lane boundary (green). In [](#fig:search_line) you see the search lines (yellow) along which we search for white elements.

<center><img figure-id="fig:classification" figure-caption="Classification if objects are dangerous or not" src="classification.png" style="width: 200px;"/></center>

<center><img figure-id="fig:search_line" figure-caption="Search lines to infer if something white is in between" src="search_line.jpg" style="width: 200px;"/></center>

**9.** As the last step of the detection pipeline we return a list of all obstacles including all the information via the _Posearray_.

### Part 2: Avoidance in Real World - Functionality {#saviors-functionality-avoidance} 

The Avoidance deals with drawing the right conclusions from the received data and forwarding it. 

#### **Theoretical Description** {#saviors-avoidance-theoretical}

With the separation of the detection, an important part of the avoidance node is the interaction with the other work packages.
We determined the need of getting information about the remaining Duckietown besides the detected obstacles. The obstacles need to be in relation to the track, in order to assess whether we have to stop, can drive around obstacles or if it is even already out of track. Due to other teams already working on the orientation within Duckietown, we deemed it best to not implement any further detections (lines, intersections etc.) in our visual perception pipeline. This saves similar algorithms being run twice on the processor. We decided to acquire the values of our current pose relative to the side lane, which is determined by the _devel-linedetection_ group.

The idea was to make the system highly flexible. The option to adapt to following situations was deemed desirable:

* Multiple obstacles. Different path planning in case of a possible avoidance might be required.
* Adapted behavior if the robot is at intersections.
* Collision avoidance dependent on the fleet status within the Duckietown. Meaning if a Duckiebot drives alone in a town it should have the option to avoid a collision by driving onto the opposite lane.

Obstacles sideways of the robot were expected to appear as the Duckietowns tend to be flooded by duckies. Those detections on the side as well as far away false positive detections should not make the robot stop. To prevent that, we intended on implementing a parametrized bounding box ahead of the robot. Only obstacles within that box would be considered. Depending on the certainty of the detections as well as the yaw-velocities the parametrization would be tuned.

The interface getting our computed desired values to impact the actual Duckiebot is handled by _devel-controllers_. We agreed on the usage of their custom message format, in which we send desired values for the lateral lane position and the longitudinal velocity. Our intention was to account for the delay of the physical system in the avoider node. Thus our planned trajectory will reach the offset earlier than the ideal-case trajectory would have to.

Due to above mentioned interfaces and multiple levels of goals we were aiming for an architecture which allows **gradual commissioning**. The intent was to be able to go from basic to more advanced for us as well as for groups in upcoming years. Those should be able to extend our framework and not have to rebuild it.

The logic shown in [](#fig:avoidance_logic) displays one of the first stages in the commissioning. Key is the reaction to the number of detected obstacles. Later stages will not trigger an emergency stop in case of multiple obstacle detections within the bounding box.

<center><img figure-id="fig:avoidance_logic" figure-caption="Logic of one of the First Stages in Commissioning" src="avoidance_logic.png" style="width: 500px;"/></center>

Our biggest concern were the added inaccuracies until the planning of the trajectory. Those include:

* Inaccuracy of the currently determined pose
* Inaccuracy of the obstacle detection
* Inaccuracy of the effectively driven path aka. controller performance

To us the determination of the pose was expected to be the most critical. Our preliminary results of the obstacle detection seemed reasonably accurate. The controller could be tweaked that the robot would rather drive out of the track than into the obstacle. Though an inaccurate estimation of the pose would just widen the duckie artificially.

_Devel-controllers_ did not plan on being able to intentionally leave the lane. Meaning the space left to avoid an obstacle on the side of the lane is tight making above uncertainties more severe.

We evaluated the option to keep track of our position inside the map. Given a decent accuracy of said position we’d be able to create a map of the detected obstacles. Afterwards - especially given multiple detections (also outside of the bounding box) - we could achieve a further estimation of our pose relative to the obstacles. This essentially would mean creating a _SLAM-algorithm_ with obstacles as landmarks. We declared as out of scope given the size of our team as well as the computational constraints.
The goal was to make use of a stable, continuous detection and in each frame react on it.

#### **Actual Implementation** {#saviors-avoidance-implementation}

**Interfaces**

One important part of the Software is the handling of the interfaces, mainly to _devel_controllers_. For further informations on this you can refer to the [Software Architecture Chapter](#saviors-software-architecture).

**Reaction**

The obstacle avoidance part of the problem is handled by an additional node, called the _obstacle_avoidance_node_.
The node uses two main inputs which are the obstacle pose and the lane pose.
The obstacle pose is an input coming from the obstacle detection node, which contains an array of all the obstacles currently detected. Each array element consists of an x and y coordinate of an obstacle in the robot frame (with the camera as origin) and the radius of the detected object. By setting the radius to a negative value, the detection node indicates that this obstacle is outside the lane and should not be considered for avoidance.
The lane pose is coming from the line detection node and contains among other unused channels the current estimated distance to the middle of the lane (d) as well as the current heading of the robot $\theta$.
[](#fig:definitions_top) introduces the orientations and definitions of the different inputs which are processed in the obstacle avoidance node.

<center><img figure-id="fig:definitions_top" figure-caption="Variable Definitions seen from the Top" src="definitions_top.jpg" style="width: 500px;"/></center>

Using the obstacle pose array we determine how many obstacles need to be considered for avoidance. If the detected obstacle is outside the lane and therefore marked with a negative radius by the obstacle detection node we can ignore it. Furthermore, we use the before mentioned bounding box with tunable size which assures that only objects in a certain range from the robot are considered. As soon as an object within limits is inside of the bounding box, the obstacle_avoidance_active flag is set to true and the algorithm already introduced in [](#fig:avoidance_logic) is executed.

**Case 1: Obstacle Avoidance**

If there is only one obstacle in range and inside the bounding box, the obstacle avoidance code in the avoider function is executed.
First step of the avoider function is to transform the transmitted obstacle coordinates from the robot frame to a frame which is fixed to the middle of the lane using the estimated measurements of $\theta$ and d. Doing this transformation allows us to calculate the distance of the object from the middle line. If the remaining space (in the lane (subtracted by a safety avoidance margin) is large enough for the robot to drive through we proceed with the obstacle avoidance, if not we switch to case 2 and stop the vehicle. Please refer to [](#fig:coordinate)

<center><img figure-id="fig:coordinate" figure-caption="Geometry of described Scene" src="coordinate.png" style="width: 500px;"/></center>

If the transformation shows that an avoidance is possible we calculate the d_ref we need to achieve to avoid the obstacle. This is sent to the lane control node and then processed as new target distance to the middle of the lane. The lane control node uses this target and starts to correct the Duckiebot's position in the lane. With each new obstacle pose being generated this target is adapted so that the Duckiebot eventually reaches target position. The slow transition movement allows us to avoid the obstacle even when it is not visible anymore shortly before the robot is at the same level as the obstacle.

At the current stage, the obstacle avoidance is not working due to very high inaccuracies in the estimation of $\theta$. The value shows inaccuracies with an amplitude of 10°, which leads to wrong calculations of the transformation and therefore to misjudgement of the d_ref. The high amplitude of these imprecisions could be transformed to a uncertainty factor of around 3 which means that each object is around 3 times its actual size which means that even a small obstacle on the side of the lane would not allow a safe avoidance to take place. For this stage to work, the estimation of $\theta$ would need significant improvement.

**Case 2: Emergency Stop**

Conditions for triggering an emergency stop:

* More than one obstacle in range
* Avoidance not possible because the obstacle is in the middle of the lane
* **Currently every obstacle detection in the bounding box triggers an emergency stop due to the above reasons**

If one of the above scenarios occurs, an avoidance is not possible and the robot needs to be stopped. By setting the target speed to zero, the lane controller node stops the Duckiebot.
As soon as the situation is resolved by removing the obstacle which triggered the emergency stop, the robot can proceed with the lane following.

These tasks are then repeated at the frame rate of the obstacle detection array being sent. 

### Required Infrastructure - Visualizer {#saviors-visualizer}

Especially when dealing with a vision based obstacle detection algorithm it is very hard to infer what is going on. One has to also keep the visual outputs low, to consume as less computing power as possible, especially on the Raspberry Pi. This is why we decided to not implement one single _obstacle detection node_, but effectively two of them, together with some scripts which should help to tune the parameters offline and to infer the number of false positives, etc.. The node which is designed to be run on the Raspberry Pi is our normal obstacle_detection_node. This should in general be run such that there is no visual output at all but that simply the PoseArray of obstacles is published through this node.

The other node, namely the _obstacle_detection_visual_node_ is designed to be run on your own laptop which is basically visualising the information given by the posearray. There are two visualisations available. On the one hand there is a marker visualisation in rviz which shows the position and size of the obstacles. In here all the dangerous obstacles which must be considered are shown in red, whereas the non critical (which we think that they are outside the lane boundaries) are marked in green. On the other hand there is also a visualisation available which shows the camera image together with bounding boxes around the detected obstacles.
Nevertheless, this online visualisation is still dependent on the connectivity and you can only hardly “freeze” single situations where our algorithm failed. That is why we also included some helpful scripts into our package. One script allows to thoroughly input many pictures and outputs them labelled together with the bounding boxes, while another one outputs all the intermediate steps of our filtering process which allows to fastly adapt e.g. the color thresholds which is in our opinion still the major reason for failure. More information on our created scripts can be found in our [Readme on GitHub](https://github.com/duckietown/Software/blob/devel-saviors-23feb/catkin_ws/src/25-devel-saviors/obst_avoid/README.md).

### Recorded Logs {#saviors-logs}

For being able to thorougly evaluate and tune our algorithms, we recorded various bags, which we uploaded to the [Duckietown logs database](http://logs.duckietown.org/).

## Formal Performance Evaluation / Results {#saviors-final-formal}
<!--_Be rigorous!_
- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?-->

### Evaluation of the Interface and Computational Load {#saviors-eval-interface} 

In general as we are dealing with many color filters a reasonable color corrected image is the key to the good functioning of our whole module, but turned out to be the greatest challenge when it comes down to computational efficiency and performance. As described above we are really dependent on a color corrected image by the *Anti Instagram* module. Throughout the whole project we planned to use their *continuous anti-instagram node* which is supposed to compute a color transformation in fixed intervals of time. However, when it came down we acutally had to change this for the follwing reason: The continuous anti-instagram node, running at an update interval of 10 seconds, consumes a considerable amount of computing power, namely 80%. In addition to that, the image transformer node which is in fact transforming the whole image and currently running at 4 Hz needs another 74% of one kernel. If you now run those two algorithms combined with the lane-following demo which makes the vehicle move and combined with our own code which needs an additional 75% of computing power, our safety critical module could only run at 1.5Hz and resulted in poor behaviour. 

Even if you increase the time interval in which the continuous anti-instagram node computes a new transformation there was no real improvement. That is why in our final setup we let the anti-instagram node once compute a reasonable transformation and then keep this one for the entire drive. Through this measure we were able to safe the 80% share entirely and this allowed our overall node to be run at about 3 Hz with introducing an additional maximal delay of about 0,3 seconds. Nevertheless we want to point out that all the infrastructure for using the continuous anti instagram node in the future is provided in our package.

To sum up, the interface between our node and the Anti Instagram node was for sure developed very well and the collabroation was very good but when it came to getting the code to work, we had to take one step back to achieve good performance. That is why it might be reasonable to put effort into this interface in the future, to being able to more efficiently transform an entire image and to reduce the computational power consumed by the node which continuously computes a new transformation.


### Evaluation of the Obstacle Detection {#saviors-eval-obst-detect}

In general, since our obstacle classification algorithm is based on the rotational invariant feature of the eigenvalues of the inertia tensor it is completely invariant to the current orientation of the duckiebot and its position with respect to the lanes. 

To rigorously evaluate our detection algorithm, we started off with evaluating **static scenes**, meaning the Duckiebot is standing still and not moving at all. Our algorithm performed extremely well in those static situations. You can place an arbitrary amount of obstacles, where the orientation of the respective obstacles does not matter at all, in front of the Duckiebot. In those situations and also combining them with changing the relative orientation of the Duckiebot itself, we achieved a false positive percentage of **below 1%** and we labelled all of the obstacles with respect to the lane boundaries correctly. The only static setup which is sometimes problematic is when we place the smallest duckies very close in front of our vehicle (below 4 centimeters), without approaching them. Then we sometimes cannot detect them. However this problem is mostly avoided during the dynamic driving, since we anyways want to stop earlier than 4 centimeters in front of potential obstacles. We are very happy with this static behaviour as in the worst case, if during the dynamic drive something goes wrong, you can still simply stop and rely upon the fact that the static performance is very good before continuing your drive. In [the log chapter](#saviors-logs) it is possible to find the corresponding logs. 

This in return also implies that most of the misclassification errors during our **dynamic drive** are due to the effect of motion blur, assuming a stable color transformation provided by the anti instagram module. E.g. in [](#fig:motion_blur_error) two line segments in the background “blurred” together for two consecutive frames resulting in being labelled as an obstacle.  

<center><img figure-id="fig:motion_blur_error" figure-caption="Obstacle Detector Error due to motion blur" src="motion_blur_error.jpg" style="width: 500px;"/></center>

Speaking more about of numbers, we took 2 duckiebots at a gain of around 0.6 and performed two drives at different days, so also at different lights and the results are the following:
Evaluating each picture which will be given to the algorithm, we found out that on average, we detect **97%** of all the yellow duckies in each picture. In terms of cones we detect about **96%** of all cones in the evaluated frames. We consider these to be very good results as we have a very low rate of false positives (**below 3%**). 

| Date | #correctly detected duckies | #correctly detected cones | #missed ducks | #missed cones | #false positive | #false position |
| -------- | :---------: | :---------: | :---------: | :---------: | :---------: | ---------: |
| 19/12/2017 | 423 | 192 | 14 | 8 | 9 | 45 |
| Robot:Arki |    |     | 3,2% | 4% | 1,4% | 7,2% |
| Duration:82s |     |     |           |        |          |           |
|---           |     |     |           |        |          |        ---|  
| 21/12/2017 | 387 | 103 | 10 | 5 | 15 | 28 |
| Robot:Dori |    |     | 2,5% | 4,4% | 3% | 5,7% |
| Duration:100s |     |     |           |        |          |           |

When it comes to evaluating the performance of our obstacle classification with respect to classifying them as dangerous or not dangerous our performance is not as good as the detection itself, but we did also not put the same effort into it. As you can see in the table above, we have an error rate of **above 5%** when it comes to determining whether the obstalce's position is inside or outside the lane boundaries (this is denoted as false position in the table above). We are especially encountering problems when there is direct illumination on the yellow lines which are very reflective and therefore appear whitish. [](#fig:detector_missclass) shows such a situation where the current implementation of our obstacle classification algorithm fails. 

<center><img figure-id="fig:detector_missclass" figure-caption="Obstacle Detector Classification Error" src="classification_error.jpg" style="width: 500px;"/></center>

### Evaluation of the Obstacle Avoidance {#saviors-eval-obst-avoid}

Since at the current state we stop for every obstacle which is inside the lane and inside the bounding box, the avoidance process is very stable since it does not have to generate avoidance trajectories. The final performance on the avoidance is mainly relying on the placement of the obstacles:

**1. Obstacle placement on a straight:** If the obstacle is placed on a straight with a sufficient distance from the corner the emergency stop works nearly every time if the obstacle is detected correctly. 

**2. Obstacle in a corner:** Due to the currently missing information of the curvature of the current tile the bounding box is always rectangular in front of the robot. This leads to problems if an obstacle is placed in a corner because it might enter the bounding box very late (if at all). Since the detection very close to the robot is not possible, this can lead to crashes. 

**3. Obstacles on intersection:** These were not yet considered in our scope but still work if the detection is correct. It then behaves similar to case 1. 

Furthermore there a few cases which can lead to problems independent of the obstacle placement: 
**1. Controller oscillations:** If the lane controller sees a lot of lag due to high computing loads or similar its control sometimes start to oscillate. These oscillations lead to a lot of motion blur which can induce problems in the detection and shorten the available reaction time to trigger an emergency stop. 

**2. Controller offsets:** The current size of the bounding box assumes that the robot is driving in the middle of the lane. If the robot is driving with an offset to the middle of the lane it can happen that obstacles at the side of the lane aren't detected. This however rarely leads to crashes because then the robot is just avoiding the obstacle instead of stopping for it. 

While testing our algorithms we saw successfull emergency stops in  10/10 cases for obstacles on a straight and in 3/10 cases for obstacles placed in corners assuming that the controller was acting normally. It is to be noted that the focus was lying on the reliable detections on the straights, which we intended to show on the demo day. 

## Future Avenues of Development {#saviors-final-next-steps}
<!--_Is there something you think still needs to be done or could be improved? List it here, and be specific!_-->

As already described above in the [eval interface section](#saviors-eval-interface), we think that there is still room for **improving the interface** between our code and the *Anti Instagram* module in terms of making the *continouus anti instagram node* as well as the *image transformer node* more computationally efficient. Another interesting thought which might be taken into consideration concerning this interface is the follwoing: As long as the main part of the anti instagram's color correction is linear (which was in most of our cases sufficient), it might be reasonable to just adapt the filter values than to subscribe to a fully transformed image. This effort could save a whole publisher and subscriber and it is obvious that it is by far more efficient to transform a few filter values once than to transform every pixel of every incoming picture. Towards the end of our project we invested some time in trying to get this approach to work but as time was not enough we could not make it. We especially struggled to transform the orange filter values, while it worked for the yellow ones (BRANCH: devel-saviors-ai-tryout2). We think that if in the future one will stick to the current hardware this might be a very interesting approach, also for other software components such as the lane detection or any other picture related algorithms which are based on the concept of filtering colors.

Another idea of our team would be to **exploit the transformation to the bird's view also for other modules**. We think that this approach might be of interest e.g. for extracting the curvature of the road or performing the lane detection from the rather more undistorted top view. 

Another area of improvement would be to further develop our provided scripts to being able to **automatically evaluate** the performance of our entire pipeline. As you can see in our code description in github there is a complete set of scripts available which makes it easily possible to transform a bag of raw camera images to a set of pictures on which we applied our obstacle detector, including the color correction part of Anti Instagram. The only missing step left is an automatic detection whether the drawn box is correct and in fact around an object which is considered to be an obstacle or not.

Furthermore to achieve more general performance propably even adaptions in the hardware might be considered (see [](#bib:2017)) to tune the obstacle detection algorithm and especially its generality. We think that setting up a **neural network** might make it possible to release the restrictions on the color of the obstacles.

In terms of avoidance there would be **possibilities to handle the high inacurracies of the pose estimation** by relying on the lane controller to not leave the lane and just use a kind of closed loop control to avoid the obstacle (use the new position of the detected obstacle in each frame to securely avoid the duckie). Applying filters to the signals, especially the heading estimation, could further improve the behaviour. This problem was detected late in the development and could not be tested due to time constraints.
Going further, having both the line and obstacle detection in the same algorithm would allow the direct information on how far away obstacles are from the track. We expect that this would increase the accuracy compared to computing each individually and bringing it together.

The infrastructure is in place to include new scenarios like obstacles on intersection or multiple detected obstacles inside the bounding box. If multiple obstacles are in proximity, a more sophisticated trajectory generation could be put in place to avoid these obstacles in a safe and optimal way. 

Furthermore the **avoidance in corners** could be easily significantly improved if the line detection would estimate the curvature of the current tile which would enable adaptions to the bounding box oncorner tiles. If the pose estimation is significantly improved one could also implement an adaptive bounding box which takes exactly the form of the lane in front of the robot (see [](#fig:adaptive_bbox))

<center><img figure-id="fig:adaptive_bbox" figure-caption="Adaptive bounding box" src="adaptive_bbox.png" style="width: 300px;"/></center>

## Theory Chapter {#saviors-theory-chapter}

### Introduction to Computer Vision

In general a camera is consisting of a converging lens and an image plane ([](#fig:converging_lens)). In the following theory chapter, we will assume that the picture in the image plane is already undistorted, meaning we preprocessed it and eliminated the lens distortion. 

<center>
<div figure-id="fig:converging_lens">
<img src="thin_lens.png" style="width: 300px;"/>
<figcaption>Simplyfied camera model \ref{bib:Scaramuzz2017}</figcaption>
</div>
</center>

It is quite easy to infer from [](#fig:converging_lens) that for a real world point to be in focus, it has to hold, that both of the "rays" (see [](#fig:converging_lens)) intersect in one point in the image plane, namely in point B. Mathematically written this means:

\[
    1: \frac{B}{A}=\frac{e}{z}
\]
\[
    2: \frac{B}{A}=\frac{e-f}{f}
\]
\begin{equation}
    \Leftrightarrow \frac{e}{f}-1=\frac{e}{z} \label{eq:two}
\end{equation}

This last equation \eqref{eq:two} can be approximated since usually $z \gg f$ such that we effectively arrive at the pin-hole approximation with: $e \approx f$ (see [](#fig:pin_approx))

<center>
<div figure-id="fig:pin_approx">
<img src="pinhole_approx.png" style="width: 300px;"/>
<figcaption>Pinhole camera approximation \ref{bib:Scaramuzz2017}</figcaption>
</div>
</center>

For the pixel coordinate on the image plane it holds: 

$\frac{h'}{h}=\frac{f}{z} \Leftrightarrow h'=\frac{f}{z}*h$.

In a more general case, when you consider a 3 dimensional setup and think of a 2 dimensional image plane you have to add another dimension and it follows that a real world point being at $ \vec{P_W} =  \left( \begin{array}{c} X_W \\ Y_W \\ Z_W \end{array} \right) $ will therefore be projected to the pixels in the image plane: 

$x_{pix}=\alpha * \frac{f}{Z_W}*X_W + x_{offset}$ and $y_{pix}=\beta * \frac{f}{Z_W}*Y_W + y_{offset}$ 

where $\alpha$ and $\beta$ are scaling parameters and $x_{offset}$ and $y_{offset}$ are constants which can be always added. Those equations are usually rewritten in homogeneous coordinates such that we have only linear operations left as:
\[
\lambda * \left( \begin{array}{c} x_{pix} \\ y_{pix} \\ 1 \end{array} \right) =  \left( \begin{array}{ccc} \alpha * f & 0 & x_{offset} \\ 0 & \beta * f & y_{offset} \\ 0 & 0 & 1 \end{array} \right) * \left( \begin{array}{c} X_W \\ Y_W \\ Z_W \end{array} \right)
\] 

\begin{equation}
\Leftrightarrow \lambda * \vec{P_{pix}} = H * \vec{P_W} \label{eq:one}
\end{equation}

Note: In general this Matrix H is what we get out of the intrinsic calibration procedure and it might happen, that if the World frame and Camera frame are not completely aligned that then the (1,2) and (2,1) entry of the H Matrix are not exactly zero.

This equation \eqref{eq:one} and especially [](#fig:pin_approx) clearly show that since in every situation you only know H as well as $x_{pix}$ and $y_{pix}$ of the respective objects on the image plane, there is no way to determine the real position of the object, since everything can only be determined up to a scale ($\lambda$). Frankly speaking you only know the direction in which the object has to be but nothing more, which makes it a very difficult task to infer potential obstacles given the picture of a monocular camera only. This scale ambiguity is illustrated in [](#fig:scale_amb). 

<center>
<div figure-id="fig:scale_amb">
<img src="scale_amb.png" style="width: 300px;"/>
<figcaption>Scale ambiguity \ref{bib:Scaramuzz2015}</figcaption>
</div>
</center>

To conclude, given a picture from a monocular camera only you have no idea at which position the house really is, so without exploiting any further knowledge it is extremely difficult to reliably detect obstacles which is also the main reason why the old approach did not really work. On top of that come other artifacts such as that the same object will appear larger if it is closer to your camera and vice versa, and lines which are parallel in the real world will in general not be parallel in your camera image.

Note: The intuition, why we humans can infer the real scale of objects is that if you add a second camera, know the relative Transformation between the two cameras and see the same object in both images, then you can easily triangulate the full position of the object, since it is at the place where the two "rays" intersect! (see [](#fig:triang)).

<center>
<div figure-id="fig:triang">
<img src="triang.png" style="width: 300px;"/>
<figcaption>Triangulation to obtain absolute scale \ref{bib:Rasmussen}</figcaption>
</div>
</center>

### Inverse Perspective Mapping / Bird's View Perspective {#saviors-transformations}

The first chapter above introduced the rough theory which is needed for understanding the follwing parts. The important additional information that we exploited heavily in our approach is that in our special case we know the coordinate $Z_W$. The reason therefore lies within the fact that unlike in another more general usecase of a mono camera, we know that our camera will always be at height $h$ with repsect to the street plane and that the angle $\theta_0$ also always stays constant. ([](#fig:fixed_cam))   

<center>
<div figure-id="fig:fixed_cam">
<img src="fixed_cam.png" style="width: 300px;"/>
<figcaption>Illustration of our fixed camera position \ref{bib:2012}</figcaption>
</div>
</center>

This information is used in the actual extrinsic calibration such that in Duckietown, due to the assumption that everything we see should in general be on the road, we can determine the full real world coordinates of every pixel, since we know the coordinate $Z_W$ which uniquely defines the absolute scale and can therefore uniquely determine $\lambda$ and H! Intuitively this comes from the fact that we can just intersect the known ray direction (see [](#fig:pin_approx)) with the known "gound plane".

This makes it possible to project every pixel back into the "road plane" by computing for each available pixel:
\[
\vec{P_W}=H^{-1} * \lambda * P_{pix} 
\] 

**This "projection back onto the road plane" is called inverse perspective mapping!**

If you now visualize this "back" projection, you basically get the bird's view since you can now map back every pixel in the image plane to a unique place on the road plane.

The only trick of this easy maths is that we exploited the knowledge that everything we see in the image plane is in fact on the road and has one and the same z-coordinate. You can see that the original input image [](#fig:raw_img) is nicely transformed into the view from above where every texture and shape is nicely reconstructed if this assumption is valid [](#fig:normal_bird). You can especially see that all the yellow line segments in the middle of the road roughly have the same size in this bird's view [](#fig:normal_bird) which is very different if you compare it to the original image [](#fig:raw_img).

<center><img figure-id="fig:raw_img" figure-caption="Normal incoming image without any obstacle" src="4.jpg" style="width: 300px;"/></center>

<center><img figure-id="fig:normal_bird" figure-caption="Incoming image without obstacle reconstructed in bird's view" src="4_bird.png" style="width: 300px;"/></center>

The crucial part is now what happens in this bird's view perspective, if the camera sees an object which is not entirely part of the ground plane, but stands out. These are basically obstacles we want to detect. If we still transform the whole image to the bird's view, these obstacles which stand out of the image plane get heavily disturbed. Lets explain this by having a look at [](#fig:reason_for_dist). 

<center>
<div figure-id="fig:reason_for_dist">
<img src="shadow.png" style="width: 300px;"/>
<figcaption>Illustration why obstacle standing out of ground plane is heavily disturbed in bird's view, modified: \ref{bib:2012}</figcaption>
</div>
</center>

The upper picture in [](#fig:reason_for_dist) depicts the real world situation, where the cone is standing out ot the image plane and therefore the tip is obviously not at the same height as the ground plane. However, as we still have this assumption and as stated above intuitively intersect the ray with the ground plane, the cone gets heavily disturbed and will look like the lower picture in [](#fig:reason_for_dist) after performing the inverse perspective mapping. From this follows that if there are any objects which DO stand out of the image plane then in the inverse perspective you basically see their shape being projected onto the ground plane. This behaviour can be easily exploited since all of these objects are heavily disturbed, drastically increase in size and can therefore be easily separated from the other objects which belong to the ground plane.

Let's have one final look at an example in Duckietown. In [](#fig:incoming_img) you see an incoming picture seen from the normal camera perspective, including obstacles. If you now perform the inverse perspective mapping, the picture looks like [](#fig:bird_img) and as you can easily see, all the obstacles, namely the two yellow duckies and the orange cone which stand out of the ground plane are heavily disturbed and therefore it is quite easy to detect them as real obstacles.

<center><img figure-id="fig:incoming_img" figure-caption="Normal situation with obstacles in Duckietown seen from Duckiebot perspective" src="image_cropped.png" style="width: 300px;"/></center>

<center><img figure-id="fig:bird_img" figure-caption="Same situation seen from bird's perspective" src="bird_view.png" style="width: 300px;"/></center>

### HSV Color Space {#saviors-HSV}

#### **Introduction and Motivation** {#saviors-HSV-motivation}

The "typical" color model is called the RGB color model. It simply uses three numbers for the amount of the colors _red_, _blue_ and _green_. It is an additive color system, so we can simply add two colors to produce a third one. Mathematically written it looks as follows and shows the way of how we deal with producing new colors:

\[
\left( \begin{array}{c} r_{res} \\ g_{res} \\ b_{res} \end{array} \right) =  \left( \begin{array}{c} r_{1} \\ g_{1} \\ b_{1} \end{array} \right) + \left( \begin{array}{c} r_{2} \\ g_{2} \\ b_{2} \end{array} \right)
\] 

If the resulting color is white, the two colors _1_ and _2_ are called to be complementary (e.g. this is the case for blue and yellow). 

This color system is very intuitive and is oriented on how the human vision perceives the different colors. 

The _HSV_ color space is an alternative representation of the RGB color model. On this occasion _HSV_ is an acronym for _Hue_, _Saturation_ and _Value_. It is not so easy summable as the RGB model and it is also hardly readable for humans. So the big question is: **Why should we transform our colors to the HSV space? Does it derive a benefit?**

The answer is yes. It is hardly readable for humans but it is way better to filter for specific colors. If we look at the definition openCV gives for the RGB space, the higher complexity for some tasks becomes obvious:

_In the RGB color space all "the three channels are effectively correlated by the amount of light hitting the surface", so the color and light properties are simply not separated._ (see: [](#bib:GUPTA2017))

Expressed in a more simpler way: In the RGB space the colors also influence the brightness and the brightness influences the colors. However, in the HSV space, there is only one channel - the _H_ channel - to describe the color. The _S_ channel represents the saturation and _H_ the intensity. This is the reason why it is super useful for specific color filtering tasks.

The HSV color space is therefore often used by people who try to select specific colors. It corresponds better to how we experience color. As we let the **_H (Hue)_** channel go from 0 to 1, the colors vary from red through yellow, green, cyan, blue, magenta and back to red. So we have red values at 0 as well as at 1. As we vary the **_S (saturation)_** from 0 to 1 the colors simply vary from unsaturated (more grey like) to fully saturated (no white component at all). Increasing the **_V (value)_** the colors just become brighter. This color space is illustrated in [](#fig:hsv_illustration). (see: [](#bib:2018))

<center>
<div figure-id="fig:hsv_illustration">
<img src="hsv_illustration.png" style="width: 200px;"/>
<figcaption>Illustration of the HSV Color Space \ref{bib:2018}</figcaption>
</div>
</center>

Most systems use the so called RGB additive primary colors. The resulting mixtures can be very diverse. The variety of colors, called the _gamut_, can therefore be very large. Anyway, the relationship between the constitutent amounts of red, green,  and blue lights is unintuitive. 

#### **Derivation** {#saviors-HSV-derivation}

The _HSV_ model can be derived using geometric strategies. The RGB color space is simply a cube where the addition of the three color components (with a scale form 0 to 1) is displayed. You can see this on the left of [](#fig:color_comparison).

<center>
<div figure-id="fig:color_comparison">
<img src="rgb_hsv.jpg" style="width: 400px;"/>
<figcaption>Comparison between the two colors spaces \ref{bib:Alves2015}</figcaption>
</div>
</center>

You can now simply take this cube and tilt it on its corner. We do it this way so that black rests at the orgin whereas white is the highest point directly above it along the vertical axis. Afterwards you can just measure the _hue_ of the colors by their angle around the vertical axis (red is denoted as 0°). Going from the middle to the outer parts from 0 (where the grey like parts are) to 1 determines the _saturation_. This is illustrated in [](#fig:derivation_1). 

<center>
<div figure-id="fig:derivation_1">
<img src="derivation_1.png" style="width: 200px;"/>
<figcaption>'Cutting the cube' \ref{bib:2017a}</figcaption>
</div>
</center>

The definitions of _hue_ and _chroma_ (proportion of the distance from the origin to the edge of the hexagon) amount to a geometric warping of hexagons into circles (for more informations see: [](#bib:2017a)). Each side of the hexagon is mapped linearly onto a 60° arc of the circle. This is visualized in [](#fig:derivation_2).

<center>
<div figure-id="fig:derivation_2">
<img src="derivation_2.png" style="width: 200px;"/>
<figcaption>Warping hexagons to circles \ref{bib:2017a}</figcaption>
</div>
</center>

For the _value_ or lightness there are several possibilities to define an appropriate dimension for the color space. The simplest one is just the average of the three components, which is nothing else then the vertical height of a point in our tilted cubic. For this case we have:

\[
I = 1/3 * (R + G + B)
\] 

For another definition the _value_ is defined as the largest component of a color. This places all three primaries and also all of the "secondary colors" (cyan, magenta, yellow) into a plane with white. This forms a hexagonal pyramid out of the RGB cube. This is called the HSV "hexcone" model and is the common one. We get:

\[
V = max(R, G, B)
\]

(see: ([](#bib:2017a)))

#### **In Practice** {#saviors-HSV-practice}

**1.** Form a hexagon by projecting the RGB unit cube along its pincipal diagonal onto a plane. 

<center>
<div figure-id="fig:hsv_practice_1">
<img src="hsv_practice_1.png" style="width: 400px;"/>
<figcaption>First layer of the cube (left) and flat hexagon (right) \ref{bib:CHENG}</figcaption>
</div>
</center>

**2.** Repeat projection with smaller RGB cube (subtract 1/255 in length of every cube) to obtain smaller projected hexagon. Like this a _HSV hexcone_ is formed by stacking up the 256 hexagons in decreasing order of size. 

<center>
<div figure-id="fig:hsv_practice_2">
<img src="hsv_practice_2.png" style="width: 400px;"/>
<figcaption>Stacking hexagons together \ref{bib:CHENG}</figcaption>
</div>
</center>

Then the value is again defined as:

\[
V = max(R, G, B)
\]

**3.** Smooth edges of hexagon to circles (see previous chapter). 

#### **Application** {#saviors-HSV-application}

One nice example of the application of the HSV color space can be seen in [](#fig:hsv_example).

<center>
<div figure-id="fig:hsv_example">
<img src="hsv_example.png" style="width: 400px;"/>
<figcaption>Image on the left is original. Image on the right was simply produced by rotating the H of each color by -30° while keeping S and V constant \ref{bib:2017a}</figcaption>
</div>
</center>

It just shows how simple color manipulation can be performed in a very intuitive way. We can turn many different applications to good account using this approach. As you have seen, color filtering also simply becomes a threshold query. 
