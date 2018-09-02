#  Group name: intermediate report {#template-int-report status=ready}

_It's time to commit on what you are building, and to make sure that it fits with everything else._

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

Please note that for this part it is necessary for the system architect and software architect to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time.

### Logical architecture

- Please describe in detail what the desired functionality will be. What will happen when we click "start"?

- Please describe for each quantity, what are reasonable target values. (The system architect will verify that these need to be coherent with others.)

- Please describe any assumption you might have about the other modules, that must be verified for you to provide the functionality above.

<!--
The above must have a check-off by the software architect:

System architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

### Software architecture

- Please describe the list of nodes that you are developing or modifying.

- For each node, list the published and subscribed topics.

- For each subscribed topic, describe the assumption about the latency introduced by the previous modules.

- For each published topic, describe the maximum latency that you will introduce.

<!--
The above must have a check-off by the software architect:

Software architect check-off: I, XXX, (agree / do not agree) that the above is compatible with system-level constraints.
-->

## Part 2: Demo and evaluation plan

_Please note that for this part it is necessary for the VPs for Safety to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Demo plan

The demo is a short activity that is used to show the desired functionality, and in particular the difference between how it worked before (or not worked) and how it works now after you have done your development.

It should take a few minutes maximum for setup and running the demo.

- How do you envision the demo?

- What hardware components do you need?

### Plan for formal performance evaluation

- How do you envision the performance evaluation? Is it experiments? Log analysis?

In contrast with the demo, the formal performance evaluation can take more than a few minutes.

Ideally it should be possible to do this without human intervention, or with minimal human intervention, for both running the demo and checking the results.

<!--
Check-off by Duckietown Vice-President of Safety:

Duckietown Vice-President of Safety: I, (believe / do not believe) that the performance evaluation above is
-->
## Part 3: Data collection, annotation, and analysis

_Please note that for this part it is necessary for the Data Czars to check off before you submit it. Also note that they are busy people, so it's up to you to coordinate to make sure you get this part right and in time._

### Collection

- How much data do you need?

- How are the logs to be taken? (Manually, autonomously, etc.)

Describe any other special arrangements.

- Do you need extra help in collecting the data from the other teams?

### Annotation

- Do you need to annotate the data?

- At this point, you should have you tried using [thehive.ai](https://thehive.ai/) to do it. Did you?

- Are you sure they can do the annotations that you want?

### Analysis

- Do you need to write some software to analyze the annotations?

- Are you planning for it?

<!--
Check-off by Data Zars:

Data czars check-off: We, XXX and YYY, (believe / do not believe) that the plan above is well structured, and that we can provide the level of support requested.
-->
