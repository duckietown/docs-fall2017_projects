#  Explixit Coordination: preliminary design document {#project-name-preliminary-design-doc status=ready}

<!-- EXAMPLE COMMENT
-->

## Part 1: Mission and scope

### Mission statement

Coordinate intersection navigation safely through explicit communication.

### Motto

<div class='check' markdown="1">

IN HOC SIGNO VINCES 

</div>

### Project scope



Employ LEDs based communication to efficiently manage traffic at intersections. First, LEDs should be used to reliably communicate DBs positions and intentions . Then, optimal control theory or game theory might be considered to safely clear the intersection in reasonable time. 
By safely, we mean that no collisions occur (100% success). This requires robustness in message interpretation, which requires a correct bot positioning (constraints for the controllers).

We want to solve the problem of clearing the intersection in max. 60 seconds (meaning that even in the case of four bots participating we can solve the problem below this time).


#### What is in scope

The following aims are identified:

* List the messages that need to be exchanged between duckiebots;
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
* Implicit-navigation (determinate where the duckie in the row has to stop);
* Anti-instagram filtering;
* Integration heros;
* Map designers;
* Traffic navigation.


## Part 2: Definition of the problem

### Mission

DB must be able to cross an intersection in the defined reasonable time and without collisions.


### Problem statement

The following problems are to be tackled:

* Led-based communication; 
* Coordination at intersections.

### Assumptions

The following assumptions are made for the LEDs communication:

* The duckiebot is of type DB17-l.
* One to four duckiebots are at the intersection with a certain position and orientation with respect to the stop line (projection of duckiebot on 2D lane):
    * Min. 0 cm behind red line; 
    * Max. 6 cm behind red line;
    * Max. +-2 cm from center of the line;
    * +/- 5° of rotation.
* Duckiebots are able to see the vehicles in front and on the right with respect to their position.
* LEDs work properly and emit the signals with the right color and frequency, and we can also detect the LEDs as the correct state that they represent, and attribute those LEDs to the correct duckiebot that is displaying the LEDs.
* The duckiebots do not move while “waiting” in the intersection, but they can move on the spot to look around to see the left.
* Camera (frequency 30Hz, tradeoff with resolution).

The following assumptions are made for the coordination:

* Signals are correctly recognized and associated to the corresponding messages.
* The intersection is among one of the standard intersections of Duckietown. That is, weird intersections will be ignored (at least in a first approach to the problem).
* The intersection type and the presence of a traffic lights are known.
* Intersection navigation is guaranteed to be working safely.
* One bot navigates the intersection at the time.


### Approach

Firstly, test the existing code and see its limits. 
Design well-defined benchmark tests.

Check the literature to see what has already been implemented.

Two cases are necessary: one for the case in which a traffic light is present, the other for when it is not present. (centralized vs decentralized)

Possible approaches for LEDs communication:

* Last year’s algorithm;
* Alternative algorithm 0 (in part 3).

Possible approaches for intersection coordination:

* Last year’s algorithm;
* Alternative algorithm 1 (in part 3);
* Alternative algorithm 2 (in part 3);
* Alternative algorithm 3 (in part 3).

### Functionality-resources trade-offs

The space of possible implementations / battle plans is infinite.
We need to understand what will be the trade-offs.

### Functionality provided

Metrics for LEDs communication:

* Percentage of success in detecting a LED light or LED blinking in a picture taken from the DB camera.
* Percentage of failed attempt of communication in a Duckietown intersection.
* Time needed in order to detect signals.

Metrics for coordination:

* Maximize the times the intersection is cleared safely, i.e., without crashing.
* Minimize the time needed to clear the intersection for each bot and for the fleet.
* Maximize the number of successful intersections cleared safely below a threshold time (which may depend on the intersection itself and on the number of duckiebot at the intersection).


### Resources required / dependencies / costs

The following resources are needed in order to test the behaviour:

* Four duckiebots are needed to test the coordination. 
* A Duckietown intersection. 
* Traffic lights.


### Performance measurement

Performance measurements for LEDs communication:

* We put one DB in a lane at the intersection, one DB on the opposite lane across the intersection, and then one on the right. First the opposite and then the right bot will emit signals, the observer will receive them. We will see what happens if we let LED-detector-node run. We should be able to detect in which regions are LEDs blinking.
* We put four duckiebots at an intersection (with and without traffic light) and let them communicate. This allows to test whether the communication was successful or failed and the time needed to perform it.

