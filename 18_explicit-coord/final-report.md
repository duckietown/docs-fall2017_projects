#  Explicit coordination: final report {#explicit-coord-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

_The objective of this report is to bring justice to your hard work during the semester and make so that future generations of Duckietown students may take full advantage of it. Some of the sections of this report are repetitions from the preliminary and intermediate design document (PDD, IDD respectively)._

## The final result {#explicit-coord-final-result}

* Post a video of your best results (e.g., your demo video)

Add as a caption: see the [operation manual](#demo-template) to reproduce these results.

## Mission and Scope {#explicit-coord-final-scope}

Our mission is to coordinate the intersection navigation safely and cleverly through explicit communication.

### Motivation {#explicit-coord-final-result-motivation}

Duckietowns are complex systems where the traffic situations of a real city should be emulated. These towns contain three- and four-way intersections: the Duckiebots should be able to navigate them without crashing into each other and this requires a clever coordination scheme. Intersections represent a key element of a smooth city navigation.


In order to guarantee the success in any condition, it makes sense to have a decentralised system, where each Duckiebot can operate on its own, independently. The ways one can coordinate Duckiebots in an intersection are two:

- Using a traffic light
- Using a communication protocol between the vehicles

### Existing solution {#explicit-coord-final-literature}

A prior implementation for intersection coordination was already available from the 2016's MIT class. The principle was simple: when the Duckiebot comes at an intersection, it stops at the red line and is able to detect with the april tags if there is or not a traffic light. In the first case, the Duckiebot detects the frequency at which the traffic light is blinking and acts based on the road rules. Without a traffic light, the Duckiebot detects the frequency at which the other bots are blinking and adjusts its emitted frequency depending on its state.

Two modules can be distiguished:

- LED emission and detection
- Coordination based on the detected signals


For the emission, three signals can be produced: red, yellow and green light, blinking at different frequencies to distinguish whether the Duckiebot is negotiating (and in which phase of the negotiation it is) or navigating the intersection.

For the detection, the position and frequency of blinking LEDs are registered.

For the coordination, with the assumption that each vehicle can only see other vehicles on its right but not its left, the Duckiebot yields its position if the only visible car is on the right, otherwise the Duckiebot waits (light green or red) or crosses (yellow light).



### Opportunity {#explicit-coord-final-opportunity}

The existing solution had essentially two drawbacks:

- The overall success rate was about 50%: the algorithm for LED-detection, and/or coordination failed, leading to a potential crash of the Duckiebots.
- In case of success, the LED-detection and/or coordination algorithms required an average of four minutes to clear an intersection with four Duckiebots. This results in an extremely slow process, which would block a city with dozens of Duckiebots.

Although the solution was problematic, it still gave us some important intuitions on how to solve the problem. First of all, using a LED-communication protocol is a brillant idea to let the Duckiebots communicate with each other. Since the communication algorithm had the only disadvantage of being slow, we started by re-thinking the coordination algorithm, which contained some bugs. The existing implementation for the coordination was rather complex and articulated, resulting in confused strategies which led to the failure rate of 50%. In order to develop a simpler and lighter algorithm we took inspiration from an existing media access control protocol (MAC): the so called Carrier Sense Multiple Access (CSMA, https://en.wikipedia.org/wiki/Carrier-sense_multiple_access). This algorithm gave us the basic idea behind our strategy and allowed us to have a lighter protocol. In the second place, we re-designed the LED-detection/-interpreter to be faster and more efficient based on the detection of blobs rather than frequencies.



### Preliminaries (optional) {#template-final-preliminaries}

- Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.

Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)

## Definition of the problem {#explicit-coord-final-problem-def}

### Final objective
Given a Duckietown, Duckiebots must be able to cross an intersection efficiently.
An intersection is said to be cleared efficiently if and only if:

- The time needed to cross the intersection is below 1 minute.
- Only one Duckiebot at a time crosses the intersection.
- The Duckiebots do not incur into incidents during the navigation.

### Assumptions
The following assumptions are made for the LEDs communication:
- The Duckiebot is of type DB17-l, i.e. has LEDs mounted on it.
- One to four Duckiebots are at the intersection with a certain position and orientation with respect to the stop line. Responsible for this assumption are The Controllers, which should guide the Duckiebot towards the intersection based on the following measures, based on the projection of the Duckiebot on the 2D lane of the road. The Duckiebot should be
  - Min. 0 cm behind the stop red line;
  - Max. 6 cm behind the stop red line;
  - Max. +/- (left/right deviations) 2 cm from the center of the line;
    +/- 10° of rotation with respect to the perpendicular line of the red line.

- Duckiebots are able to see the vehicles in front and on the right with respect to their position: one cannot assume that the left visual is clear.
- LEDs work properly and emit the signals with the correct color and frequency. We can also detect the LEDs as the correct state that they represent, and attribute those LEDs to the correct Duckiebot that is displaying them.
- The Duckiebots do not move while “waiting” at the intersection, but they can move on the spot to look around to see the left.
- Camera works properly (frequency 30Hz, resolution of 64x48).

The following assumptions are made for the coordination:

- Signals are correctly recognised and associated to the corresponding messages.
- The intersection is among one of the standard intersections of Duckietown, detected through april tags.
- The intersection type and the presence of a traffic lights are known.
- Intersection navigation (for which The Navigators are responsible) is guaranteed to be working safely.
- One Duckiebot navigates the intersection at the time.


### Performance metrics
Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal

## Contribution / Added functionality {#explicit-coord-final-contribution}

Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_

## Formal performance evaluation / Results {#explicit-coord-final-formal}

_Be rigorous!_

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#explicit-coord-final-next-steps}

There is room for improvement for the coordination part of this project.
Our approach, in the case of an intersection without traffic light, prioritises robustness rather than efficiency (in some cases all the Duckiebots at an intersection could turn off and restart the whole protocol again) and it is easy to imagine a scenario with an improved efficiency (tradeoff with complexity).

An idea would be to encode in the signal also the intentions of the Duckiebot and, by doing so, allow multiple Duckiebots to navigate the intersection at the same time if their directions are compatible. In fact, if two Duckiebots wanted to go straight they could move at the same time.
The clearing time could also be reduced in the case of an intersection with traffic light if the latter was able to see where the vehicles are (prevent the light to turn green in a direction where no vehicle is waiting to cross).
