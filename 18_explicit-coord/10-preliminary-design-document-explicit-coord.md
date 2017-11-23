# PDD - Explicit Coordination {#explicit-coordination-pdd status=beta}


## Part 1: Mission and scope

### Mission statement

Coordinate intersection navigation safely through explicit communication.

### Motto



> IN HOC SIGNO VINCES

English: In this sign you will conquer

### Project scope


Employ LEDs based communication to efficiently manage traffic at intersections. First, LEDs should be used to reliably communicate Duckiebots positions and/or intentions. Then, optimal control theory or game theory might be considered to safely clear the intersection in reasonable time. By safely, we mean that no collisions occur (100% success). This requires robustness in message interpretation.

We aim to solve the problem of clearing the intersection in max. 60 seconds (meaning that even in the case of four Duckiebots participating we can solve the problem below this time). A decentralized solution should be implemented.


#### What is in scope

The following aims are identified:

* List the messages that need to be exchanged between Duckiebots;
* Encode the messages in LED language;
* Produce the LED signal;
* Detect the LED signal;
* Decode the LED signal;
* Act safely upon the received information.


#### What is out of scope

The following aims are beyond the scope of the project:

* Changing the communication protocol, i.e., we will stick on using LEDs.


#### Stakeholders

The following pieces of Duckietown will be interacting with the project:

* Fleet planning;
* Implicit-navigation (determinate where the Duckiebot in the row has to stop);
* Anti-instagram filtering;
* Integration heros;
* Map designers;
* Traffic navigation;
* Controllers.


## Part 2: Definition of the problem

### Mission

Duckiebots must be able to cross an intersection in the defined reasonable time and without collisions.



### Problem statement

The following problems are to be tackled:

* LED-based communication;
* Coordination at intersections.


### Assumptions

The following assumptions are made for the LEDs communication:

* The Duckiebot is of type DB17-l.
* One to four Duckiebots are at the intersection with a certain position and orientation with respect to the stop line (projection of Duckiebot on 2D lane):
    * Min. 0 cm behind red line;
    * Max. 6 cm behind red line;
    * Max. +/- 2 cm from center of the line;
    * +/- 10° of rotation.
* Duckiebots are able to see the vehicles in front and on the right with respect to their position.
* LEDs work properly and emit the signals with the right color and frequency, and we can also detect the LEDs as the correct state that they represent, and attribute those LEDs to the correct Duckiebot that is displaying the LEDs.
* The Duckiebots do not move while “waiting” in the intersection, but they can move on the spot to look around to see the left.
* Camera works properly (frequency 30Hz, resolution of 64x48).

The following assumptions are made for the coordination:

* Signals are correctly recognized and associated to the corresponding messages.
* The intersection is among one of the standard intersections of Duckietown. That is, weird intersections will be ignored (at least in a first approach to the problem).
* The intersection type and the presence of a traffic lights are known.
* Intersection navigation is guaranteed to be working safely.
* One Duckiebot navigates the intersection at the time.


### Approach

Firstly, the existing code and see its limits should be tested. Benchmark tests have to be designed in order to carefully measure the performances of the implementation. Then, the literature should check to see what has already been implemented.

The problem can be splitted in LED-based communication and intersection coordination. We further distinguish between intersections with traffic lights and intersections without traffic lights (recall that it is assumed that the Duckiebots know the intersections they entering). In all cases we aim for a decentralized solution.

Possible approaches for LEDs communication:

* Last year’s algorithm;
* Alternative approach 0.

Possible approaches for intersection coordination without traffic lights:

* Last year’s approach;
* Alternative approach 1;
* Alternative approach 2;
* Alternative approach 3.

Possible approach for intersection coordination with traffic lights:

* Detect the signal of the traffic light and behave accordingly.

#### ** Approaches for communication **

*Last year’s approach*: Communication works through LEDs blinking at different frequencies. Colors are just for human understanding and are operational meaningless. See last year’s documentation for further details.


*Alternative approach 0*: Communication works through LEDs blinking at different frequencies, colors, and patterns.


