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




###Implicit  Coordination Motivation:
Intersection coordination is a crucial task when it comes to safety and avoiding congestions, since it is a bottleneck in road traffic. We built a system for implicit coordination which is able to function without traffic lights and explicit communication e.g. communicating over WLAN or LEDs with other vehicles.
The advantage of our coordination algorithm is that it doesn’t rely on the latter. Hence we can still guarantee a fluid and save road traffic at intersections, even if the communication or coordination hardware is perturbed or dead.

###Follow the Leader Motivation:
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
The idea to use fiducial tags for the follow the leader problem on the other hand already existed. However, this task was implemented in a rather crude way. The Duckiebots were just ought to perform a full stop, whenever they detected another duckiebot. We added a pose estimation of the leader and thus were able to create a much more sophisticated controller.



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
The success can be easily evaluated by how many Duckiebots can follow their respective leader at the same time. Furthermore keeping an equal distance between the duckiebots performance criterion.











## Contribution / Added functionality {#template-final-contribution}

Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_
###Contribution Implicit Coordination:
Our implicit coordination algorithm is inspired by the  Carrier Sense Multiple Access/Collision Detection (CSMA/CD) algorithm which handles the access of different parties on a shared resource. In our case the Duckiebots represent the parties and the shared resource correlates with the intersection. This CSMA/CD not just guarantees us, that all duckiebots are crossing the intersection safely, but is also enables us to give insightful estimates of the maximum throughput and the average waiting time at the intersection, given by the rich theory behind CSMA/CD. Our implementation of CSMA/CD for intersection coordination works the following:
1. Drive towards the intersection and stop at the stopline
2. Wait a random timespan and check if a Duckiebot in your field of view is driving using the duckiebot detection algorithm
3. If no  other duckiebot is driving cross the intersection. Else repeat Step 2.
Additionally we have implemented rigth priority option in order to accelerate the traffic at the intersection. Rigth priority doesn't allow a duckiebot to drive and as lang as another duckiebot is standing right to them at an intersection.

<div figure-id="fig:DemoMap" figure-caption="Process Flow Chart Implicit Coordination">
     <img src="FlowChartImplicit.png" style='width: 15em'/>
</div>

###Contribution Follow The Leader:
The Duckiebots have two steering inputs, the forward velocity and the angular velocity. The angular velocity is calculated by the lane following algorithm and not changed here. All our algorithm does is calculate the velocity required to maintain a predefined distance to the vehicle in front:




## Formal performance evaluation / Results {#template-final-formal}

The algorithm is designed for up 4 robots at the stoplines, but since we depend on the indefinit navigation, it only runs robustly for 2-3 robots at the moment. If the tracking and navigation are correct, we are able to coordinate 4 way intersections for up to 4 Duckiebots. Empirical tests showed that our algorithm never needed more than 30 seconds to clear a four way intersection with 4 duckiebots.






_Be rigorous!_

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#template-final-next-steps}

_Is there something you think still needs to be done or could be improved? List it here, and be specific!_
