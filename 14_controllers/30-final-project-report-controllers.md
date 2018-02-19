#  Group name: final report {#controllers-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

_The objective of this report is to bring justice to your hard work during the semester and make so that future generations of Duckietown students may take full advantage of it. Some of the sections of this report are repetitions from the preliminary and intermediate design document (PDD, IDD respectively)._

## The final result {#controllers-final-result}

_Let's start from a teaser._

* Post a video of your best results (e.g., your demo video)

Add as a caption: see the [operation manual](#demo-template) to reproduce these results.

## Mission and Scope {#controllers-final-scope}

**IMPERIUM ET POTESTAS EST ** (With control comes power)

Our **Mission** was to make lane following more robust to model assumptions and Duckietown geometric specification violations and provide control for a different reference control.

Our **Scope** was
- Control Duckiebot on straight lane segments and curved lane segments.
- Robustness to geometry (width of lane, width of lines)
- Detection and stopping at red (stop) lines
- Providing control for a given reference d for avoidance and intersections (but for intersections, we additionally need the pose estimation and a curvature from the navigators team)

Out of sope was  
- Pose estimation and curvature on Intersections (plus navigation / coordination)
- Model of Duckiebot and uncertainty quantification of parameters (System Identification)
- Object avoidance involving going to the left lane
- Extraction and classification of edges from images (anti-instagram)
- Any hardware design
- Controller for Custom maneuvers (e.g. Parking, Special intersection control)
- Robustness to non existing line

### Motivation {#controllers-final-result-motivation}

_Now step back and tell us how you got to that mission._

- What are we talking about? [Brief introduction / problem in general terms]
- Why is it important? [Relevance]

In the Duckietown, Duckiebots  are cruising on the streets and also Duckies are sitting on the sidewalk waiting for a Duckiebot to pick them up. To ensure a baseline safety of the Duckiebots and the Duckies, we have to make sure the Duckiebots are able to follow a path or the lane and stop in front of red lines. For example, the Duckiebot is driving on the right lane it should never cross the middle lane to avoid any collisions with an oncoming Duckiebot. 

The goal of our module is to stay in the lane while driving and stop in front of a red line. Due to the tight time plan we focused on improving the existing code and benchmarking the module. In order to let the Duckiebot drive to a given point, the robot has to know where it is in the lane and where it should be in the lane. To retrieve the information location, a pose estimator is implemented. The estimator receives line segments from the image pipeline with information about line tape colour (white, yellow, red) and whether the segment is on the left or right edge of the line tape. Using the given information, we determine if the Duckiebot is inside or outside of the lane and how far it is from the middle of the lane.  The relative location to the middle of the lane and the orientation of the Duckiebot are passed to the controller. 

### Existing solution {#controllers-final-literature}

- Was there a baseline implementation in Duckietown which you improved upon, or did you implemented from scratch? Describe the "prior work"

From the project last year, a baseline implementation of a pose estimator and a controller were provided for us for further improvement. The prior pose estimator was designed to deliver the pose for a Duckiebot on straight lanes. If the Duckiebot was in or before a curve and in the middle of the lane, the estimated pose showed an offset **d** , see definition of **d** in figure below. The existing controller was a P controller and worked well on straight lines. Due to the inputs from the pose estimator to the controller, the Duckiebot overshot in the curves and crossed the left/right line during or after the curve. 

<center><img figure-id="fig:curve_plot" figure-caption="Pose of Duckiebot in a curve element." src="curve_plot.png" alt="Curve plot" style="width: 200px;"/></center>


### Opportunity {#controllers-final-opportunity}

- What didn't work out with the existing solution? Why did it need improvement?
In the previous implementation, the following of the lane was not guaranteed for curved lane because the Duckiebot often left the lane during or after the curve. Although the Duckiebot sometimes returned to the right lane after leaving it and continued following the lane, robust lane following was not guaranteed. On straight lanes, the Duckiebot often had a large offset. To ensure the Duckiebot driving in the middle of the lane regardless of lane geometry, the pose estimator and the controller had room for improvement. 


#### Estimator 
To improve curve following, our main goal is to decrease the overshoot after a turn because the previous implementation of the pose estimator was designed for straight lanes only. We planned to predict if a curve is upcoming and in which direction the turn will be. The curvature prediction should set the according parameter of the feedforward controller,  so the PID controller is supported in the curve for smoother transition. 

We planned on improving the **d** and **$\phi$** estimation by
- Increasing the number of ranges in image where the segments are detected. 
- First approach: 7 ranges to weight the the closer segments more and the further segments less. The closer segments return the actual pose and heading of the Duckiebot. The further segments refer to the up-coming lane tile type straight or curved and the problem of the further segments is the higher noise ratio.
- Second approach: 3 ranges
- Third approach: fft
- Fourth approach: dft
To improve curve following, our main goal is to decrease the overshoot after a turn. We planned to predict if a curve is upcoming and in which direction the turn will be. The curvature prediction should set the according parameter of the feedforward controller,  so the PID controller is supported in the curve for smoother transition. 

#### Controller

Running the current lane following demo, we determined the weaknesses of the Controller’s performance. Since the current controller only had a P-part, we decided to implement following parts and benchmark its performance.
- Integrator for both **$\phi$** and **d**
- First approach: No saturation for Integrator
- Problem: Very strong oscillation
- Second approach: Saturation for integrator
- Performance improved very much. Oscillation is reduced
- Third approach: Saturation of Integrator and Reset of Integrator whenever zero error is reached
- Performance improved again.
- Fourth approach: Added true integration with correct timestep instead of simply summing up the error and neglecting the timestep
- This is the correct approach since the time step must need to be taken into account.
- Feed forward for driving on a curved lane
- We take the current curvature as an Input for the feedforward part
- The Feedforward is very much dependent on the correct curvature estimation. If correct curvature was estimated feedforward worked good.

#### General

The previous work wasn’t benchmarked for robustness nor performance, so we could define various tests to benchmark the previous and our solution and in the end to compare our implemented improvement.  

### Preliminaries (optional) {#controllers-final-preliminaries}

- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.

Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)