#### ** Approaches for coordination without traffic lights **

*Last year’s approach*: See last year’s documentation.


*Alternative approach 1*:

Assumption: every Duckiebot does not see on the left. Here, Duckiebots do not need to turn in place to see the Duckiebot on the left

There are two messages:

* One signal to say if the Duckiebot can see someone on its right (e.g., a red LED blinking, signal A);
* One signal to say whether the Duckiebot is at intersection or is about to go (e.g., same LED blinking at different frequencies, signal B).

The coordination plan works as follows:

1. If Duckiebot:
    1. does not see signals from the right and
    2. it does see signal A from the opposite side of intersection.
Then it will enter the intersection.
That could happen with 1, 2, 3 Duckiebot at intersection.

2. If 4 Duckiebots are at intersection, that means, every Duckiebot is emitting signals A and B, each Duckiebot turns off with a certain probability (say 25%), so that one is going to see nobody on the right and will navigate the intersection.

3. If one Duckiebot does not see any kind of signal (A or B) it will proceed to navigate the intersection.

4. One case still need to be analyzed, i.e., the one in which we have 2 Duckiebots one opposite to the other. In this case:
     1. The Duckiebot does not see signals from the right and
     2. Does not see signal A from the opposite Duckiebot (but it does see signal B).


Then each Duckiebot turns off with some probability (say 50%) so that it will see no more signals and then navigate the intersection, as explained in case 3.



*Alternative approach 2*:
There are four messages:

* No color: waiting to enter the red queue;
* Red: waiting to enter the negotiation;
* Yellow: about to enter the intersection but still looking around, can go back to green;
* Green: negotiating.

In the following, let you be a Duckiebot.

1. You currently have no color. You stop at the intersection, turn left 20° in order to see all existing bots. You have no lights on at the moment. Possible scenarios:
    * If there are cars with yellow or green colors, you turn red. You will start negotiation after current negotiation ends and Duckiebots involved in the current negotiation pass the intersection.
    * If there is no green or yellow lights, but there is at least one Duckiebot with red color, you wait until you see a green or yellow color.  You turn red. You go to 2 a).
    * If no lights are present, you turn yellow and wait for t seconds. If after t seconds  there are still no lights on or all lights are red, you enter the intersection.
    * If there is at least one vehicle with yellow lights, you turn green (you need to negotiate with yellow), go to 2 b).


2. 
    1. You are currently red. Wait until yellows and/or greens are gone (assuming we also know their spots: opposite, left, right), check if there are any other reds:
        * If no reds are present, turn yellow and enter the intersection (you can execute immediately because you were red in the beginning of this step so if there are no reds when you checked, Duckiebots should be waiting for you o turn yellow or green so that they can turn red, therefore there cannot be a synchronization problem here).
        * If reds, turn green and go to 2 b) (if some reds turned green too quickly so that one red was unable to catch the red color, it still knew the spots of greens/yellow that existed in the beginning of 2 a), so if you see greens in the new spots can, you can still turn green and go to 2b).
    2. You are currently green. Wait for random (0.2, 1) seconds:
        * If there exists a yellow negotiator wait for t seconds and repeat 2 b).
        * Else (if all negotiators show green), turn yellow, wait for t seconds.
        * If still all negotiators show green, enter the intersection.
        * If there is a yellow negotiator wait for t seconds and repeat 2 b).


(after the above algorithm, we could also implement the below to increase throughout)


Back lights: If you are entering the intersection, turn back lights to green. Else (negotiating, or waiting), turn off back lights.

Duckiebots behind a Duckiebot that stopped: If see a green back light, get ready to run, turn front lights to yellow and immediately enter the intersection after the car in front of you. If no green light on the back of the Duckiebot in front of you, stop at the red line as usual and start executing the intersection algorithm.

Improvements: We could possibly give frequency to back lights to let more than 2 Duckiebots enter the intersection, such that the first Duckiebot that is about to enter the intersection has constant green lights on the back, 2nd one behind it has green back lights with frequency x, and 3rd behind 2nd one knows it is the 3rd since it sees green light with frequency so it also enters the intersection immediately (and has lights off on the back so that 4th has to stop at the red line and execute the default intersection algorithm).