Performance measurements for coordination:

* First, test the algorithm used to clear the intersection with computer simulations to see if there are any theoretical problems with the algorithm itself.
* After having an algorithm that computes a good “plan” to clear the intersection, we run it x times and see how many times the intersection is cleared safely.
* We measure how many seconds it takes to clear the intersection.



## Part 3: Preliminary design

### Modules

Following subprojects are detected:

1. LED communication

    * Duckiebot A emits a signal encoded in LED;
    * Duckiebot B detects the signal from duckiebot A;
    * Duckiebot B interpret signal from duckiebot A. 

2. Coordination at intersection
    * Each single duckie computes “the plan” to clear the intersection;
    * Each duckie leave the intersection according to “the plan”.

### Interfaces

1. For the LED communication:
    * Led_emitter: Input for the emission is the state of duckiebot A (waiting, entering, or navigating the intersection). The output is the corresponding LED signal. 
    * Led_detection: Input is the image of the camera. The output is the frequency/color/etc of the detected LED(s).
    * Led_interpreter: Input is the frequency/color/etc of the detected LED. The output is the corresponding message.
2. For the coordination:
The inputs are the type of intersection (with or without traffic light and number of streets) as well as the interpretation of the signals emitted by the other duckiebots. The output is decision on when to go, taken accordingly to the coordination policy.


### Preliminary plan of deliverables

FOR MONDAY, NOVEMBER 20TH: COORDINATION ALGORITHMS

#### Last year’s intersection coordination algorithm: no traffic lights case

green: negotiating

red: someone is at the intersection

yellow: about to execute but still checking around

1. Bring ego vehicle to full stop at stop line and switch on **green** LED. Go to 2.
2. Go through the following cases:
    1. If there is at least one car showing a red LED. Wait until that LED is turned green and then go back to 2.
    2. If there is at least one car in the field of view with its yellow light on:
        * Wait for at least t'
        * Check if yellow light is still on: If yes, turn on red LED and wait for P seconds. Then turn on green LED and go back to 2. Otherwise, go back to 2.

    3. If no vehicle is on the right, no opposing vehicle is present or has a green light, and no vehicle is currently on the intersection / both the opposing vehicle and the right vehicle have green LED / only the opposing vehicle is present, switch on yellow LED and wait for t'
If during t' all visible cars have green LED or LED Off, then go to 3.
Otherwise change LED color to green, choose a random waiting time t* in [0.2, 1] s (This interval should be calibrated) and go to 2.
    4. If only the right vehicle is present and has either no light or green light, do nothing and go back to 2.

3. Change to intersection driving mode, keep light on until the exit of the intersection

#### Alternative algorithm 1

DB does not see on the left.
If DB does not see nobody on the right, then he will enter the intersection.
That could happen with 1, 2, 3 DB at intersection.

If 4 DB are at intersection, we turn down one randomly (25% probability), then one do not see nobody on the right and will start.

Messages to be sent:

* 1 signal to say: I am here
* 1 signal to say: i am going
* 1 signal to say: I sees someone in front and on the RHS. 

#### Alternative algorithm 2 (idea is that not everyone immediately joins the negotiation): no traffic lights case

No color: waiting to enter the red queue

red: waiting to enter the negotiation

yellow: about to enter the intersection but still looking around, can go back to green

green: negotiating

you = DB

1. (you currently have no color) you stop at the intersection, turn left 10° - need to test how many degrees - so that you can see all bots if they exist, you have no lights on at the moment.
    * If there are cars with yellow or green colors, you turn red, meaning you will start negotiation after current negotiation ends and bots in the current negotiation pass the intersection
    * If there is no green or yellow lights, but there is at least one bot with red color, you wait until you see a green or yellow color to turn red, when you turn red go to 2a
    * If no lights, you turn yellow and wait for t seconds
        * if still no lights on or all red, you enter the intersection
        * if there is at least one vehicle with yellow lights, you turn green meaning you need to negotiate with yellow, go to 2b

