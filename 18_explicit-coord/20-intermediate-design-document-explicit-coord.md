#  Explicit Coordination: intermediate report {#explicit-coordination-int-report status=ready}


## Part 1: System interfaces


### Logical architecture

Our job starts when Duckiebots are stationary at the red-line of the intersection (this is communicated to us via controllers/ parking).
By clicking “start” the LED-coordination-node tells the LED-emitter-node to turn the LEDs white for all Duckiebots.
Afterwards, the LED-detector-node checks for each Duckiebot if other LEDs are seen  and tells it to the LED-coordination-node. Note that here there is, at least in a first approach to the problem, no turning, i.e., LEDs of Duckiebots on the left are not identified.
The LED-coordination-node estimates the coordination move (either “hold on” or “go”) for each Duckiebot. The final output is a signal, named move_intersection, that will be used by the Navigators to start the procedure to navigate the intersection. Thereafter, we are not going to intervene until the Duckiebot finds itself at another intersection. Should the explicit coordination fail (for instance, because of Duckiebots not equipped with LEDs), the task of coordinating the intersection is given to the implicit coordination.

Our LED-detection, LED-emission and LED-coordination nodes affect only the Duckiebots behavior at intersection. Surely, our LED-signal could be seen from other Duckiebots in Duckietown but, at least for now, no group (except for the fleet planning group, see below) needs LEDs-based communication in other situations. A LED-signal will be used by fleet-planning to indicate the status of each vehicle (free, occupied, waiting, etc.). The Fleet planning will be using one LED for implementing this functionality (back-right one) while the other LEDs remain available for coordination purposes.

The following assumptions are made about other modules:


1. When the Duckiebot is made to stop at the red line by the Controllers at an intersection a flag “at_intersection” will be set and that is when the coordination will start. Most likely this flag will be sent out by the Parking group after it has been verified that after the intersection there is no parking.

2. Controllers guarantee that the Duckiebots will stop at the red line within the agreed tolerances (i.e., Min. 10cm behind center of red line;  Max. 16cm behind center of red line; +/- 10° of rotation ; +/- 5 cm offset from center of the driving lane.).

3. Fleet planning and neural-SLAM are the ones responsible to give information about where the Duckiebots should go at the intersection (information that will not be used in any case for determining how the intersection will be cleared).

4. Navigators will take over once the Duckiebot has received the order that it can proceed to navigate the intersection (a signal “go”), moment from which our team, explicit-coordination, will no longer intervene.  

5. If the Fleet planning and neural-SLAM decision is not available, the Navigators are responsible to generate a random choice for the direction that each Duckiebot will have to follow in the intersection navigation, once again, the direction that the Duckiebot will take is not of interest for the coordination part that is performed regardless of this information.

6. Explicit coordination and implicit coordination will never run at the same time on a Duckiebot.


<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

Nodes:

1. LED_coordination:
    * Input: From Parking group “you are at an intersection” (additionally there is a parameter that indicates whether intersections are cleared with explicit or implicit coordination)
    * Output: Duckiebot move (“go”/ ”not go”)
    * Subscribed topic:
        * flag_at_intersection from Parking group, bool message: true/ false
    * Published topic: move_intersection
        * string message: go/ no_go

2. LED_emitter:
    * Input: Communication is needed
    * Output: LED turn on or stay off
    * Subscribed topic:
        * LED_switch from LED-coordination, string message: on/ off
    * Published topics: None

3.
    1. LED_detection: Depending on the algorithm implemented:
        * Input: camera_image (possibly after anti-instagram) and message indicating whether detection is needed
        * Output: LED detected/ LED not detected
        * Subscribed topic:
            * LED_to_detect from LED_coordination, string message: yes/ no
            * camera_image from anti-instragram, CompressedImage
        * Published topic:
            * string message: LED_detected/ no_LED_detected

    2. LED_detection: second option:
        * Input: camera_image (possibly after anti-instagram)
        * Output: LED detected/LED not detected with position and/or color and/or frequency
        * Subscribed topic:
            * LED_to_detect from LED_coordination, string message: yes/ no
            * camera_image from anti-instragram, CompressedImage
        * Published topic:
            * string message: LED_detected/ no_LED_detected with position and/or color and/or frequency