Therefore, this alternative needs 3 different signals, such as green, red, yellow and it will also use the no color case as a 4th signal.

*Alternative approach 3*:

This approach consists of is an exponential backoff model (assuming that two other visible cars is a low enough number to have it run quickly enough). Here:

* No turning is needed.
* If you see a Duckiebot at the right, give it “right of way”.

The intersection policy works as follows:

START:

* go to CHECKING (blue)

CHECKING:

* if a bot is “out of place” (in the intersection)
    * go to WAIT (red)
    * go to CHECKING
* if no Duckiebots in CHECKING,
    * GO (green)
* if Duckiebot at right or front is CHECKING
    * go to WAIT (red)
    * go to  CHECKING

WAIT:

* exponential backoff



### Functionality-resources trade-offs

#### Functionality provided

Metrics for LEDs communication:

* Maximize percentage of success in detecting a LED light or LED blinking in a picture taken from the Duckiebot camera.
* Maximize percentage of failed attempt of communication in a Duckietown intersection.
* Minimize time needed in order to detect signals.

Metrics for coordination:

* Maximize the times the intersection is cleared safely, i.e., without crashing.
* Minimize the time needed to clear the intersection for each Duckiebot and for the fleet.
* Maximize the number of successful intersections cleared safely below a threshold time (which may depend on the intersection itself and on the number of Duckiebot at the intersection).


#### Resources required / dependencies / costs

The following resources are needed in order to test the behaviour:

* Four Duckiebots are needed to test the coordination.
* A Duckietown intersection.
* Traffic lights.


#### Performance measurement

Performance measurements for LEDs communication:

* We put one Duckiebot in a lane at the intersection, one Duckiebot on the opposite lane across the intersection, and then one on the right. First the opposite and then the right Duckiebot will emit signals, the observer will receive them. We will see what happens if we let LED-detector-node run. We should be able to detect in which regions are LEDs blinking (See f23-presentation, Minute 11:20 in Google Drive).

* We put four Duckiebots at an intersection (with and without traffic light) and let them communicate. This allows to test whether the communication was successful or failed and the time needed to perform it.


Performance measurements for coordination:


* First, test the algorithm used to clear the intersection with computer simulations to see if there are any theoretical problems with the algorithm itself.
* After having an algorithm that computes a good “plan” to clear the intersection, we run it x times and see how many times the intersection is cleared safely.
* We measure how many seconds it takes to clear the intersection.


## Part 3: Preliminary design

#### Modules

Following subprojects are detected:

1. LED communication

    1. Duckiebot A emits a signal encoded in LED;
    2. Duckiebot B detects the signal from Duckiebot A;
    3. Duckiebot B interpret signal from Duckiebot A.

2. Coordination at intersection
    1. Each single Duckiebot computes “the plan” to clear the intersection;
    2. Each Duckiebot leave the intersection according to “the plan”.

#### Interfaces

1. For the LED communication:
    1. Led_emitter: Input for the emission is the state of Duckiebot A (waiting, entering, or navigating the intersection). The output is the corresponding LED signal.
    2. Led_detection: Input is the image of the camera. The output is the frequency/color/etc of the detected LED(s).
    3. Led_interpreter: Input is the frequency/color/etc of the detected LED. The output is the corresponding message.
2. For the coordination:
The inputs are the type of intersection (with or without traffic light and number of streets) as well as the interpretation of the signals emitted by the other uckiebots. The output is decision on when to go, taken accordingly to the coordination policy.



### Preliminary plan of deliverables


#### Specifications
The Duckietown specification do not need to be revisited.

#### Software modules

The software will be organized as follows:

* One ROS node for the emission.
* One ROS node for the detection.
* One ROS node for the interpretation.
* One ROS node for the coordination.

The existing code has already a similar structure, meaning that part of the code might be reused.


#### Infrastructure modules

No infrastructure modules are needed.

## Part 4: Project planning

### First steps for the next phase