2. 
    1. (you are currently red) Wait until yellows and/or greens are gone (assuming we also know their spots: opposite, left, right), check if there are any other reds
        * If no reds, turn yellow and enter the intersection (you can execute immediately because you were red in the beginning of this step so if there are no reds when you checked, bots should be waiting for you to turn yellow or green so that they can turn red, therefore there cannot be a synchronization problem here)
        * If reds, turn green and go to 2b (if some reds turned green too quickly so that one red was unable to catch the red color, it still knew the spots of greens/yellow that existed in the beginning of 2a, so if you see greens in the new spots can, you can still turn green and go to 2b)
    2. (you are currently green) wait for random (0.2, 1) seconds
        * If there exists a yellow negotiator wait for t seconds and repeat 2b
        * Else(if all negotiators green), turn yellow, wait for t seconds 
            * if still all negotiators green, enter the intersection
            * if there is a yellow negotiator wait for t seconds and repeat 2b


(after the above algorithm, we could also implement the below to increase throughput)
Back lights:

* If you are entering the intersection, turn back lights to green
* Else(negotiating, or waiting), back lights off

Bots behind a bot that stopped:

* If see a green back light, get ready to run, turn front lights to yellow  and immediately enter the intersection after the car in front of you. 
* If no green light on the back of the bot in front of you, stop at the red line as usual and start executing the intersection algorithm
    
*we could possibly give frequency to back lights to let more than 2 bots enter the intersection, such that the first bot that is about to enter the intersection has constant green lights on the back, 2nd one behind it has green back lights with frequency x, and 3rd behind 2nd one knows it is the 3rd since it sees green light with frequency so it also enters the intersection immediately(and has lights off on the back so that 4th has to stop at the red line and execute the default intersection algorithm)

Therefore, this alternative needs 3 different signals, such as green, red, yellow and it will also use the no color case as a 4th signal. 

####Alternative algorithm 3 (That does not provide “correct” behavior according to traffic laws but is potentially fast)

This is just an exponential backoff model (assuming that 2 other visible cars is a low enough number to have it run quickly enough)

* No turning needed.
* if you see a bot at the right, give it “right of way”



**START:**
 
* go to CHECKING (blue)

**CHECKING:**

* if a bot is “out of place” (in the intersection)
    * go to WAIT (red)
    * go to CHECKING
* if no bots in CHECKING,
    * GO (green)
* if bot at right or front is CHECKING
    * go to WAIT (red)
    * go to  CHECKING

**WAIT:**
    * exponential backoff


**What needs to be designed? What needs to be implemented? What already exists and needs to be revised?**

LED emitter, LED receiver and LED interpreter are coded from MIT 2016. We are going to check them out.
Possible implementation for LED emitter and receiver could include detect colors and not just detect blinking.
Possible implementation for LED interpreter could be completely based on camera detection of colors/ edges (contrast)/… instead of frequency of blinking interpreted with FFT.


### Specifications

The Duckietown specification do not need to be revisited. 

### Software modules

The software will be organized as follows:

* One ROS node for the emission.
* One ROS node for the detection.
* One ROS node for the interpretation.
* One ROS node for the coordination.


### Infrastructure modules

No.

## Part 4: Project planning

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

### Data collection

* Test what is seen at different initial orientations. This will be employed as specifications for the controller group and for the smart city group.
* Run experiments, take pictures, and screenshots for both LEDs detection and and intersection coordination.


### Data annotation

No data needs to be annotated. 

#### Relevant Duckietown resources to investigate

These are some of the important features used in the past course:

* Produce LED signal (protocol for emitting signal) - led_emitter
* Detect LED blinking signal from camera - led_detection
* Interpreting signaling data includes - led_interpreter 
    1. Determining what kind of signal we have (is it a DB? A traffic light? Something unwanted?)
    2. knowing how many other cars at intersection (A, B, C) and if there is a traffic light up (present/ absent)
* Coordination policy and ros node from last year
* MIT 2016 presentation 


#### Other relevant resources to investigate

Literature and codes from the class of 2016 at MIT. Additionally, research papers in computer vision (How to detect LEDs? How to use the camera? Etc.) and coordination (how to optimally coordinate a fleet? Etc.) might be of interest.

### Risk analysis

The following risks for LEDs communication are identified:

* Detecting colors. A possible solution would be to test colors and frequency to see which one is more robust.
* Limited visibility. Slightly turning in place (and then coming back to the initial position) might alleviate this problem.

The following risks for coordination are identified:

* Synchronization problems when bots arrive at the intersection at different times and when they detect signals from other bots while bots are changing signals. 
Summarized in a table:

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

