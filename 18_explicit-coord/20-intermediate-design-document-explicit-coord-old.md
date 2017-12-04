#  Explicit Coordination: Intermediate Report {#template-int-report status=ready}


This consists of 3 parts:

- Part 1: System interfaces: Does your piece fit with everything else? You will have to convince both system architect and software architect and they must sign-off on this.

- Part 2: Demo and evaluation plan: Do you have a credible plan for evaluating what you are building? You will have to convince the VPs of Safety and they must sign-off on this.

- Part 3: Data collection, annotation, and analysis: Do you have a credible plan for collecting, annotating and analyzing the data? You will have to convince the data czars and they must sign-off on this.


<div markdown="1">

 <col2 id='checkoff-people-intermediate-report' figure-id="tab:checkoff-people-intermediate-report" figure-caption="Intermediate Report Supervisors">
    <s>System Architects</s>                         <s>Sonja Brits, Andrea Censi</s>
    <s>Software Architects</s>                       <s>Breandan Considine, Liam Paull</s>
    <s>Vice President of Safety</s>                  <s>Miguel de la Iglesia, Jacopo Tani</s>
    <s>Data Czars</s>                                <s>Manfred Diaz, Jonathan Aresenault</s>
 </col2>

</div>

## Part 1: System interfaces


### Logical architecture

Our job starts when Duckiebots are stationary at red-line of the intersection.
By clicking “start” LED-coordination-node tells LED-emitter-node to turn LEDs white for all Duckiebots.

Afterwards, the LED-detector-node will check for each Duckiebot if other LEDs are seen and tells it to the LED-coordination-node.

The LED-coordination-node will for each Duckiebot estimate the coordination move (either “hold on” or “go”). The final output will be this signal that will be used by the Navigators to start the procedure to clear the intersection.  After that we are not going to intervene until the Duckiebot finds itself at another intersection.
 
Our led-detection, led-emission, and led-coordination nodes will affect only the Duckiebots behaviour at intersection. Surely, our LED-signal could be seen from other Duckiebots in Duckietown but for now no group need LEDs to communicate in other situations. 

However, a LED-signal will be used by fleet-planning to indicate the status of each vehicle (free, occupied, waiting,..). Fleet planning will use one LED for implementing this functionality (back-right one) while the other LEDs remain available for coordination purposes.

The following assumptions are made about other modules:

1. When the Duckiebot is made to stop at the red line by the Controllers at an intersection a flag “at_intersection” will be set and that is when the coordination will start.

2. Controllers guarantee that the Duckiebots will stop at the red line within the agreed tolerances (i.e. Min. 10cm behind center of red line;  Max. 16cm behind center of red line; +/- 10° of rotation ; +/- 5 cm offset from center of the driving lane.).

3. Fleet planning and neural-SLAM are the ones responsible to give information about where the Duckiebots should go at the intersection (information that will NOT be used in any case for determining how the intersection will be cleared);

4. Navigators will take over once the Duckiebot will receive the order that it can proceed to navigate the intersection (a signal “go”), moment from which our team, explicit-coordination, will no more intervene. 

5. If the Fleet planning and neural-SLAM decision is not available, the Navigators are responsible to generate a random choice for the direction that each Duckiebot will have to follow in the intersection navigation, once again, the direction that the Duckiebot will take is not of interest for the coordination part that is performed regardless of this information.

6. Explicit coordination and implicit coordination will never run at the same time on a Duckiebot. 


<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

Nodes:

1. LED-coordination
    * Input:     From group ? ”Red line is intersection”     
    * Output: Duckiebot move (“go”/ ”not go”)
    * Subscribed topic: from group ? 
        *string message: inters_yes/ inters_no
    * Published topic: 
        * string message: go/ no_go

2. LED-emitter
    * Input: Communication is needed
    * Output: LED turn on
    * Subscribed topic: from LED-coordination
        * string message: on/ off
    
3. LED-detection
    * Input: camera_image
    * Output: LED detected/ LED not detected
    * Subscribed topic: from LED-coordination
        * string message: detect/ no_detect
    * Published topic: to LED-coordination
        * string message: LED_detected/ no_LED_detected

A diagram of our nodes is shown below:


<div figure-id="fig:Nodes" figure-caption="Nodes">
     <img src="nodes.png" style='width: 80ex; height: auto'/>
</div>

The initial pose of Duckiebot have to fulfill following requirements (given to The Controllers):

* Min. 10cm behind center of red line;
* Max. 16cm behind center of red line;
* +/- 10° of rotation;
* +/- 5 cm offset from center of the driving lane. 



<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan


### Demo plan

Our demo will be conceptually similar to the MIT2016 “openhouse-dp5”,  available from last year. The Duckiebots that are navigating in Duckietown, will stop at the red line and LED-communication and coordination will be performed leading to the eventual clearing of the intersection.

From testing last years’ code we realized that the coordination does not seem to work with the mentioned demo. Duckiebots stop at the red line but they do not communicate so that they never leave the intersection or decide to go independently of the presence and decision of the other Duckiebots. 

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
* Mean clearing time
* Success rate 

Both for each case separately and for all combined.</span>
<span>LED emitter and LED detector</span>
<span>Run the specific nodes</span>
<span>Two to three Duckiebots in an intersection configuration (all relative positions have to be analyzed)</span>
<span>Number of unsuccessful LED emissions (the output has not the desired colour and/or frequency) and number of unsuccessful LED detections: the output of the LED detector does not contain all the signals it should have detected or it contains more than the ones effectively present (false positives)</span>
<span>
* Success rate 
For both LED emission and detection.</span>
<span>LED detector</span>
<span>Run the specific node</span>
<span>Two to three Duckiebots in an intersection configuration (all relative positions have to be analyzed)</span>
<span>Time required to perform the LED detection</span> 
<span>
* Mean detection time</span>
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
