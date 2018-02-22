me: final report {#implicit-coord-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->


## The final result {#template-final-result}

_Let's start from a teaser._

* Post a video of your best results (e.g., your demo video)

Add as a caption: see the [operation manual](#demo-template) to reproduce these results.

## Mission and Scope {#template-final-scope}

_Now tell your story:_

Define what is your mission here.



### Motivation {#template-final-result-motivation}

_Now step back and tell us how you got to that mission._

- What are we talking about? [Brief introduction / problem in general terms]

- Why is it important? [Relevance]




### Implicit  Coordination Motivation:
Intersection coordination is a crucial task when it comes to safety and avoiding congestions, since it is a bottleneck in road traffic. We built a system for implicit coordination which is able to function without traffic lights and explicit communication e.g. communicating over WLAN or LEDs with other vehicles.
The advantage of our coordination algorithm is that it doesn’t rely on the latter. Hence we can still guarantee a fluid and save road traffic at intersections, even if the communication or coordination hardware is perturbed or dead.

### Follow the Leader Motivation:
A fluid and smooth road traffic is beneficial in many ways: It saves time and leads to better energy efficiency and more safety.  Therefore wavelike road traffic behaviour has to be omitted. Our idea was to create an algorithm that tries to homogenize the road traffic by forcing the vehicles to keep a certain distance to each other.






### Existing solution {#template-final-literature}

- Was there a baseline implementation in Duckietown which you improved upon, or did you implemented from scratch? Describe the "prior work"

### Opportunity {#template-final-opportunity}

- What didn't work out with the existing solution? Why did it need improvement?

Examples:
- there wasn't a previous implementation
- the previous performance, evaluated according to some specific metrics, was not satisfactory
- it was not robust / reliable
- somebody told me to do so (/s)

* How did you go about improving the existing solution / approaching the problem? [contribution]
- We used method / algorithm xyz to fix the gap in knowledge (don't go in the details here)
- Make sure to reference papers you used / took inspiration from



### Existing Solution and Opportunity Implicit Coordination
There was no previously existing solution for the implicit coordination problem. As stated in the paragraph above, a solution for this problem is very desirable if not absolutely necessary for an autonomous driving system. Since implicit coordination at intersections was rather a tabula rasa for us, we gathered our ideas from various fields including game, communication and network theory.

### Existing Solution and Opportunity Follow the Leader
The idea to use fiducial tags for the follow the leader problem on the other hand already existed. However, this task was implemented in a rather crude way. The Duckiebots were just ought to perform a full stop, whenever they detected another Duckiebot. We added a pose estimation of the leader and thus were able to create a much more sophisticated controller.



### Preliminaries (optional) {#template-final-preliminaries}

- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.

Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)

## Definition of the problem {#template-final-problem-def}

_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical definition of the problem you tackled. You can use part of the preliminary design document to fill this section._

Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal







### Definition of the Problem Implicit Coordination:
The final objective for this part was that, when two, three or four Duckiebots arrive at the same time at an intersection, they are able to handle the challange of who is allowed to drive first, autonomously and without any means of explicit communication. They are however allowed to use implicit communication. Which means they are allowed to observe the other Duckiebots and draw conclusions about the intents of the other Duckiebots from these observations. For this, we assumed that:
Duckiebots do not use explicit communication, e.g. LEDs, WLAN ...
Duckiebots have different appearance.
All Duckiebots are autonomous, not remote controlled
All Duckiebots use the same formation and implicit control algorithm.
For evaluating the performance, we decided to test our algorithm at an intersection and judge by how many Duckiebots can be handled and in what time it does so.

Definition of the Problem Follow the Leader:
The final goal here, was that the Duckiebots can follow another Duckiebot in front of them and adjust their velocity accordingly. Meaning ideally, they slow down if the leading Duckiebot does so and accelerate analogously. The assumptions here were:
All Duckiebots use the same algorithm
All Duckiebots are equipped with a fiducial tag that allows us to estimate their relative position and pose.
The success can be easily evaluated by how many Duckiebots can follow their respective leader at the same time. Furthermore keeping an equal distance between the Duckiebots performance criterion.











## Contribution / Added functionality {#template-final-contribution}

Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_
### Contribution Implicit Coordination:
Our implicit coordination algorithm is inspired by the  Carrier Sense Multiple Access/Collision Detection (CSMA/CD) algorithm which handles the access of different parties on a shared resource. In our case the Duckiebots represent the parties and the shared resource correlates with the intersection. This CSMA/CD not just guarantees us, that all Duckiebots are crossing the intersection safely, but is also enables us to give insightful estimates of the maximum throughput and the average waiting time at the intersection, given by the rich theory behind CSMA/CD. Our implementation of CSMA/CD for intersection coordination works the following:
1. Drive towards the intersection and stop at the stopline
2. Wait a random timespan and check if a Duckiebot in your field of view is driving using the Duckiebot detection algorithm
3. If no  other Duckiebot is driving cross the intersection. Else repeat Step 2.

Additionally we have implemented rigth priority option in order to accelerate the traffic at the intersection. Rigth priority doesn't allow a Duckiebot to drive and as lang as another Duckiebot is standing right to them at an intersection.

<div figure-id="fig:DemoMap" figure-caption="Process Flow Chart Implicit Coordination">
     <img src="FlowChartImplicit.png" style='width: 10em'/>
</div>

### Contribution Follow The Leader:
The Duckiebots have two steering inputs, the forward velocity and the angular velocity. The angular velocity is calculated by the lane following algorithm and not changed here. All our algorithm does is calculate the velocity required to maintain a predefined distance to the vehicle in front:

#### Pose Estimation
This algorithm consists of roughly two parts: The first one is the pose estimation of the leading Duckiebot and the other one is the adjustment of the velocity according to this estimate.
We used an OpenCV library blob detector that detects fiducial tags from the camera. From the pixel coordinates of the detected blobs, i.e. the markers of the fiducial tag, the transformation matrix between the tag coordinate frame and the camera coordinate frame is calculated. The OpenCV algorithm gives us the the transformation T_CP, which is the homogeneous transformation from camera to tag. We map the this transformation in 3D to the 2D case, where the pose of the Duckiebot will be represented by ρ, psi and theta..
First T_CP is inverted which results in T_PC. From this transformation we extract the rotation matrix R_PC and the translation vector t_PC.
From T_CP = [R_CP, t_CP; 0,0,0,1] we extract R_CP and t_CP. From there we can easily obtain R_PC = R_CP⁻1 and t_PC = -t_CP.
We assume that the tag is centered on the Duckiebot’s rear such that the Z-axis of the tag coordinate system is aligned with the X-axis of the Duckiebot and the X-axis of the tag with the Y-axis of the leading Duckiebot. (The view of the image is from above.)
With these assumptions we can calculate the pose of the leading Duckiebot in the coordinate system of the following Duckiebot.
ρ is the distance between the Duckiebots, calculated from the translation vector.
psi is the rotation of the tag around its Y-axis
And finally theta can be calculated from the calculated psi.
ρ = sqrt(x²+y²) (from t_CP)
theta = arctan2(y/x) o.ä?
psi = ? arccos(R(1,1)) ?
The blob detector outputs the distance ρ between the camera and the tag, the angle theta and finally, the angle psi of the tag relative to the camera.
The velocity of the leader is calculated according to the difference in distance to the leader between two consecutive detections, divided by time between the two detections.
(ρ_1 – ρ_2)/deltaT
Thus, we get the two ouputs of the black box in the picture, d_Leader and v_Leader.

<div figure-id="fig:DemoMap" figure-caption="Coordination Trafo1">
     <img src="CoordTrafo1.png" style='width: 10em'/>
</div>
<div figure-id="fig:DemoMap" figure-caption="Coordination Trafo2">
     <img src="CoordTrafo2.png" style='width: 10em'/>
</div>


#### Velocity Control
The following calculations are also illustrated in the picture. First, the actual distance between the two Duckiebots is subtracted from the desired distance d*.
e_d = d* - d_Leader
e_d holds information whether the distance to the leading Duckiebot is too large, too small or just right. From here, we calculate a velocity to adjust this distance to the desired one.
deltav = K_D/T * e_d
e_d/T is the velocity required to compensate the missing distance till the (presumed) next measurement. K_D is a design parameter.
Deltav is then added to the estimated velocity of the leader.
e_v = delta_v + v_Leader
delta_v adjusts the distance between the two Duckiebots and by adding it to v_Leader we ensure that the two vehicles drive with roughly the same velocity. The final velocity e_v is then again multiplied with a design parameter K_p. This helps to dampen the the rather noisy pose estimation of the leader.
v_Duckiebot = K_p*e_v
The resulting v_Duckiebot is then used as the input for the Duckiebot.
Further Details
Lastly, there are some precautions not shown in the picture: If the velocity v_Duckiebot is smaller or equals to 0, both the velocity and omega input of the Duckiebot are set to 0. It is undesirable, that the Duckiebots start to drive backwards, as they cannot follow the lanes or avoid obstacles that way. If omega is not set to 0, the Duckiebots start rotating on the spot which – besides looking bad – causes them to lose track of the fiducial tag of the Duckiebot in front of them which in turn causes them to collide.
Finally, if the distance d_Leader falls under a certain threshold, an emergency brake is performed.
<div figure-id="fig:DemoMap" figure-caption="Controller">
     <img src="Controller.png" style='width: 10em'/>
</div>



## Formal performance evaluation / Results {#template-final-formal}

### Results and Performance Evaluation Implicit Coordination
Omitting possible errors which might occur in case of the implicit coordination at intersections, one should take the following precautions.
You need the correct april tags at the intersection, otherwise the Duckiebot won't know what kind of situation (intersection) it is dealing with. When the stopline isn't detected the algorithm doesn't start, so all Duckiebots should stop at the stopline. Furthermore you can get problems with twisted coordination systems for the detected position of other Duckiebots if your extrinsic camera calibration is wrong on the laptop (assuming your running the detection node on your laptop).Sometimes a robot is detected if there isn't actually one, which could slow down the traffic at the intersection. We agreed on this with our Canadian friends who did the detection, since we would otherwise risk to overlook a real Duckiebot which would be fatal. In rare cases the detection does not detect a robot. In order to assure the detection works as good as possible I would suggest relaunching the multivehicle detection node regularly, since it seems to start lagging the longer it is running. If you would like to keep track of the detection you can run rostopic echo /robotname/multivehicle_tracker_node/tracking.
The algorithm is designed for up 4 robots at the stoplines, but since we depend on the indefinit navigation, we are prone to navigation mistakes. If the tracking and navigation are correct, we are able to coordinate 4 way intersections for up to 4 Duckiebots. Empirical tests showed that our algorithm never needed more than 30 seconds to clear a four way intersection with 4 Duckiebots.

### Results and Performance Evaluation Follow the Leader
We tested our follow the leader with up to four Duckiebots in duckietown and there doesn’t seem to be an upper limit on the number of Duckiebots following each other. Regarding the equal distance we are somewhat restricted by the computational power of the Duckiebots and hence the time needed for the detection of the antecedent Duckiebot. The detection time can vary from image frame to image frame however, 0.4 seconds used to be an appropriate upper bound. We found that this delay lead to deviations of maximally 20% from our optimal reference distance. In order to function properly the gain of the wheel calibration should be set to 0.6 as proposed by the Controllers to assure a smooth interplay between our controller and the lane following algorithm. Note that very high gains can dramatically worsen the deviations from the reference distance. Additionally, as always, a correct camera and wheel calibration are crucial for a fluid traffic.



_Be rigorous!_

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#template-final-next-steps}


_Is there something you think still needs to be done or could be improved? List it here, and be specific!_


### Future avenues Implicit Coordination
Here, the detection algorithm could be improved. As described above, it starts to lag after a certain time and needs to be restarted time and again. Otherwise the algorithm is not very robust. Additionally, the tradeoff between false positives and false negatives could be tuned. Right now, the Duckiebots are far more likely to detect vehicles that are not there then to not detect vehicles that are there. While this makes sense in order to avoid collisions, it can also lead to a Duckiebot waiting for a long time at a free intersection. Also, the detection requires a lot of computational power from the Duckiebots that is currently not available other than on a laptop. This leads to the aforementioned lagging. Maybe, there is a different solution?

### Future Avenues Follow the Leader
An improvement for the Follow the Leader algorithm could be to git rid of the fiducial tags and try to follow each other solely depending on detecting the other Duckiebots. This could be done with the detection node we used for the imlicit coordination, however the detection is a lot less robust.