We are going to implement and test the alternative coordination algorithms along with the last year’s algorithm and check safety and performance metrics.


<col3 figure-id="tab:timetable" figure-caption="Timetable">
<span>Week</span>
<span>Task</span>
<span>Expected Outcome</span>
<span>12.11.17-
19.11.17</span>
<span>Kick-off
Hardware up to date
Specifications for other groups
First tests</span>
<span>Preliminary design</span>
<span>20.11.17-
26.11.17</span>
<span>Test existing code
Implementing new algorithms</span>
<span>Benchmark results
First implementation</span>
<span>27.11.17-
03.12.17</span>
<span>Implementing new algorithms continues
Benchmark the new algorithms</span>
<span>Performance of the new algorithmsDeadline for Chicago</span>
<span>04.12.17-
10.12.17</span>
<span>Try more sophisticated implementations such as algorithms that include communication with the bots behind, through back lights</span>
<span>Implementation</span>
<span>11.12.17-
17.12.17</span>
<span>Benchmark new algorithms</span>
<span>Benchmark results</span>
<span>18.12.17-
24.12.17</span>
<span>TBD</span>
<span>TBD</span>
<span>25.12.17-
31.12.17</span>
<span>TBD</span>
<span>TBD</span>
<span>01.01.18-
07.01.18</span>
<span>TBD</span>
<span>End of the project</span>
</col3>

#### Data collection

* Test what is seen at different initial orientations. This will be employed as specifications for the controller group and for the smart city group.
* Run experiments, take pictures, and screenshots for both LEDs detection and and intersection coordination.


#### Data annotation

No data needs to be annotated.

#### Relevant Duckietown resources to investigate

These are some of the important features used in the past course:

* Produce LED signal (protocol for emitting signal) - led_emitter.
* Detect LED blinking signal from camera - led_detection.
* Interpreting signaling data includes - led_interpreter.
    1. Determining what kind of signal we have (is it a Duckiebot? A traffic light? Something unwanted?).
    2. knowing how many other cars at intersection (A, B, C) and if there is a traffic light up (present/ absent).
* Coordination policy and ros node from last year.
* MIT 2016 presentation.


#### Other relevant resources to investigate

Literature and codes from the class of 2016 at MIT. Additionally, research papers in computer vision (How to detect LEDs? How to use the camera? Etc.) and coordination (how to optimally coordinate a fleet? Etc.) might be of interest.


### Risk analysis

The following risks for LEDs communication are identified:

* Detecting colors. A possible solution would be to test colors and frequency to see which one is more robust.

* Limited visibility. Slightly turning in place (and then coming back to the initial position) might alleviate this problem.

The following risks for coordination are identified:

* Synchronization problems when Duckiebots arrive at the intersection at different times and when they detect signals from other Duckiebots while Duckiebots are changing signals.

We can summarize the risks in the following table.


<col7>
<span>Risk</span>
<span>Potential error</span>
<span>Consequence</span>
<span>Likelihood</span>
<span>Impact</span>
<span>Risk Priority Number
(0-100)</span>
<span>Actions required</span>
<span>Detecting color</span>
<span>Camera does not recognize colors</span>
<span>Signal cannot be encoded in color</span>
<span>7</span>
<span>9</span>
<span>63</span>
<span>Alternative interpreter strategy.</span>
<span>Visibility</span>
<span>Limited visibility from DB camera</span>
<span>Limited exchange of messages</span>
<span>10</span>
<span>8</span>
<span>80</span>
<span>Strategy that does not need information from every DB</span>
<span>Synchronization before messages</span>
<span>Problems when bots arrive at the intersection at different times</span>
<span>Wrong communication</span>
<span>3</span>
<span>10</span>
<span>30</span>
<span>Take into account while formulate strategy. Safety procedure needed.</span>
<span>Synchronization during messages</span>
<span>Problems when bots detect signals from other bots while bots are changing signals. Problems if bot detects signals while other bots are changing signals</span>
<span>Wrong communication</span>
<span>3</span>
<span>10</span>
<span>30</span>
<span>Safety procedure for this case.</span>
</col7>
