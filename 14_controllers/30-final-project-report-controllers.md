#  Group name: final report {#controllers-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

<!--
_The objective of this report is to bring justice to your hard work during the semester and make so that future generations of Duckietown students may take full advantage of it. Some of the sections of this report are repetitions from the preliminary and intermediate design document (PDD, IDD respectively)._
-->

## The final result {#controllers-final-result}

<!--
_Let's start from a teaser._
* Post a video of your best results (e.g., your demo video)
-->

<div figure-id="fig:demo-video">
    <figcaption>The Controllers Demo Video</figcaption>
    <dtvideo src="vimeo:250511342"/>
</div>
See the [operation manual](#demo-lane-following) to reproduce these results.


## Mission and Scope {#controllers-final-scope}

Motto: IMPERIUM ET POTESTAS EST <br/> (With control comes power)

Our **Mission** was to make lane following more robust to model assumptions and Duckietown geometric specification violations and provide control for a different reference.


### Motivation {#controllers-final-result-motivation}

In Duckietown, Duckiebots are cruising on the streets and also Duckies are sitting on the sidewalk waiting for a Duckiebot to pick them up. To ensure a baseline safety of the Duckiebots and the Duckies, we have to make sure the Duckiebots are able to follow the lane (or a path on intersections and in parking lots) and stop in front of red lines. For example, the Duckiebot is driving on the right lane. It should never cross the centerline to avoid any collisions with an oncoming Duckiebot. 

The overall goal of our project is to stay in the lane while driving and stop in front of a red line. Due to the tight time plan, we focused on improving the existing code and benchmarking the tasks. In order to let the Duckiebot drive to a given point, the robot has to know where it is in the lane, calculate the error and define a control action to reach the target. To retrieve the location and orientation information, a pose estimator is implemented. The estimator receives line segments from the image pipeline with information about line tape colour (white, yellow, red) ([](#fig:segment_foto)) and whether the segment is on the left or right edge of the line tape. Using the given information, we determine if the Duckiebot is inside or outside the lane, how far it is from the middle of the lane and at what angle it stands. The relative location to the middle of the lane and the orientation of the Duckiebot are passed on to the controller. To correct the error, the controller calculates the desired velocity and heading of the Duckiebot using the inputs and controller parameters. The importance of our project in the framework “Duckietown” was obvious, as it contains the fundamental functionality of autonomous driving. Furthermore, many other projects relied on our project’s functionality such as obstacle avoidance, intersection navigation or parking of a Duckiebot. We had to ensure that our part is robust and reliable.




<center><img figure-id="fig:segment_foto" figure-caption="Image with Line Segments, $d_{err}$ and $\phi_{err}$ displayed." src="segment_foto.png" alt="Image with Line Segments" style="width: 300px;"/></center>

### Existing solution {#controllers-final-literature}

<!--
- Was there a baseline implementation in Duckietown which you improved upon, or did you implemented from scratch? Describe the "prior work"
-->
<center><img figure-id="fig:curve_plot" figure-caption="Pose of Duckiebot in a curve element." src="curve_plot.png" alt="Curve plot" style="width: 200px;"/></center>

From last year’s project, the baseline implementation of a pose estimator and a controller were provided to us for further improvement. The prior pose estimator was designed to deliver the pose for a Duckiebot on straight lanes only. If the Duckiebot was in or before a curve and in the middle of the lane, the estimated pose showed an offset **d**, see definition of **d** in figure below. The existing controller worked ok on straight lines. Due to the inputs from the pose estimator to the controller, the Duckiebot overshot in the curves and crossed the left/right line during or after the curve.

<!--
<insert video> see folder: 21_Media: Final Report > 02_controller_special_curve…
No Sound Video: https://drive.google.com/file/d/1y9L4_-VFRy9ay0EB-X7cyJkQ33ORXL49/view?usp=sharing
-->

<div figure-id="fig:demo-video">
    <figcaption>Old vs. new controller</figcaption>
    <dtvideo src="vimeo:256663571"/>
</div>

### Opportunity {#controllers-final-opportunity}

<!--
- What didn't work out with the existing solution? Why did it need improvement?
-->

In the **previous** implementation, the lane following was not guaranteed on curved lane segments, because the Duckiebot often left the lane during or after the curve. Although the Duckiebot sometimes returned to the right lane after leaving it and continued following the lane, robust lane following was not provided. On straight lanes, the Duckiebot often drove with a large static offset. The previously implemented pose estimator and controller left room for improvement.

Further, the previous work wasn’t benchmarked for robustness nor performance, so we defined various tests to benchmark the previous and our solution. During the project, we continuously tested our code with the entire lane following pipeline for best practice and compared our implemented solution to the existing one to record the improvement.  

Our **Scope** was first of all to enable controlled autonomous driving of the Duckiebot on straight lane segments and curved lane segments which are in compliance with the geometry defined in [Duckietown Appearance Specifications](#duckietown-specs). Further, we wanted to increase the robustness to arbitrary geometry of lane width or curvature of the lane to ensure the autonomous driving of the Duckiebot in an individual Duckietown setup. 
We also tackled the detection and stopping at red (stop) lines. With the previous implementation of the task, the Duckiebot stopped rather at random points in front of the red line than in the middle of the lane and in a predefined range. 
As the Duckietown framework is a complex system involving various functionalities such as obstacle avoidance and intersection, our lane following pipeline provides the basic function for those functionalities and it has to be able to interact with the modules of other teams. Hence, it was also our duty to design an interface which can receive and apply information from other modules.  For example, our controller can take reference d from obstacle avoidance, intersection crossing and parking (but for intersections and parking, we additionally need the pose estimation and a curvature from the navigators and parking team respectively).

Out of scope was:

* Pose estimation and curvature on Intersections (plus navigation / coordination)
* Model of Duckiebot and uncertainty quantification of parameters (System Identification)
* Object avoidance involving going to the left lane
* Extraction and classification of edges from images (anti-instagram)
* Any hardware design
* Controller for Custom maneuvers (e.g. Parking, Special intersection control)
* Robustness to non existing line

### Preliminaries (optional) {#controllers-final-preliminaries}

<!--
- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.

Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)
-->


## Definition of the problem {#controllers-final-problem-def}

<!--
_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical definition of the problem you tackled. You can use part of the preliminary design document to fill this section._
-->

Our final objective is to keep the Duckiebots at a given distance **d** from the center of the lane, on straight and curved roads, under bounded variations of the city geometric specifications. 

The project was on the bottom line, taking the line segments which gave information about the line colour and the segment positions to estimate the Duckiebot’s pose and return a command for the motors to steer the robot to the center of the lane. After roughly analysing the existing solution, we divided the work load into two topics **pose estimation** and **controller** to enable parallel dealing with the problems in the short period of time. 

In our [Preliminary Design Document](#controllers-pdd) and [Intermediate Report](#controllers-int-report), we have listed all variables and their definitions, as well as all system interfaces with other groups and assumptions we made. Due to limitation of time, some integrations with other teams are not yet activated but they are already prepared in the code (some of it commented out), see this repository branch [devel-controllers-external](https://github.com/duckietown/Software/tree/devel-controllers-externaluse).
TODO insert link to description or link to devel-controllers-external use?
 
### Pose Estimation

Starting with the image taken by a monocular camera, we assume that ‘Anti-Instagram‘ compensates for color changes from different ambient light conditions and the detected segments of the line edges are always in the corresponding colour (yellow, white, red) to the tape and point to the correct direction (to the Duckiebot = left edge, away from the Duckiebot = right edge), see ([](#fig:segment_foto)). The detected line segments are passed on in a list to the ‘Lane Filter‘, the Duckiebot estimates the distance $d_{est}$ from center of the lane and heading $\theta_{est}$ with respect to the center of the lane.  

To improve curve following, our main goal was to decrease the overshoot after a turn. A main reason for the overshoot was that the previous implementation of the pose estimator was designed for straight lanes only. We planned to predict if a curve is upcoming and in which direction the turn will be. This curvature estimation could then be used as an input for a feedforward part of the controller. This would help for a smoother transition in curves.

We planned on testing two different ideas for curvature estimation. One based on the distribution of segments to different domains with different ranges. The second one based on the Discrete Fourier Transform (DFT).

TODO: Image of pose estimation/ vote generation

Our general goal was to improve the accuracy of estimated pose in regard to reference pose and increase the computational efficiency. Also robustness regarding slight changes in width of the lane, width of the lines and curvature should be achieved.

### Controller
<center><img figure-id="fig:coordinates" figure-caption="Definition of used coordinates." src="coordinates.png" alt="Definition of used coordinates." style="width: 300px;"/></center>

In the existing implementation, the controller has taken the output of the estimator as input and calculated the motor command, velocity **v** and angular velocity **$\omega$**, with hardcoded parameters. Running the current lane following demo, we determined the weaknesses of the controller’s performance. Since the existing controller only had a P-part, we decided to implement following parts and benchmark its performance.

* Integrator for both **$\theta$** and **d**
    - First approach: No saturation for integrator
        * Problem: Very strong oscillation
    - Second approach: Saturation for integrator
        *Performance improved a lot. Oscillation is reduced
    - Third approach: Saturation of integrator and reset of integrator whenever zero error is reached
        * Performance improved again.
    - Fourth approach: Added true integration with correct timestep instead of simply summing up the error and neglecting the timestep
        *This is the correct approach since the time step needs to be taken into account.
* Feedforward for driving on a curved lane
    - We take the current curvature as an Input for the feedforward part
        *The feedforward part is very much dependent on the correct curvature estimation. If curvature was estimated correctly, feedforward worked well.


In some cases, the controller won’t need the real time pose input of the pose estimator but a given path from other teams for example during intersection navigation. For this task, we had to define a more general ‘Lane Pose‘ message, communicate and coordinate the integration with other teams and subscribe to their nodes, see [Subscribed topics by Lane Controller Node](#controllers-int-report). To make a decision about the input source, we have to subscribe to flags passed by the other teams, see [Flags received by other modules](#The Controllers: Intermediate Report) and create a prioritization logic, see .... 

Regarding performance, the Duckiebot should be left with a small steady state error within a tile and never leave the lane given the [Duckietown appearance specifications](#duckietown-specs). 


<!--
Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal
-->


## Contribution / Added functionality {#controllers-final-contribution}

### Curvature Estimation
#### Curvature estimation using multiple domains

The idea of our curvature estimation approach is to split the domain in front of the Duckiebot into multiple range areas. A version using three areas is shown in [](#fig:curvature_estim).

<center><img figure-id="fig:curvature_estim" figure-caption="Curvature estimation using the segments in different domains." src="curvature_estim.svg" alt="Curvature estimation" style="width: 300px;"/></center>



In each of the range areas the road can roughly be assumed as a straight lane. But for every area further away from the bot this straight lane fit is tilted more towards the left. For each area, only those segments with center point is inside the area are considered. Using these segments for each area we run the standard estimation and thereafter we get for each area a **d** and a **$\theta$** value. Now we can compare those results with the **d** and a **$\theta$** value from the estimation of the position. 


The expected results are shown in [](#fig:curvature_estimation_expected) where the left most points in each graph represent the actual position estimation and the further three point represent the estimations of the three different range areas. As a leftover of the existing code where this was already the case, **$\phi$** and **$\theta$** are still used as synonyms within the code. Due to limited time, it did not make it to our highest priority at any time within our project, to merge those names. This should be done in further work.

<center><img figure-id="fig:curvature_estimation_expected" figure-caption="Expected results of d and $\phi$ values for straight lane, left curve and right curve." src="curvature_estimation_expected.jpg" alt="Expected results of curvature estimation" style="width: 300px;"/></center>


Unfortunately, the measurements for the higher ranges are very noisy as there are only a few line segments detected at further distance and therefore the signal to noise ratio is very bad. To get rid of outliers we decided to add a median filter over values of the ranges. Additionally we saved this median **d** and **$\phi$** values over time for the last five time steps and again took the median value of it. Then we checked if it is above or below the value of the closest range.

Another possibility would be to fit a line through the data points and decide on the lane type based on the slope of the line. We decided to go for the above mentioned method because we wanted to keep the computational complexity as low as possible.

By testing we found good results for the cutoffs shown in [](#tab:curvature_cutoffs) where $d_{median}$ and $\phi_{median}$ represent the median over 5 time steps of the values resulting from the range area and $d_{est}$ and $\phi_{est}$ represent the actual pose estimate.

<col3 align='center' style="text-align:left" id='curvature_cutoffs' figure-id="tab:curvature_cutoffs" figure-caption="Cut offs for the decision on the curvature type.">
   <span> </span>     <span>$d_{median} - d_{est}$</span>    <span>$\phi_{median} - \phi_{est}$</span>
   <span>left curve</span>    <span>$<-0.3$</span>    <span>$>0.05$</span>
   <span>right curve</span>    <span>$> 0.2$</span>   <span>$< -0.02$</span>
</col3>


#### Curvature estimation using Discrete Fourier Transform

By generating a discrete binary image from the segments projected to the ground and applying the discrete fourier transform to this image, the curvature of the road in front of the Duckiebot can be detected. Fourier transforms of such binary images are shown in [](#fig:curvature_estimation_fourier). By using the right fourier features straight lanes, right and left curves could be detected thanks to their differing fourier transform.

This method has been implemented successfully by jukindle but the problem was first of all choosing the right resolution for the segment images and additionally the method introduced a delay of about 0.2 seconds, which again lowered the lane following performance of the Duckiebot. 

<center><img figure-id="fig:curvature_estimation_fourier" figure-caption="Discrete Fourier Transform (DFT) of a street image in ground frame." src="curvature_estimation_dft.png" alt="Discrete Fourier Transform of street image" style="width: 300px;"/></center>



### Benchmark 

To benchmark the state zero at the beginning of the project and our final implementation and compare them, we implemented a benchmark package. This package contains the benchmark code used for the Controllers project. It basically takes one or more rosbags in a specific folder and evaluates the run of the corresponding Duckiebot for **$d_{ref}$** and **$\phi_{ref}$** and plots them into a diagram.

Additionally if the rosbag does not contain any pose information, it takes the pictures and calculates the transformation and line segments itself. It does also plot the values onto the pictures, so those pictures can be combined to a video.

#### Output

Output diagram should look like the one shown in [](#fig:ducktaped_ETHZ_2018-01-12-14-24-51).

<center><img figure-id="fig:ducktaped_ETHZ_2018-01-12-14-24-51" figure-caption="Output diagram of benchmark code." src="ducktaped_ETHZ_2018-01-12-14-24-51.png" alt="Output diagram" style="width: 400px;"/></center>


Output data should look like the one shown in [](#fig:ducktaped_ETHZ_2018-01-12-14-24-51_eval).

<center><img figure-id="fig:ducktaped_ETHZ_2018-01-12-14-24-51_eval" figure-caption="Output data of benchmark code." src="ducktaped_ETHZ_2018-01-12-14-24-51_eval.png" alt="Output results" style="width: 300px;"/></center>



An example of a processed frame is shown in [](#fig:segment_foto).


#### How to run

You need to navigate to the folder that contains the benchmark code. In this case:

    $ /catkin_ws/src/10-lane-control/benchmark/

Then execute the following code:

    $ python benchmark path/to/folder_containing_rosbags

You can specify the path to a single rosbag or you can specify the general folder containing several rosbags and it will process all those which have not been processed yet and save the output into a folder __output__ unless otherwise defined.
To get help on the flags type:

    $ python benchmark --help

There are the following flags:

     --output     Name of the output folder (Default: 'output')
     --save_images     Extract and safe all frames from the rosbag (Default: False)
     --preparer    Define other preparer for image pipeline (Default: prep_120_40)
     --fast        Do not process image frames (Default: True)


### Controller

In the controller, a feedforward part was added to figuratively speaking straighten the lane and ease the work of the controller. Therefore, the feedforward part takes the estimated curvature **$c_{est}$** and reference velocity **$v_{ref}$** as inputs and returns the needed yaw rate **$omega$**. Which is then added to the output of the controller.

To reduce static offset, integral parts were implemented for both **$d$** and **$\theta$**. To prevent the integral parts from diverging, an Anti Reset Windup was implemented. Therefore, whenever actuator limits were reached (meaning the motors were not sent lower values than would be necessary to reach the controller outputs, because of certain limitations within the software), the integral steps at the corresponding time step were not added to the integrator.

In curves the integrator values accumulate rapidly and lead to an overshoot after the curve. A possible approach would be to turn off the integrator in curves, but in consequence the curvature estimation would need to be used and in addition need to be robust. Or if the feedforward part could be fully used (while also needing a robust and low-latency curvature estimation), the problem might be diminished. In the current state, the integrator is reset to zero whenever it is at or crosses the zone (since the pose is estimated in) of zero error. In addition the integrator was also reset to zero, whenever the velocity sent to the motors was zero.

The resulting parameter, the proportional gains and the integrator gains of both **$d$** and **$\theta$**,s were tuned. First with … initial values were approximated. Then the parameters were tuned by hand using …, therefore the unstable state and the approximate boundary thereof was looked for for each parameter with all the other parameters in a stable state. This was repeated multiple times with ever more aggressive controller behavior and an optimum was found.
 

Only obstacle avoidance is integrated 

TODO

<!--
Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them
-->

<!--
_Feel free to create subsections when useful to ease the flow_
-->

## Formal performance evaluation / Results {#controllers-final-formal}

<!--
_Be rigorous!_
-->

<!--
- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. 
-->

<!--
Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?
-->

We evaluated the improvement of performance with help of several evaluations. The evaluation procedure are defined in our [Indermediate Report](#controllers-int-report). The main benchmark feature was the average deviation from tracking reference during a run (distance to middle lane) and the standard deviation of the same value. We also benchmarked the deviation from the heading angle aswell, but since the bot is mainly controlled according to the deviation of the tracking distance it was the main feature to lead our development.
Benchmarking in general occurred by letting the Duckiebot run a specific experiment and recording a rosbag. We wrote a distinct benchmarking application, that analyzes the rosbag containing the recorded values and creates plots with the extracted information about tracking distance and heading angle over the run. 

Furthermore, we assessed the performance of the Duckiebots in the following dimensions:

* **Estimator**:
    - Static lane pose estimation benchmark
    - Static curve pose estimation benchmark
    - Image resolution benchmark
    - Segment interpolation benchmark
    - Curvature estimation benchmark
* **Controller**:
    - Stop at red line benchmark
    - Controller benchmark
    - Non-conforming curve benchmark
### Performance Evaluation of Estimator

#### Static lane pose estimation benchmark

In the static lane pose estimation, we put the Duckiebot on predefined poses and checked how well the pose estimator performs. In this section, the Duckiebot was placed on a straight lane segment with different distances from the middle of the lane and different heading angles. The results can be seen in the following graphs: 

<center><img figure-id="fig:perf_static_straight_d" figure-caption="Old Lane Estimator Performance of estimating $d$ on straight lane." src="perf_static_straight_d.png" alt="Old Lane Estimator Performance on straight lane" style="width: 400px;"/></center>




As one can see from [](#fig:perf_static_straight_d), the estimated distance from the middle of the lane and the actual value correspond very good in most experiments. There is only one case where the actual deviation was $-1 cm$ and measured was $4 cm$. Note that the histogram resolution used to determine the pose is $1 cm$.

<center><img figure-id="fig:perf_static_straight_phi" figure-caption="Old Lane Estimator Performance of estimating $\phi$ on straight lane." src="perf_static_straight_phi.png" alt="Old Lane Estimator Performance on straight lane" style="width: 400px;"/></center>


[](#fig:perf_static_straight_phi) shows a similar picture for the heading angle estimation from the segments. Deviation from actual values vary from $1$ to $12^{\circ}$, whereas the Duckiebot performed better when being rotated to the left. Note that the histogram resolution to determine the heading angle is $3^{\circ}$ or $0.15 rad$.

<center><img figure-id="fig:perf_static_straight_delta" figure-caption="Differences of $d_{actual}$ and $d_{measured}$ and $\phi_{actual}$ and $\phi_{measured}$ respectively." src="perf_static_straight_delta.png" alt="Old Lane Estimator Performance on straight lane" style="width: 400px;"/></center>


If we look at the overall deviations in all experiments shown in [](#fig:perf_static_straight_delta), we can see that the pose estimator performs fairly well, and it is possible to control on the deviation of the distance. The heading angle shows more error. The average deviation from the actual tracking distance in all experiments accounts to $1.6 cm$ and the average deviation from the actual heading angle in all experiments is $7.2^{\circ}$.

#### Static curve pose estimation benchmark

In the static curve pose estimation we put the Duckiebot on predefined poses and checked how well the pose estimator performs. In this section the Duckiebot was placed on left curve with different distances from the middle of the lane and different heading angles. The results can be seen in the following graphs: 

<center><img figure-id="fig:perf_static_curve_d" figure-caption="Old Lane Estimator Performance of estimating $d$ on curved lane." src="perf_static_curve_d.png" alt="Old Lane Estimator Performance on curved lane" style="width: 400px;"/></center>



As one can see from [](#fig:perf_static_curve_d), the estimated distance from the middle lane and the actual value correspond partially to the actual values. Especially for the distance of $10 cm$ to the right of the middle of the lane in a left curve the estimator has problems to detect the correct deviation. This is due to the low number of segments and the fact that the pose estimator is actually only constructed to estimate the pose on a straight lane. Also, there is quite some noise which leads to wrong interpretation of the distance, even when the duckiebot is perfectly situated in the middle of the lane. For some experiments there is no pose estimation due to too much noise in the segment list. Note that the histogram resolution used to determine the pose is $1 cm$.

<center><img figure-id="fig:perf_static_curve_phi" figure-caption="Old Lane Estimator Performance of estimating $\phi$ on a curved lane." src="perf_static_curve_phi.png" alt="Old Lane Estimator Performance on curved lane" style="width: 400px;"/></center>



Whereas the estimator is still able to estimate $d$ quite well on a left curve, for the heading angle most of the values are completely off as can be seen in [](#fig:perf_static_curve_phi). This means the heading angle prediction is not reliable on curved lanes. Note that the histogram resolution to determine the heading angle is $3^{\circ}$ or $0.15 rad$.

<center><img figure-id="fig:perf_static_curve_delta" figure-caption="Differences of $d_{actual}$ and $d_{measured}$ and $\phi_{actual}$ and $\phi_{measured}$ respectively on a curved lane." src="perf_static_curve_delta.png" alt="Old Lane Estimator Performance on curved lane" style="width: 400px;"/></center>


If we look at the overall deviations in all experiments shown in [](#fig:perf_static_curve_delta), we can see that the pose estimator performs ok in determining the distance from the middle of the lane in a curved section. The values from the heading angle are unlikely correct and therefore should not be used as control input. The average deviation from the actual tracking distance in all experiments accounts to $1.8 cm$ and the average deviation from the actual heading angle in all experiments is $21.6^{\circ}$.

#### Image resolution benchmark

Since the image resolution has an impact on the number of segments being visible to the Duckiebot and the image processing latency time, we benchmarked the impact on the performance. We tested different image resolutions, top cut off amounts and changed the histogram size to evaluate how it influences the control performance.

<center><img figure-id="fig:perf_image_res_yaf" figure-caption="Measured $d_{mean}$ and $\phi_{mean}$ values for different image resolutions for Duckiebot 'yaf'." src="perf_image_res_yaf.png" alt="Image Resolution performance yaf" style="width: 400px;"/></center>



As one can see in [](#fig:perf_image_res_yaf) the performance of the Duckiebot measured as the mean deviation from the reference trajectory (which is usually $0 cm$) is getting worse the higher the resolution. There are outliers though, since the highest resolution being tested shows better performance than the resolution just one step smaller. The best performance is achieved with slightly higher resolution at 150x200 pixels. To validate these results, we tested it on another Duckiebot aswell.

<center><img figure-id="fig:perf_image_res_a313" figure-caption="Measured $d_{mean}$ and $\phi_{mean}$ values for different image resolutions for Duckiebot 'a313'." src="perf_image_res_a313.png" alt="Image Resolution performance a313" style="width: 400px;"/></center>



We see that the results shown in [](#fig:perf_image_res_yaf) and [](#fig:perf_image_res_a313) are not congruent. We think that this has to do with the fact that each Duckiebot is slightly different and also has different latencies. 

The worse performance for higher resolutions can be explained with the change in processing time of the images. Although there are more line segments, which means more precise information about our pose, the processing time increases, and thus this adds latency and affects the whole system performance. The Duckiebot reacts slower to offsets of its pose. [](#fig:perf_image_res_time) shows the segment processing time and number of segments for different image resolutions.

<center><img figure-id="fig:perf_image_res_time" figure-caption="Segment processing time and median of number of segments for different image resolutions." src="perf_image_res_time.png" alt="Image Resolution segment processing time" style="width: 400px;"/></center>


Increasing the top cutoff value means, that from the input image more of the top part is cut away to reduce visual clutter from the image background. At the same time this also decrease the number of pixels being processed and thus lowers the mean latency as well.

<center><img figure-id="fig:perf_image_res_top_d" figure-caption="Performance of 'a313' for different top cutoffs in pixels." src="perf_image_res_top_d.png" alt="Top cutoff performance test" style="width: 400px;"/></center>



We run a benchmark to evaluate the influence of the top cutoff on the performance. The test has been performed with an image resolution of 120x160 pixels. The results are shown in [](#fig:perf_image_res_top_d). 40 pixels is the standard top cutoff values. This means the upper 40 pixels are cut away from each image. While increasing the top cutoff amount, the $d_{mean}$ decreases slightly while $\phi_{mean}$ increases slightly. We don’t see big changes in performance until the top cutoff gets quite big. At this point the Duckiebot does not see enough to control according to the actual pose situation. 

<center><img figure-id="fig:perf_image_res_top_time" figure-caption="Segment process time and number of segments for different top cutoffs in pixels." src="perf_image_res_top_time.png" alt="Top cutoff performance test" style="width: 400px;"/></center>



As we can see in [](#fig:perf_image_res_top_time), the segment process time and therefore the latency decreases proportionally to the number of segments. This graph also explains the reduced performance in [](#fig:perf_image_res_top_d) since with under 25 segments it is hard to get an accurate pose estimation. In this case a higher top cutoff lowers the performance and at the same time the latency. So, we might see an increase in performance if we combine higher top cutoff with higher resolution, since there the increased latency was an issue.

<center><img figure-id="fig:perf_image_res_hist" figure-caption="Performance for different resolutions of $d$ in histogram in $cm$." src="perf_image_res_hist.png" alt="Histogram performance test" style="width: 400px;"/></center>



We also tested the influence of the histogram size for the generation of the votes. The results are shown in [](#fig:perf_image_res_hist). Making the vote histogram cell size smaller increases the accuracy of the pose estimation. At the same time more segments are needed to get a precise estimate and reduce the influence of noise. We see that the performance is going down for a higher histogram resolution. At the same time [](#fig:perf_image_res_hist_time) shows that the segment processing time stays more or less constant for different histogram resolutions. This actually shows, that the decrease in performance results from the missing of a distinct pose for higher resolutions.

<center><img figure-id="fig:perf_image_res_hist_time" figure-caption="Segment processing time and number of segments for different resolutions of $d$ in histogram in $cm$." src="perf_image_res_hist_time.png" alt="Histogram performance test" style="width: 400px;"/></center>



<col6 align='center' style="text-align:left" id='image_res_top' figure-id="tab:image_res_top" figure-caption="Combining image resolution and top cutoff.">
   <span>Resolution</span>      <span>Top Cutoff</span>     <span>$t_{latency} [s]$</span>       <span>$d_{mean} [cm]$</span>    <span>$\phi_{mean} [rad]$</span>    <span>$n_{segments} [-]$</span>
   <span>120x160</span>      <span>40</span>     <span>$0.019$</span>       <span>$-1.802$</span>    <span>$-0.025$</span>    <span>$39$</span>
   <span>150x200</span>      <span>75</span>     <span>$0.012$</span>       <span>$-0.011$</span>    <span>$-0.004$</span>    <span>$23$</span>
</col6>

As we can see from [](#tab:image_res_top), the configuration with resolution 150x200 and top cutoff 75 can improve the lane control performance compared to the standard configuration with resolution 120x160 and top cutoff 40 without changing the lane controller itself or the pose estimator. Note that all the results from this section have been tested with the improved lane controller.

#### Segment interpolation benchmark

Another approach to improve the pose estimator is to increase the amount of line segments without increasing the image resolution. Here we take each line segment and divide it into smaller pieces of which each has a vote on the belief image. Good line segments cast more votes to the same pose estimate, while bad segments (e.g. which are further away or outliers) have less weight on casting wrong results. Think of it as a filter to improve quality of the lane pose estimate.

<center><img figure-id="fig:perf_segment_d" figure-caption="Performance for different segment interpolations." src="perf_segment_d.png" alt="Performance evaluation of segment interpolation" style="width: 400px;"/></center>

 

As we can see in [](#fig:perf_segment_d), we tested up to interpolating a line segment 5 times. There aren’t any significant changes to the lane following performance except for one outlier while interpolating with 3 line segments. If we look closer, we can see that the actual performance gets worse the more we interpolate due to processing speed of the raspberry pi.

<center><img figure-id="fig:perf_segment_t" figure-caption="Standard deviation of $d$ and mean latency for different segment interpolations." src="perf_segment_t.png" alt="Performance evaluation of segment interpolation" style="width: 400px;"/></center>




[](#fig:perf_segment_t) shows the standard deviation of $d$ ($d_{stdev}$) and how it increases the more we interpolate due to higher latency. This behavior is observable on straight lanes where the Duckiebot oscillates around the reference trajectory. We can see this in [](#fig:perf_segment_osc) for a run with 5 interpolated segments per detected segments. From 50 to 70 sec we can observe the oscillations on a straight lane due to high latencies.

<center><img figure-id="fig:perf_segment_osc" figure-caption="Benchmark graph for a run with 'a313' and 5 interpolated segments per detected segment." src="perf_segment_osc.png" alt="Performance evaluation of segment interpolation" style="width: 400px;"/></center>



#### Curvature estimation benchmark

In this section we want to evaluate the curvature estimation performance. What the curvature estimator basically does is dividing the input image into several circular sections with equi-radial distance to the Duckiebot. From each section it derives the pose and evaluates, how it changes in these sections. This will tell us, how the road in front of the Duckiebot looks like. Then again, this feature has an impact on the lane following performance of the Duckiebot since the processing power of the raspberry pi is limited and any added latency will slow the bot down.

<center><img figure-id="fig:perf_curvature_estim" figure-caption="Performance for different numbers of belief images from 1 to 7 where the first image is for the actual pose estimation and the further ones are for the curvature estimation (curvature resolution)." src="perf_curvature_estim.png" alt="Performance evaluation of curvature estimation" style="width: 400px;"/></center>

    

In [](#fig:perf_curvature_estim) on the horizontal we can see the number of belief images being evaluated (curvature estimation resolution). The higher the number, the better we can forecast the type of the road (left curve, right curve, straight lane). With a number of 1, there is no curvature estimation (basically the old pose estimator). We can see that the performance compared to the old pose estimator is much better. This is because the reference run with 1 belief image has been recorded before and the calibration may have changed. Anyway, we can see a decrease in lane following performance, the higher the amount of belief images or image sections are created. This is due to higher cpu cost and increased latency. From tests we can see that a number of 4 belief images is sufficient to tell in most cases, at what kind of road type we are looking at. 

<center><img figure-id="fig:perf_curvature_estim_dstd" figure-caption="Standard deviation of $d$ and $\phi$ for different numbers of belief images." src="perf_curvature_estim_dstd.png" alt="Performance evaluation of curvature estimation" style="width: 400px;"/></center>



A look at [](#fig:perf_curvature_estim_dstd) showing the standard deviation tells us, that performance decreases with higher numbers of belief images (curvature estimation resolution).

<center><img figure-id="fig:perf_curvature_estim_t" figure-caption="Segment processing time and number of segments for different numbers of belief images." src="perf_curvature_estim_t.png" alt="Performance evaluation of curvature estimation" style="width: 400px;"/></center>



Same as in other sections, the main performance is heavily depending on the overall latency of the code being executed on the Duckiebot. The latency of segment processing is shown in [](#fig:perf_curvature_estim_t)

We improved the code on curvature estimation and retook all tests to better compare how the Duckiebot behaves. In the following we will see similar graphs with 1 belief image on the old pose estimator, 4 and 7 belief images on the new curvature estimator and again 1, 4 and 7 belief images on the improved estimator.

<center><img figure-id="fig:perf_curvature_estim_improved" figure-caption="Performance of 1, 4 and 7 belief images for old curvature estimation on the left 1, 4 and 7 belief images for improved curvature estimation on the right." src="perf_curvature_estim_improved.png" alt="Performance evaluation of improved curvature estimation" style="width: 400px;"/></center>




It is observable in [](#fig:perf_curvature_estim_improved) that the improved curvature estimation performs slightly better in all three cases. In [](#fig:perf_curvature_estim_t_improved) we see that the latency for the improved curvature estimator is lower and therefore the case with just one belief image (meaning the curvature estimation is turned off) performs especially well. 

<center><img figure-id="fig:perf_curvature_estim_t_improved" figure-caption="Segment processing time and number of segments of 1, 4 and 7 belief images for old curvature estimation on the left 1, 4 and 7 belief images for improved curvature estimation on the right." src="perf_curvature_estim_t_improved.png" alt="Performance evaluation of improved curvature estimation" style="width: 400px;"/></center>




### Performance Evaluation of Controller

#### Stopping in front of red stop line

We evaluated if the Duckiebot is able to stop in front of the red stop line within the defined specifications. In order to test the stopping behavior, we tested the old controller and the new controller and measured the pose in front of the stop line. The results in [](#tab:redline_results) show that we are able to improve the stopping in front of the red line. The performance shows to be in the bound of the target values.  The stopping distance to the center of the red line should be 16 to 10 cm and the final angle should be in the range of $\phi=-10^{\circ}$ to $\phi=10^{\circ}$.

<col4 align='center' style="text-align:left" id='redline_results' figure-id="tab:redline_results" figure-caption="Stopping at Stop Line Evaluation.">
   <span> </span>      <span>$d_{mean}$</span>     <span>$$\phi_{mean}$$</span>       <span><b>Mean stopping distance to center of red line</b></span>
   <span> Old Controller </span>     <span>$5.6cm$</span>    <span>$5^{\circ}$</span>    <span>$17.4 cm$</span>
   <span>New Controller</span> <span>$-0.6cm$</span>     <span>$3.6^{\circ}$</span>    <span>$8.2cm$</span>
</col4>

#### Controller benchmark

The controller has been improved in several steps. In the first step, control parameters have been tuned and an integral part has been added. In a second step, the static offset has been corrected as well.

<col5 align='center' style="text-align:left" id='controller_results' figure-id="tab:controller_results" figure-caption="Results for controller evaluation of old controller, new integral part and offset correction.">
  <span> </span>    <span>$d_{mean} [cm]$</span>    <span>$d_{std} [cm]$</span>    <span>$\phi_{mean} [rad]$</span>    <span>$\phi_{std} [rad]$</span>
  <span>Old Controller</span>    <span>3.16</span>     <span>0.45</span>   <span>-0.40</span>    <span>0.1</span>
  <span>New Integral Part</span>    <span>-2.08</span>    <span>0.08</span>    <span>-0.11</span>   <span>0.07</span>
  <span>Offset Correction</span>    <span>-0.45</span>     <span>0.16</span>    <span>-0.03</span>     <span>0.20</span>
</col5>


As observable in [](#tab:controller_results), the lane following performance increases drastically after implementing all improvements to the controller.

Some more results and words by the controller part? These are the results highlighted in violet on the results excel file on the control sheet.

#### Non-conforming curve benchmark

Shall we even put this?

<object width= "500" height="500" type="application/pdf" data="nonconforming_curve.pdf"></object>


<!-- Ich han es pdf vo de tabelle igfüegt -->


#### Offset minimization in straight lanes

Some words by devel-mstalder on this maybe?

#### Performance of the controller on curvy road

Benchmark of Duckiebot with baseline controller from last year’s implementation. Most notably, the median lateral position of the Duckiebot $d_{est\_median}$ is higher compared to the new implementation of the controller.

Benchmark of Duckiebot with new controller implementation. Most notably, the median lateral position of the Duckiebot $d_{est\_median}$ is lower compared to the old implementation of the controller.

#### Performance of the controller on lanes with dynamic width

Do we even have logs for these?

### Failed Implementations 

**Estimator**    
Although the 7 ranges estimation provided low mean deviation from the actual position and provided good prediction of the upcoming curve as well as its curvature direction.  The 7 ranges estimation failed in the implementation of the lane following demo due to high computation requirement and the caused time latency. 


**Controller**    
Feedforward during lane following:
As the feedforward part during lane following depends entirely on the estimation of the curve, this part failed due to bad estimation of the curves in certain situations as well as the increased latency due to the curvature estimation. Whenever a curve is not correctly detected or not precisely at the beginning of the curve, the feedforward part introduces additional instability. This is especially a problem in the notorious S curves. Therefore, the implementation of the feedforward works good if a precises estimation of the curve is available that works without introducing high latencies. Although, such a precise curvature estimation with low latency is not available at the moment. Hence, the feedforward part during lane following is not robust enough for the current curvature estimation. Nevertheless, the feedforward part is useful for other nodes to interact with the controller. In certain situations other nodes are able to use the feedforward part in order to drive in curves (navigators on intersections and parking team on parking lots).

### Challenges

TODO: Introduction and conclusion 

* Limited computational power of Raspberry Pi.
    - Estimation of curvature introduced high latencies.
    - By increasing the resolution of the picture we would get more segments and this would make both a better pose and curvature estimation possible. Nevertheless, the latency is also increased significantly.
* Duckiebot with different wheels (slippery and non-slippery wheels)
* Some Duckiebots prove to have higher latency when running the same code. This increased latency is a problem.
* Lightning has a very big influence on the performance
    - Depending on the light condition of duckietown the number of detected segments as well as the correctness of the color is varying. Especially reflection on yellow tape makes it appear white. To tackle this issue, a polarisation filter was found to have a positive influence. This might need to be considered in future hardware updates.
    - Anti instagram might not be as good for every light condition.

<!--
<insert video> https://drive.google.com/drive/folders/1sn5SBtQD6vK03epUSpMijCnvV0BMTyAX
No Sound video: https://drive.google.com/open?id=1Hy6EjQ8QakfZliiSp_j2NV78_VpyPvCq
-->

<div figure-id="fig:demo-video">
    <figcaption>Anit Instagram failure on curve</figcaption>
    <dtvideo src="vimeo:256664213"/>
</div>
Effect of anti-instagram on curve detection: If Anti-Instagram is badly calibrated, the Duckiebot will not see enough line segments. This is especially a problem in the curve and the Duckiebot will leave the lane. An example of this failure can be seen in the video above for which we had a bad Anti-Instagram calibration. Hence, the Duckiebot sees not enough line segments and the lane following fails in the curve. To solve the problem Anti-Instagram needs to be relaunched. In the last part of the video above the __X__ button on the joystick is pressed and the Anti-Instagram node gets relaunched. We can see in the last part of the video RVIZ that the number of detected line segments gets increased drastically after the recalibration.

## Future avenues of development {#controllers-final-next-steps}

As there is always more to do and the performance for both the controller and the estimator can still be further enhanced we list in this section some suggestions for next steps to take.

### Estimator
To make curvature estimation applicable it has to be made more robust and at the same time more computationally efficient adding less delay to the system. In its actual state the added delay is too high and the performance with curvature estimation switched on decreases. 

### Controller

Integrate the inputs of other teams, [see.](#sec:controllers-int-report)

* After doing the new wheel calibration (link to the system id report?), the controller parameters need to be adjusted
* To reduce impact of time delays, e.g. a Smith Predictor could be implemented
* For the activation of the remaining interfaces (e.g. intersection navigation and parking), the respective commented out sections of the final code needs to be activated.
* The control parameters should be made dependent on the calibration of the Duckiebot, such that the gains do not have to be tuned for every Duckiebot individually.

### General

* Anti-Instagram should be enhanced, in order to identify more line segments and perceive the correct color.
* Add a polarization filter to reduce impact of reflections on color perception.
* New edge detection with higher accuracy.
* Replace the Raspberry Pi with something more computationally powerful.



<!--
Controller
Identify situations where the car commands are to harsh and shutdown the controller.
Implementation of controller that can compensate for high latencies
Feedforward
Come up with a robust implementation of it that
Feedforward that also works with high latency
Further tuning of the controller parameters.

-->






