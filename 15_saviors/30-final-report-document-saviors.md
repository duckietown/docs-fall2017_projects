#  The Saviors: final report {#saviors-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

## The final result {#saviors-final-result}

The Saviors Teaser: 

[![Vimeo](vimeo_screenshot.png){:height="50%" width="50%"}](https://player.vimeo.com/video/251523150 "The Saviors Teaser - Click to Watch!")

See the code description (readme-file): 

See the [operation manual](#demo-template) to reproduce these results.

***LINKS STILL MISSING!***

## Mission and Scope {#saviors-final-scope}
<!--Define what is your mission here.-->

**_“URBEM ANATUM TUTIOS FACIENDA (EST) - MAKE DUCKIETOWN A SAFER PLACE”_**

The goal of Duckietown is to provide a relative simple platform to explore, tackle and solve many problems linked to autonomous driving in the simple but infinitely expandable environment of “Duckietown”. From controlling single driving Duckiebots until complete fleet management, every scenario is possible and can be put into practice. Due to the previous classes and also the great work of many volunteers many software packages were already developed and provided a solid basis. But something was still missing.   

### Motivation {#saviors-final-result-motivation}
<!--_Now step back and tell us how you got to that mission._
- What are we talking about? [Brief introduction / problem in general terms]
- Why is it important? [Relevance]-->

So far, none of the mentioned modules was capable of reliably detecting obstacles and reacting to them in real time. We were not the only ones who saw a problem in the situation at that time: _“ Ensuring safety is a paramount concern for the citizens of Duckietown. The city has therefore commissioned the implementation of protocols for guaranteed obstacle detection and avoidance.”_(cite from the Project Pitch Slides). Therefore the foundation of our complete module lies in the disposal of this shortcoming. 
Finding a good solution for this safety related and very important topic helped us to stay motivated every day we were trying to improve our solution.

The goal of our module is to detect obstacles and react accordingly. Due to the limited amount of time, we limited and focused the scope of our module to two points: 

1. In terms of detection, we focused on the one hand to reliably detecting yellow duckies and therefore to saving the little duckies that want to cross the road and on the other hand to detect orange cones to not crash into any construction site in Duckietown.

2. In terms of reacting to the detected obstacles we were mainly restricted by the constraint given by the controllers of our Duckiebots who do not allow us to cross the middle of the road. This eliminated the need of also having to have a Duckiebot detection algorithm. So we focused on writing software which tries to avoid obstacles within our own lane if it is possible (e.g. for avoiding cones on the side of the lane) and to stop otherwise. 

Besides from those first restrictions and simplifications we faced the general problem of detecting obstacles given images from a monocular RGB camera mounted at the front of our Duckiebot and reacting to them properly without crashing or erroneously stopping the Duckiebot. Both processes above have to be implemented and run on the RasberryPi’s in real time. Due to the strong hardware limitations, we decided to not use any learning algorithms for the obstacle detection part. 
As it later transpired a working "hard coded" software needs thorough analysis and understanding of the given problem.
However, in the future, considering additional hardware like e.g. --- [Tung, "Google offers Raspberry Pi owners this new AI vision kit" (2017)](http://www.zdnet.com/article/google-offers-raspberry-pi-owners-this-new-ai-vision-kit-to-spot-cats-people-emotions/), this decision might be adapted. 

In practice a well working obstacle detection is of course one of the most important parts of an autonomous system to certainly improve the outcome of unexpected situations. Therefore the relevance of an obstacle detection in a framework like "Duckietown" is very important because the aim of duckietown is to simulate the real world as realistic as possible and also in other topics as fleet planning a system with obstacle detection behaves completely different than a system without. 

### Existing solution {#saviors-final-literature}
<!--- Was there a baseline implementation in Duckietown which you improved upon, or did you implemented from scratch? Describe the "prior work"-->

There was a previous implementation from the MIT classes in 2016. Of course we had a look into the old software and found out that one step of them was quite similar to ours: They based their obstacle detection on the colors of the obstacles. Therefore they also did their processing in the HSV color space as we did. Further information on why filtering colors in the HSV space is advantageous can be found ***HERE TODO.!!!! (THEORY CHAPTER!!!)***

Nevertheless, we implemted our solution from scratch and didn't base ours on any further concepts found in their software. That is why you won't find any further similarites between the two implementations. The reasons for implementing our own code from scratch can be found in the next section [Opprtunity](#saviors-final-opportunity). In short, last year's solution considered the image given the original camera's perspective and tried to classify the objects based on their contour. We are using a very different approach concerning those two crucial parts as you can see in the following section. ***TO DO!!!!! INSERT LINK!!!!!!!!!***

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

From the beginning it was quite clear that the old software was not working reliably enough. The information we have been given was that by far it didn't detect all of the obstacles and that there were quite a few false positives: It detected yellow line segments in the middle of the road as obstacles (color and size are quite similar to the ones of typical duckies) which led to the stopping of the car. Furthermore extracting the contour of every potential obstacle is highly computationally expensive. As mentioned we had a look into the software and tried to understand it as well as possible but because it was not documented at all we couldn't go to much into detail. On top of that, from the very beginning we had a completely different idea of how we wanted to tackle these challenges. 

We also tried to start their software but we couldn't make it run after a significant amount of time, the readme file didn't contain any information and the rest of the software was not documented as well. This also reinforced us in our decision to write our implementation from scratch. 

### Preliminaries {#saviors-final-preliminaries}
<!--- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.
Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)-->

Since our task was to reliably detect obstacles using a monocular camera only, we mainly dealt with processing the camera image, extracting the needed information, visualizing the results and to act accordingly in the real world. 

For understanding our approach we tried to explain and summarize the needed concepts in the theory chapter, see section ***LINK INSERTEN!!!! TO DO!!!!!***. There you will find all the references to the relevant sources. 
***TO DO!!!!!***

## Definition of the problem {#saviors-final-problem-def}
<!--_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical definition of the problem you tackled. You can use part of the preliminary design document to fill this section._
Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal -->

In this chapter we try to explain our problem in a more scientific way and to show all needed steps for fullfilling the superordinate functionality of **"avoiding obstacles"**.

The only input is a RGB colored image taken by a monocular camera (only one camera). The input image could look as [](#fig:image_start).

<center><img figure-id="fig:image_start" figure-caption="Sample Image including some Obstacles" src="image_start.jpg" style="width: 200px;"/></center>

With this information given we want to find out whether an obstacle is in our way or not. If so, we want to either stop or adapt the trajectory to pass without crashing into the obstacle. This information is then forwarded as an output to the controllers who will then process out commands and try to act accordingly. 

Therefore one of the first very important decisions was to separate the _detection_ and _reaction_ parts of our **saviors pipeline**. This allowed us to divide our work efficiently and to start right away. This is also supposed to ensure a wide range of flexibility: 
This separation makes it possible to easily replace, optimize or work on one of the parts (either the obstacle avoidance strategies or obstacle detection algorithms) in the future. 
Of course it also includes having to define a clear, reasonable interface in between the two modules, which will later be explained in detail. 

You can have a look in our [Preliminary Design Document](#saviors-PDD-problem-definition) to see how we defined the following topics in the beginning: The problem statement, our final objective, the underlying assumptions we lean on and the performance measurement to quantitatively check the perfomance of our algorithms. For the most part it worked out to adhere to this document but for sake of completeness we will shortly repeat them again in the following for each of the two submodules. 

### Part 1: Computer Vision - Description {#saviors-definition-computer-vision}

In principle we wanted to use the camera image only to ***reach the following***:

1. **Detect** the obstacles in the camera image
2. **Viusalize** them in the camera image for tuning parameters and optimizing the code
3. Give the **3D coordinates** of every detected obstacle in the real world
4. Give the **size** of every detected obstacle in the form of a radius around the 3D coordinate
5. Label each obstacle if it's **inside or outside the lane boundaries** (e.g. for not stopping in a curve)
6. **Visualize** them as markers in the 3D world (rviz)

Of course every algorithm has its limitations, therefore we made the following ***assumptions***: 

* Obstacles are only yellow duckies and orange cones
* Calibrated camera including intrinsics and extrinsics

Those assumptions changed slightly since the _Preliminary Design Document_ because we are now also able to detect duckies on the middle line and in intersections.

Of course it was our aim to reach the maximum within these specifeid limits. Therefore our goal was not only the detection and visualization in general but we also wanted to reach the ***maximum robustness*** to changes in:

* Obstacle size
* Obstacle color (within the orange, and yellow to detect different traffic cones and duckies)
* Illumination

For measuring the ***performance*** we used the following metrics, evaluated under different light conditions and different velocities (static and in motion):

* Percentage of correctly classified obstacles on our picture datasets
* Percentage of false positives
* Percentage of missed obstacles

An evaluation of our goals and the reached performance can be found in the [Performance Evaluation](#saviors-final-formal) section. 

Our ***approach*** is simply based on analysing incoming pictures for obstacles and trying to track them to make the algorithm more robust against outliers. Since we only rely on the monocular camera, we do not have any depth information given. In theory it would be also possible, but computational much more expensive to estimate the depth of each pixel through some monocular visual odometry algorithm considering multiple consecutive images. However, the large amount of motion blur in our setup, a missing IMU (for estimating the absolute scale) and the additional computational costs clearly argue against such an approach. 
In our approach we use the extrinsic calibration to estimate the position of the given obstacles. The intuition behind that is that it is possible to assume that all pixels seen from the camera belong to the ground plane (except for obstacles which stand out of it) and that the Duckikebot's relative position to this ground plane stays constant, so you can assign a real world 3D coordinate to every pixel seen with the camera. For more details refer to the [section below](#saviors-functionality-computer-vision).

The final output is supposed to look as [](#fig:part_1_image_final).

<center><img figure-id="fig:part_1_image_final" figure-caption="Image including Visualization of Detected Obstacles" src="part_1_image_final.jpg" style="width: 200px;"/></center>

### Part 2: Avoidance in Real World - Description {#saviors-definition-avoidance} 

With the from _Part 1_ given 3D position, size and the labelling whether the object is inside the lane boundaries or not, we wanted to reach the final objectives:

1. Plan path around obstacle if possile (we have to stay within our lane)
2. If this is not possible, simply stop

The assumptions for correctly reacting to the previously detected obstacles are:

* Heading and position relative to track given
* "The Controllers" are responsible for follwoing our trajectory
* Possibility to influence vehicle speed (slow down, stop)

As we know now the first point of the assumptions is normally not fulfilled. We describe in the [functionality section](#saviors-functionality-avoidance) why this comes out to be a problem. 

For measuring the performance we used:

* Avoid/hit ratio
* Also performed during changing light conditions

## Contribution / Added functionality {#saviors-final-contribution}
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

**1.** At first, we need to recieve an incoming picture which we want to analyse. As our chosen approach includes filtering for specific colors, we are obviously dependent on the lighting conditions. In a first stage of our project, we nevertheless simply subscribed to the raw camera image because of the considerable expense of integrating the _Anti Instagram Color Transformation_ and since the _Anti Instagram_ team also first had to further develop their algorithms. During our tests we fastly recognized that our color filtering based approach would always have some troubles if we don't compensate for the lighting change. Therefore, in the second part of the project we closely collaborated with the anti-instagram team (especially Milan Schilling) and are now subscribing to a color corrected image provided by the anti-instagram team. Currently, to keep computational power on our RaspberryPi low, the corrected image is published at 4Hz only and the color transformation needs at most 0.2 seconds.

**2.** The second part of our System Integration is the internal interface between the object detection and avoidance part. The interface here is defined as a _PoseArray_ which has the same timestamp as the picture from which the obstacles have been extracted. This Array, as the name already describes, is made up of single poses. The meaning of those are the following: The position _x and y_ describe the real world position of the obstacle which is in our case the center front coordinate of the obstacle. The _z coordinate_ of the position is not needed in our case, since we are on a plane anyways that is why we are using the _z-coordinate_ to describe the radius of the obstacle. Furthermore a negative z-Radius shows that there is a white line in between us and the obstacle which indicates that it is not dangerous to us since we assume to always having to stay in the lane boundaries. Therefore this information allows us to not stop if there is an obstacle behind a turn. As for the scope of our project, the orientation of the obstacles is not really important, we use the _remaining four elements_ of the Pose Message to pass the pixel coordinates of the bounding box of the obstacle seen in the bird view. This is not needed for our “Reaction” module but allows us to implement an efficient way of visualisation which will be later described in detail. Furthermore, we expect our obstacle detection module to add an additional delay of about max. 0.3s.

**3.** The third part is the interface between our obstacle avoidance node and the controllers. The obstacle avoidance node generates an obstacle avoidance pose array and obstacle avoidance active flag. The obstacle avoidance pose array is the main interface between the Saviors and the group doing lane control. We use the pose array to transmit d_ref (target distance to middle of the lane) and v_ref (target robot speed). The d_ref is our main control output which enables us to position the robot inside the lane and therefore to avoid objects which are placed on the side of the lane. Furthermore v_ref is used to stop the robot when there is an unavoidable object by setting the target speed to zero. The flag is used to communicate to the lane control nodes when the obstacle avoidance is activated which then triggers d_ref and v_ref tracking.

**4.** The fourth part is an optional interface between the Duckiebot and the user's personal Laptop. Especially for the needs of debugging and infering what is going on, we decided to implement a Visualisation node which can visualize on the one hand the input image including bounding boxes around all the objects which were classified as obstacles and furthermore this node can output the obstacles as markers which can be displayed in rviz. 

In the following ([](#fig:overview)) you find a graph which summarizes our software packages and gives a brief overview. 

<center><img figure-id="fig:overview" figure-caption="Overview Module 'The Saviors'" src="overview.png" style="width: 600px;"/></center>


### Part 1: Computer Vision - Functionality {#saviors-functionality-computer-vision}

Let's again have a look on the usual incoming camera picture in [](#fig:image_start). 

In the very beginning of the project, like the previous implementation in 2016, we tried to do the detection in the normal camera image but we tried to opt for more efficient and general obstacle descriptors. Due to the specifications of a normal camera, lines which are parallel in the real world are not parallel any longer and so the size and shape of the obstacles are disturbed (elements of the same size appear also larger in the front than in the back). This made it very difficult to reliably differentiate between yellow ducks and line segments. We tried several different approaches to overcome this problem, namely: 

* Patch matching of duckies viewed from different directions
* Patch matching with some kind of an ellipse (because line segments are supposed to be square)
* Measuring the maximal diameter
* Comparing the height and the width of the objects
* Taking the pixel volume of the duckies

Unfortunately none of the described approaches provided a sufficient performance. Also a combination of them didn't make the desired impact. All metrices which are somehow associated with the size of the object just won't work because duckies further away from the duckiebot are simply a lot smaller than the one very close to the Duckiebot. All metrices associated with the "squareness" of the lines were strongly disturbed by the ocurring motion blur. This makes finding a general criterion very difficult and made us think about changing the approach entirely.

**Therefore we developed and came up with the following new approach!**
 

#### **Theoretical Description** {#saviors-computer-vision-theoretical}

In our setup, through the extrinsic camera calibration, we are given a mapping from each pixel in the camera frame to a corresponding real world coordinate. It is important to mention that this transformation assumes all seen pixels in the camera frame to lie in one plane which is in our case in the ground plane/street. Now, our approach exactly exploits this fact by transforming the given camera image into a new, bird view perspective which basically shows one and the same scene from above. Therefore the information provided by the extrinsic calibration is essential for our algorithm to work properly. In [](#fig:bird_view) you can see the newly warped image seen from the _bird view perspective_. Which is one of the most important steps in our algorithm. 

<center><img figure-id="fig:bird_view" figure-caption="Image now seen from the Bird View Perspective" src="bird_view.png" style="width: 200px;"/></center>

This approach has already been shown by Prof. Davide Scaramuzza (UZH) and some other papers and is referred as **Fast inverse Perspective Mapping Algorithm**.  

--- [Davide Scaramuzza (2004)](http://rpg.ifi.uzh.ch/docs/Tesi_Laurea_Davide_Scaramuzza.pdf)
--- [Gang Yi Jiang, Tae Young Choi, Suk Kyo Hong, Jae Wook Bae, and Byung Suk Song (2015)](https://www.researchgate.net/publication/3876051_Lane_and_obstacle_detection_based_on_fast_inverse_perspective_mapping_algorithm)
--- [Massimo Bertozzi, Alberto Broggi, Alessandra Fascioli](http://www.close-range.com/docs/Stereo_inverse_perspective_mapping.pdf)

What stands out is that now the lines which are parallel in the real world are also parallel in this view. Generally in this “bird” view, all objects which really belong to the ground plane are represented by their real shape (e.g. the line segments are exact rectangles) while all the objects which are not on the ground plane (namely our obstacles) are heavily disturbed in this top view. This top view is roughly keeping the size of the elements on the ground whereas the obstacles are displayed a lot larger. 

_The theory behind the calculations and why the objects are so heavily distorted can be found in the [Theory Chapter](#saviors-transformations)._

Either way we take advantage of this poperty. 
Given this bird view perspective, we still have to extract the obstacles from it. At first we apply a filter to filter out all orange elements and yellow elements, since we said that we only want to detect yellow duckies and orange cones. For this we transform the obtained color corrected images (provided by the anti-instagram module) to the HSV color space. We use this HSV color space and not the RGB space because it is much easier to account for slightly different illuminations (which of course still exist since the performance of the color correction is still logically not perfect) in the HSV room compared to RGB. For the theory behind the HSV space, please refer to our appropriate [Theory Chapter](#saviors-HSV).

<center><img figure-id="fig:yellow_filtered" figure-caption="Yellow filtered Image" src="yellow_filtered.png" style="width: 200px;"/></center>

<center><img figure-id="fig:orange_filtered" figure-caption="Orange filtered Image" src="orange_filtered.png" style="width: 200px;"/></center>

After this first color filtering process, there are only objects remaining which have approximately the colors of the expected obstacles(see [](#fig:yellow_filtered) and [](#fig:orange_filtered)). The next challenge is therefore to filter the real obstacles out of the bunch of all the remaining objects which passed the color filter. For this purpose we firstly segment the image of the remaining objects, i.e. all connected pixels (which is in other words one object) which are still remaining in our image are getting the same label such that you can later analyse the objects one by one. For the segmentation we used the following algorithm.

--- [Wu, KeshengOtoo, EkowShoshani, Arie (2005)](https://escholarship.org/uc/item/7jg5d1zn)

<center><img figure-id="fig:segmented" figure-caption="Segmented Image" src="segmented.png" style="width: 200px;"/></center>

Now as we have the segmented (see [](#fig:segmented)) image with all possible obstacle objects we differentiate between the orange and the yellow obstacles. For each of the two colors we apply a tracking algorithm to reject the outliers and to only keep the real obstacles. 
These filters are based on a rotation invariant feature, namely the two eigenvalues of the inertia_tensor of the segmented region when rotating around its center of mass. 

--- [Richard Fitzpatrick, (2011)](http://farside.ph.utexas.edu/teaching/336k/Newtonhtml/node64.html) 

Additionaly we apply some further criterions to also eliminate the last remaining outliers:

* A Movement Tracker filter which enforces the contraint that obstacles can not jump
* A criterion which evaluates the relation between the width and length of the objects
* ...

The final output of the detection module is the one we showed in [](#fig:part_1_image_final).

#### **Actual Implementation** {#saviors-computer-vision-implementation}

Now we want to go more into detail how we implemented the described steps. 

In the beginning we again start from the picture you can see in [](#fig:image_start). In our case this is now the corrected image coming out form the _image_transformer_node_ and was implemented by the _anti instagram_ group. We then perform the follwing steps:

**1.** In a first step we crop this picture to make our algorithm a little bit more efficient and due to our limited velocities, it makes no sense to detect obstacles which are not needed to be taken into consideration by our obstacle avoidance module. However, we do not simply crop the picture by a fixed amount of pixels, but we use the extrinsic calibration to neglect all the pixels which are farther away than a user defined threshold, which is at the moment at 1.7 meters. So the amount of pixels which are neglected are different for every duckiebot and depend on the extrinsics calibration. The resulting image can be seen in [](#fig:image_cropped). The calculations to find out where You have to cut the image are quite simple (note that it still bargains for homogeneous coordinates): 

\[
    p_{camera} = H^{-1}P_{world}
\] 

<center><img figure-id="fig:cropped" figure-caption="Cropped Image" src="image_cropped.png" style="width: 200px;"/></center>

**2.** Directly detecting the obstacles from this cropped input image failed for us due to the reasons descibed [above](#saviors-functionality-computer-vision). That is why the second step is to perform the transformation to the bird’s view perspective. For transforming the image we first use the corners of the cropped image and transorm it to the real world. Then we scale the real world coordinates to pixel coordinates, so that it will have a width of 640 pixels afterwards. For warping all of the pixel with low artifacts we then use the function **cv2.getPerspectiveTransform()**. The obtained image can be seen in [](#fig:bird_view). 

**3.** Then we transform the given RGB picture into the HSV colorspace and apply the yellow and orange filter. While a HSV image is hardly readable for humans, it is way better to filter for specific colors. The obtained pictures can be seen in [](#fig:yellow_filtered) and [](#fig:orange_filtered). The color filter operation is performed by the cv2 function **cv2.inRange(im_test, self.lower_yellow, self.upper_yellow)** where lower_yellow and upper_yellow are the thresholds for yellow in the HSV color space. 

**4.** Now there is the task of segmenting/isolating the objects which remained after the color filtering process. At the beginning of the project we therefore implemented our own segmentation algorithm which was however more inefficient and led to an overall computational load of 200% CPU usage and a maximum frequency of our whole module of about 0.5 Hz only. By using the scikit-image module which provides a very efficient [label function](http://scikit-image.org/docs/dev/api/skimage.measure.html#skimage.measure.label), the computational efficiency could be shrunk considerably to about 70% CPU usage and allows the whole module to run at up to 3 Hz. It is important to remember that in our implementation the segmentation process is the one which consumes the most power. The output after the segmentation is the one in [](#fig:segmented), where the different colors represent the different segmented objects.

**5.** After the segmentation, we analyse each of the objects alone. At first there is a general filter which ensures that we are neglecting all the objects which contain less than a user influenced threshold of pixels. Since as mentioned above, the homographies of all the users are different, the exact amount of pixels an object has to have to be considered is again scaled by the individual homography. This is followed by a more in detail analysis which is color dependent. On the one hand there is the challenge to detect the orange cones reliably. Speaking about cones, the only other object that might be erroneously detected as orange are the stop lines. Of course in general the goal should be to very reliably detect orange but as the light is about to change during the drive, we prepared to also detect the stop lies and being able to cope with them when they are erroneously detected. The other challenge in general was that all object that we have to detect can appear in all different orientations and that simply inferring the height and width of the segmented box, as we did it in the beginning, is obviously not a very good measure (in the bottom left picture e.g. the segmented box is a square while the cone itself is not quadratic at all) That is why it is best to use a rotation invariant feature to classify the segmented object. In our final implementation we came up using the two eigenvalues of the inertia tensor, which are obviously rotation invariant (when being ordered by their size). Now, speaking more specifically about the detection of cones, when extracting the cone from the upper image separately it is looking like in [](#fig:segmented_cone), while an erroneous detection of a stop line is looking like in [](#fig:segmented_stop_line).

<center><img figure-id="fig:segmented_cone" figure-caption="Segmented Cone" src="segmented_cone.png" style="width: 200px;"/></center>

<center><img figure-id="fig:segmented_stop_line" figure-caption="Segmented Stop Line" src="segmented_stop_line.png" style="width: 200px;"/></center>

Our filter criteria is now the ratio between the eigenvalues of the inertia tensor. This ratio is always by a factor of about 100 greater in case the object is a cone, compared to when we erroneously segment a red stop line. This criteria is very stable that is why there is no additional filtering needed to detect the cones.

If the segmented object is yellowish, things get a little more tricky as there are always many yellow object in the picture, namely the middle lines. Line elements can be again observed under every possible orientation, therefore the eigenvalues of the inertia tensor which are as mentioned above rotation invariant are again the way to go. In [](#fig:segmented_middle_line) you can see a segmented line element and in [](#fig:segmented_duck) again a segmented duckie. 

<center><img figure-id="fig:segmented_middle_line" figure-caption="Segmented Middle Line" src="segmented_middle_line.png" style="width: 200px;"/></center>

<center><img figure-id="fig:segmented_duck" figure-caption="Segmented Duckie" src="segmented_duck.png" style="width: 200px;"/></center>

As the labelled axis already reveal, they are of a different scale, but as we also got very small duckies, we had to choose a very small threshold. To detect the yellow duckies, the initial condition is that the first eigenvalue has to be greater than 20. This criteria alone however includes to sometimes erroneously detecting the lines as obstacles, that is why we implemented an additional tracking algorithm which works as follows:
If an object’s first eigenvalue is greater than 100 pixels and it is detected twice, meaning in two consecutive images there is a object at roughly the same place it is labelled as an obstacle. However, if an object is smaller or changed the size by more than 50% then a more restrictive criteria is enforced, stating that we must have tracked this object at least for 4 consecutive frames before being labelled as an obstacle. This criteria is working pretty well and a more thorough evaluation will be provided in the next section. In general those criteria help that the obstacles can be detected in any orientation. The only danger to the yellow detecting algorithm is motion blur, namely when the single lines are not separated but connected together by “blur”. 

**6.** After analysing each of the potential obstacle objects, we decide whether it is an obstacle or not. If so, we continue to steps **7.** and **8.**. 

**7.** Afterwards, we calculate the position and radius of all of the obstacles. After segmenting the object we calculate the 4 corners (which are connected in [](#fig:position_size) to form the green rectangle). For the position we then simply use the midpoint of the lower line (this point surely lies on the ground plane). For the radius we use the distance in the real world between this point and the lower right corner. This turned out to be a good approximation of the radius. For illustration you can have a look on [](#fig:position_size).

<center><img figure-id="fig:position_size" figure-caption="Position and Radius of the Obstacle" src="position_size.jpg" style="width: 200px;"/></center>

**8.** Towards the end of the project we came up with one additional last step based on the idea that only obstacles inside the white lane boundaries are of interest to us. That is why for each obstacle, we look whether there is something white in between us and the obstacle. In [](#fig:classification) you can see an example situation where the obstacle inside the lane is marked as dangerous (red) while the other one is marked as not of interest to us since it is outside the lane boundary (green) and in [](#fig:search_line) you see the search lines (yellow) along which we search for white elements.

<center><img figure-id="fig:classification" figure-caption="Classification if Object are Dangerous or not" src="classification.png" style="width: 200px;"/></center>

<center><img figure-id="fig:search_line" figure-caption="Search Lines whether something White is in between" src="search_line.jpg" style="width: 200px;"/></center>

**9.** As the last step of the detection pipeline we return a list of all obstacles including all the information via the posearray.

### Part 2: Avoidance in Real World - Functionality {#saviors-functionality-avoidance} 

The Avoidance deals with drawing the right conclusions from the received data. 

#### **Theoretical Description** {#saviors-avoidance-theoretical}

With the separation of the detection, an important part of the avoidance node is the interaction with the other work packages.
We determined the need of getting information about the remaining duckietown besides the detected obstacles. The obstacles need to be in relation to the track in order to assess whether we have to stop or to drive around obstacles. This is given because we are not supposed to leave the lane. Due to other teams already working on this, we deemed it best to not implement any further detections (lines, intersections etc.) in our visual perception pipeline. This saves similar algorithms being run twice on the processor. We decided to acquire the values of our current pose relative to the side lane, which is determined by the _devel-linedetection_ group.

The idea was to make the system highly flexible. The option to adapt to following situations was deemed desirable:

* Multiple obstacles. Different path planning in case of a possible avoidance might be required.
* Adapted behavior if the robot is at intersections.
* Collision avoidance dependent on the fleet status within the duckietown. Meaning if a duckiebot drives alone in a town it should have the option to avoid a collision by driving onto the opposite lane.

Obstacles sideways of the robot were expected to appear as the duckietowns tend to be flooded by duckies. Those detections on the side as well as potential false positive detections should not make the robot stop. To prevent that we intended on implementing a parametrized bounding box ahead of the robot. Only obstacles within that box would be considered. Depending on the certainty of the detections as well as the yaw-velocities the parametrization would be tuned.

The interface getting our computed desired values to impact on the car is handled by _devel-controllers_. We agreed on the usage of their custom message format, in which we send desired values for the lateral lane position and the longitudinal velocity. Our intention was to account for the delay of the physical system in the avoider node. Thus our planned trajectory will reach the offset earlier than the ideal-case trajectory would have to. Tuning ours as well as controllers’ parameters was planned to cooperatively do directly besides the robot.

Due to above mentioned interfaces and multiple levels of goals we were aiming for an architecture which allows gradual commissioning. The intent was to be able to go from basic to more advanced for us as well as for groups in upcoming years. Those should be able to extend our framework and not have to rebuild it.

The logic shown in [](#fig:avoidance_logic) displays one of the first stages in the commissioning. Key is the reaction to the number of detected obstacles. Later stages will not trigger an emergency stop in case of multiple obstacle detections within the bounding box.

<center><img figure-id="fig:avoidance_logic" figure-caption="Logic of one of the First Stages in Commissioning" src="avoidance_logic.png" style="width: 500px;"/></center>

Our biggest concern were the added inaccuracies until the planning of the trajectory. Those include:

* Inaccuracy of the currently determined pose
* Inaccuracy of the obstacle detection
* Inaccuracy of the effectively driven path aka. controller performance

To us the determination of the pose was expected to be the most critical. Our preliminary results of the obstacle detection seemed reasonably accurate. The controller could be tweaked that the robot would rather drive out of the track than into the obstacle. An inaccurate estimation of the pose would just widen the duckie artificially.

_Devel-controllers_ did not plan on being able to intentionally leave the lane. Meaning the space left to avoid an obstacle on the side of the lane is tight making above uncertainties more severe.

We evaluated the option to keep track of our position inside the map. Given a decent accuracy of said position we’d be able to create a map of the detected obstacles. Given multiple detections (also outside of the bounding box) we could achieve a further estimation of our pose relative to the obstacles. This essentially would mean creating a SLAM-algorithm with obstacles as landmarks. We declared as out of scope given the size of our team as well as the computational constraints.
The goal was to make use of a stable, continuous detection and in each frame react on it.

#### **Actual Implementation** {#saviors-avoidance-implementation}

**Interfaces**

One important part of the Software is the handling of the interfaces, mainly to _devel_controllers_. For further informations on this you can refer to the [Software Architecture Chapter](#saviors-software-architecture).

**Reaction**

The obstacle avoidance part of the problem is handled on an additional node, called the obstacle_avoidance_node.
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

If the transformation shows that an avoidance is possible we calculate the d_ref we need to achieve to avoid the obstacle. This is sent to the lane control node and then processed as new target distance to the middle of the lane. The lane control node uses this target and starts to correct the duckiebots position in the lane. With each new obstacle pose being generated this target is adapted so that the duckiebot eventually reaches target position. The slow transition movement allows us to avoid the obstacle even when it is not visible anymore shortly before the robot is at the same level as the obstacle.

At the current stage, the obstacle avoidance is not working due to very high inaccuracies in the estimation of $\theta$. The value shows inaccuracies with an amplitude of 10°, which leads to wrong calculations of the transformation and therefore to misjudgement of the d_ref. The high amplitude of these imprecisions could be transformed to a uncertainty factor of around 3 which means that each object is around 3 times its actual size which means that even a small obstacle on the side of the lane would not allow a safe avoidance to take place. For this stage to work, the estimation of $\theta$ would need significant improvement.

**Case 2: Emergency Stop**

Conditions for triggering an emergency stop:
* More than one obstacle in range
* Avoidance not possible because the obstacle is in the middle of the lane
* **Currently every obstacle detection in the bounding box triggers an emergency stop due to the above reasons**

If one of the above scenarios occurs an avoidance is not possible and the robot needs to be stopped. By setting the target speed to zero, the lane controller node stops the robot.
As soon as the situation is resolved by removing the obstacle which triggered the emergency stop, the robot can proceed with the lane following.

These tasks are then repeated at the frame rate of the obstacle detection array being sent. 

### Required Infrastructure - Visualizer {#saviors-visualizer}

Especially when dealing with a vision based obstacle detection algorithm it is very hard to infer what is going on. One has to also keep the visual outputs low, to consume as less computing power as possible, especially on the Raspberry Pi. This is why we decided to not implement one single obstacle detection node, but effectively two of them, together with some scripts which should help to tune the parameters offline and to infer the number of false positives, etc. The node which is designed to be run on the Raspberry Pi is our normal obstacle_detection_node. This should in general be run such that there is no visual output at all but that simply the PoseArray of obstacles is published through this node.
The other node, namely the obstacle_detection_node_visual node is designed to be run on your own laptop which is basically visualising the information given by the posearray. There are two visualisations available. On the one hand there is a marker visualisation in rviz which shows the position and size of the obstacles. In here all the dangerous obstacles which must be considered are shown in red, whereas the non critical (which we think that they are outside the lane boundaries) are marked in green. On the other hand there is also a visualisation available which shows the camera image together with bounding boxes around the detected obstacles.
Nevertheless, this online visualisation is still dependent on the connectivity and you can only hardly “freeze” single situations where our algorithm failed. That is why we also included some helpful scripts into our package. One script allows to thoroughly input many pictures and outputs them labelled together with the bounding boxes, while another one outputs all the intermediate steps of our filtering process which allows to fastly adapt e.g. the color thresholds which is in our opinion still the major reason for failure. More information on our created scripts can be found in our **Readme on GitHub**.

### Recorded Logs {#saviors-logs}

## Formal performance evaluation / Results {#saviors-final-formal}
<!--_Be rigorous!_
- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?-->

### Eval Interface and Computational Load {#saviors-eval-interface} 

In general as we are dealing with many color filters a reasonable color corrected image is the key to the good functioning of our whole module, but turned out to be the greatest challenge when it comes down to computational efficiency and performance. As described above we are really dependent on a color corrected image by the Anti Instagram module and desperately need their color corrected image. Throughout the whole project we planned to use their continuous anti_instagram node which is supposed to compute a color transformation in fixed intervals of time but when it came down we acutally had to change this for the follwing reason. The continouus anti-instagram node, running at an update interval of 10 seconds consumes a considerable amount of computing power, namely 80%. In addition to that, the image transformer node which is in fact transforming the whole image and currently running at 4 Hz needs another 74% of one kernel. If you now run those two algorithms combined with the lane-following demo which makes the vehicle move and our own code which needs an additional 75% of computing power, our safety critical module could only run at 1.5Hz and led to poor behaviour. Even if you increase the time interval in which the continouus anti-instagram node computes a new transformation there was no real improvement. That is why in our final setup we let the anti-instagram node once compute a reasonable transformation and then keep this one for the entire drive. Through this measure we were able to safe the 80% share entirely and this let our overall node run at about 3 Hz with introducing an additional maximal delay of about 0,3 seconds. Nevertheless we want to point out that all the infranstructure for using the continouus anti instagram node in the future is provided in our package.


### Eval Obstacle Detection {#saviors-eval-obst-detect}

In general, since our obstacle classification algorithm is based on the rotational invariant feature of the eigenvalues of the inertia tensor it is completely invariant to the current orientation of the duckiebot and its position with respect to the lanes. 

<center><img figure-id="fig:detector_performance" figure-caption="Obstacle Detector Performance during drive" src="sample_detector.jpg" style="width: 500px;"/></center>

To rigorously evaluating our detection algorithm, we started off with evaluating **static scenes**, meaning the Duckiebot is standing still and not moving. Our algorithm performed extremely well in those static situations. You can place an arbitrary amount of obstacles, where the orientation of the respective obstacles does not matter at all, in front of the Duckiebot. In those situations and also combining them with changing the relative orientation of the Duckiebot itself, we achieved a false positive percentage of below 1% and we label all of the obstacles correctly. The only static setup which is sometimes problematic is when we place the smallest duckies very close in front of our vehicle (below 4 centimeters), without approaching them, then we sometimes cannot detect them. However this problem is mostly avoided during the dynamic driving, since we anyways want to stop earlier than 4 centimeters in front of potential obstacles. We are very happy with this static behaviour as in the worst case, if during the dynamic drive something goes wrong, you can still simply stop and rely upon the fact that the static performance is very good and then continue your drive. HIER EVENTTUELL LINK ZU DER BAG ODER SO,...

All in all, our algorithm is extremely reliable when it comes to evaluating static situations, meaning we come to a stop during the drive. This also implies that most of the misclassification errors are due to the effect of motion blur, assuming a stable color transformation provided by the anti instagram module. E.g. in [](#fig:motion_blur_error) two line segments in the background “blurred” together for two consecutive frames resulting in being labelled as an obstacle.  

<center><img figure-id="fig:motion_blur_error" figure-caption="Obstacle Detector Error due to motion blur" src="motion_blur_error.jpg" style="width: 500px;"/></center>

But now speaking more about of numbers, we took 2 duckiebots at a gain of around 0.6 and performed two drives each of about 4 minutes at different days, so also at different lights and the results are the following:
Evaluating each picture which will be given to the algorithm, we found out that on average, we detect 97% of all the yellow duckies in each picture. In terms of cones we detect about 96% of all cones in the evaluated frames. We consider these to be very good results as we have a very low rate of missclassifications (about 3%). Missclassifications means the ratio between objects which are inside the lane boundaries and erroneously considered as obstacles divided by the number of totally seen correct obstacles.

When it comes to evaluating the performance of our obstacle classification with respect to dangerous or not dangerous our performance is not as good as the detection itself, but we did also not put the same effort into it. We are especially encountering problems when there is direct illumination on the yellow lines which are very reflective and therefore appear whitish. An approach to resolve this problem would be to label every obstacle in a vicinity as dangerous,... Here the evaluation of the two rides revealed that this classification with respect to an obstacle being dangerous or non dangerous has an error rate of about 10%, but sometimes fails in crucial situations as shown in [](#fig:detector_missclass).

<center><img figure-id="fig:detector_missclass" figure-caption="Obstacle Detector Classification Error" src="classification_error.jpg" style="width: 500px;"/></center>

In terms of automatically evaluating our software there are only a few steps missing. As you can see in our code description in github there are complete scripts available which extract single pictures (including the application of Anti Instagram on them) and other scripts which perform the classification in those pictures. The only step for which was not enough time left is an automatic detection whether the drawn box is correct and in fact around an object which is considered to be an obstacle or not,...

## Future avenues of development {#saviors-final-next-steps}
<!--_Is there something you think still needs to be done or could be improved? List it here, and be specific!_-->

Furthermore there is still room for improvements in our opinion. E.g. in the future one might use machine learning to being able to detect duckies in different colors. Therefore, definitely machine learning is needed since we might want to set up a neural network. Another idea might be to use this bird view approach to also perform lane detection from this view and to not waste the energy put in the transformation to only extract obstacles from it.  Also another idea would be to instead of always analysing the whole image it would be reasonable, under the assumption of having a good estimate of movement to check extensively for obstacles only in the new regions which are added to the picture. Furthermore as long as the color correction of anti_instagram is linear it might be reasonable to just adapt the filter values than to subscribe to a fully transformed image, but as this might change we prefer the way we implemented it. It will be also necessary in the future to add a Duckiebot detection algorithm into the obstacle detection to allow fancier moves like safely crossing the middle line. 

Towards the really end of our project we also tried to apply this approach which worked really well for the white and yellow filtering but unfortunately not for the orange (cone) filter. (BRANCH: devel-saviors-ai-tryout2) We think that if in the future one will stick to the hardware that this might be a very interesting approach since the linear trafo provided by the anti instagram team is usually sufficient for correcting the colors and this effort could save a whole publisher and subscriber and it is obvious that it is by far more efficient to transform a few filter values once than to transform every pixel of every incoming pixel. Furthermore we think that this approach might be useful for the lane detection or other vision based algorithms which are based on the concept of filtering colors.

In terms of avoidance there would be possibilities to handle the high incurracies of the pose estimation by relying on the lane controller to not leave the lane and just use a kind of closed loop control to avoid the obstacle (Use the new position of the detected obstacle in each frame to securely avoid the duckie). Furthermore the infrastructure is in place to include new scenarios like obstacles on intersection or multiple detected obstacles inside the bounding box. If multiple obstacles are in proximity, a more sophisticated trajectory generation could be put in place to avoid these obstacles in a safe and optimal way. 
## Theory Chapter {#saviors-theory-chapter}

### Required Matrix Transformations {#saviors-transformations}

### Introduction to Computer Vision
* Shortly Introduce Linear Camera Model and Matrix writing
* Point out that with mono cam no depth information is possible at all
* Propably shortly give intuition how we humans are able to extract scale (Var 1: Stereo and Triangulation; Var 2: guessing the right scale by having some reference objects)
* Introduce our calibration technique -> if all objects are in ground plane -> we have a scale!

### Obstacle Detection exploiting Inverse Perspective Mapping
* exploiting the fact that we have scale -> show how you can transform one image plane into another one using mathematical equations!!!
* give mathematical background behind transforming one image plane into another!
* show why this inverse perspective lets you evaluate the scale and easily extract obstacles which "come out of the ground"

### HSV Color Space {#saviors-HSV}

While a HSV image is hardly readable for humans, it is way better to filter for specific colors, since effectively in the RGB color space all “the three channels are effectively correlated by the amount of light hitting the surface,” so the color and light properties are not separated ( https://www.learnopencv.com/color-spaces-in-opencv-cpp-python/ ). However, in the HSV space, there is only one channel (the H channel) to describe the color. The S represents the saturation and H the intensity. This is the reason why it is super useful for specific color filtering.

<center><img figure-id="fig:color_comparison" figure-caption="Comparison between the two Colors Spaces" src="rgb_hsv.jpg" style="width: 200px;"/></center>
--- (https://image.slidesharecdn.com/01presentationhuehistograms-150707215651-lva1-app6892/95/about-perception-and-hue-histograms-in-hsv-space-5-638.jpg?cb=1436307525)




https://www.learnopencv.com/color-spaces-in-opencv-cpp-python/
https://stackoverflow.com/questions/22588146/tracking-white-color-using-python-opencv
https://docs.opencv.org/3.2.0/df/d9d/tutorial_py_colorspaces.html