## Definition of the problem {#controllers-final-problem-def}

_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical  definition of the problem you tackled. You can use part of the preliminary design document to fill this section._



Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal

## Contribution / Added functionality {#controllers-final-contribution}

### Curvature Estimation
#### Curvature estimation using multiple domains    

The idea of our curvature estimation approach is to split the domain in front of the Duckiebot into multiple range areas. A version using three areas is shown in [](#fig:curve_plot).

<center><img figure-id="fig:curve_plot" figure-caption="Curvature estimation using the segments in different domains." src="curvature_estim.svg" alt="Curvature estimation" style="width: 300px;"/></center>




Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_

## Formal performance evaluation / Results {#controllers-final-formal}

_Be rigorous!_

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

We evaluated the improvement of performance with help of several evaluations. The evaluation procedure are defined in our [Indermediate Report](#controllers-int-report). Here we present the corresponding results.

**Stopping in front of red stop line**

We evaluated if the Duckiebot is able to stop in front of the red stop line within the defined specifications. In order to test the stopping behavior, we tested the old controller and the new controller and measured the pose in front of the stop line. The results show that we are able to improve the stopping in front of the red line. The performance shows to be in the bound of the target values.  The stopping distance to the center of the red line should be 16 to 10 cm and the final angle should be in the range of $\phi=-10°$ to $\phi=10°$.


|                    | $d_{mean}$     | $\phi_{mean}$     | Mean stopping distance to center of red line     |
|----------------    |------------    |---------------    |----------------------------------------------    |
| Old Controller     | 5.6 cm         | 5°                | 17.4 cm                                          |
| New Controler      | -0.6 cm        | 3.6°              | 8.2 cm                                           |

Table: Stopping at Stop Line Evaluation

**Pose Estimation**


**Offset minimization in straight lanes**

**Performance of the controller on curvy road**
Benchmark of Duckiebot with baseline controller from last year’s implementation. Most notably, the median lateral position of the duckiebot $d_{{dest_median}}$ is higher compared to the new implementation of the controller.

Benchmark of Duckiebot with new controller implementation. Most notably, the median lateral position of the duckiebot $d_{{dest_median}}$ is lower compared to the old implementation of the controller.

**Performance of the controller on lanes with dynamic width**



####The failed Implementations and Challenges:
**Estimator**
Although the 7 ranges estimation provided low mean deviation from the actual position and provided good prediction of the upcoming curve as well as its curvature direction.  The 7 ranges estimation failed in the implementation of the lane following demo due to high computation requirement and the caused time latency. 
**Controller**
Feedforward:
As The Feedforward dependence entirely on the estimation of the curve this part failed due to bad estimation of the curves in certain situations as well as the increased latency due to the curvature estimation. Whenever a curve is not correctly detected or not precisely at the beginning of the curve the feedforward part introduces additional instability. This is especially a problem in the notorious S curves. Therefore, the implementation of the feedforward works good if a precises estimation of the curve is available that works without introducing high latencies. Although, such a precise curvature estimation with low latency is not available at the moment. Hence, the feedforward is not robust enough for the current curvature estimation. Nevertheless, the feedforward is useful for other nodes to interact with the controller. In certain situations other nodes are able to use the feed forward part in order to drive in curves.

#### Challenges
- Limited computational power of Raspberry Pi.
- Estimation of curvature introduced high latencies.
- By increasing the resolution of the picture we would get more segments. Nevertheless, the latency is also increased significantly.
- Duckiebot with different wheels (slippery and non slippy wheels)
- Some Duckiebots prove to have higher latency when running the same code. This increased latency is a problem.
- Lightning has a very big influence on the performance
- Depending on the light condition of duckietown the number  of detected segments as well as the correctness of the color is varying.
- Anti instagram might not be as good for every light condition.

## Future avenues of development {#controllers-final-next-steps}

### Controller


### Estimator
To make curvature estimation applicable it has to be made more robust and at the same time more computationally efficient adding less delay to the system. In its actual state the added delay is too high and the performance with curvature estimation switched on goes down. 

Controller
Identify situations where the car commands are to harsh and shutdown the controller.
Implementation of controller that can compensate for high latencies
Feedforward
Come up with a robust implementation of it that
Feedforward that also works with high latency
Further tuning of the controller parameters.
General
Anti-Instagram should be enhanced, in order to identify more line segments and perceive the correct color.
New edge detection with higher accuracy.