A diagram of our nodes is shown below.


<div figure-id="fig:Nodes" figure-caption="Nodes">
     <img src="nodes.png" style='width: 80ex; height: auto'/>
</div>


We subscribe to the following topics:

* Corrected image with maximum assumed latency 1s;
* Flag at intersection with maximum assumed latency 1s.

The following topics are published:

* Flag go/no_go with maximum latency 60s (this is the time needed to make sure that the intersection can be navigated safely).




<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan


### Demo plan

Our demo will be conceptually similar to the MIT2016 “openhouse-dp5”,  available from last year [](#demo-coordination2016). The Duckiebots that are navigating in Duckietown, will stop at the red line and LED-communication and coordination will be performed leading to the eventual clearing of the intersection.

From testing last year’s code we realized that the coordination does not seem to work with the mentioned demo. Duckiebots stop at the red line but they do not communicate so that they never leave the intersection or decide to go independently of the presence and decision of the other Duckiebots. Although we investigated the problem by looking at separate nodes, no solution has been found yet.

We aim to have a working demo that will show an effective clearing of an intersection with a variable number of Duckiebots (1 to 4) regardless of the type of intersection (3-way or 4-way, with or without traffic lights). The intersection should be cleared in a reasonable amount of time (less than 1 min) and be robust to different initial conditions (within the specified tolerances on the pose of the robots).
The setup will be easy and quick: with a small Duckietown as shown in the figure below, up to four Duckiebots will be put on the road and the demo will be started from a laptop with no further interventions required.

The required hardware will therefore be:
A four way intersection tile (see image below, center), four three-way intersections tiles, twelve tiles with straight lines,four tiles with curved lines and four empty tiles. In total, twentyeight tiles, red, yellow and white tape as indicated in the figure below. Apriltags and all other required signals at the intersection will also be needed (standard type of Duckietown intersection) as well as a traffic light to illustrate the behaviour in a traffic light type of intersection.



<div figure-id="fig:town" figure-caption="Duckietown">
     <img src="town.png" style='width: 80ex; height: auto'/>
</div>

### Plan for formal performance evaluation

Performance will be evaluated with 3 tests:

<col5 figure-id="tab:Performance" figure-caption="Performance Evaluation">

<span>What is evaluated </span>
<span>How</span>
<span>Required</span>
<span>Collected quantities</span>
<span>Performance measure(s)</span>

<span>Coordination implementation performance</span>
<span>Run the demo</span>
<span>One to four Duckiebots at an intersection, every case has to be analyzed (three or four way intersection, with or without traffic light)</span>
<span>Time required to clear the intersection and binary variable that tells whether the intersection was effectively cleared without problems (collisions, lack of decision making, wrong detection of signals,etc.) or not.</span>
<span>
Mean clearing time or success rate - Both for each case separately and for all combined.</span>
<span>LED emitter and LED detector</span>
<span>Run the specific nodes</span>
<span>Two to three Duckiebots in an intersection configuration (all relative positions have to be analyzed)</span>
<span>Number of unsuccessful LED emissions (the output has not the desired colour and/or frequency) and number of unsuccessful LED detections: the output of the LED detector does not contain all the signals it should have detected or it contains more than the ones effectively present (false positives)</span>
<span>
Success rate - For both LED emission and detection.</span>
<span>LED detector</span>
<span>Run the specific node</span>
<span>Two to three Duckiebots in an intersection configuration (all relative positions have to be analyzed)</span>
<span>Time required to perform the LED detection</span>
<span>
Mean detection time</span>
</col5>



<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

### Collection

No data is needed to develop the algorithm. Data might be needed to test the implementation of the detection.

### Annotation


No data need to be annotated.



### Analysis

As no data annotation is needed, no software will be developed.

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
