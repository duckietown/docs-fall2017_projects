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


In order to guarantee the success in any condition, it makes sense to have a decentralised system, where each Duckiebot can operate on its own, independently. Given an intersection, there are two ways to coordinate Duckiebots:

- Using a traffic light
- Using a communication protocol between the vehicles

### Existing solution {#explicit-coord-final-literature}

A prior implementation for intersection coordination was already available from the 2016's MIT class. The principle was simple: when the Duckiebot comes at an intersection, it stops at the red line and is able to detect with the april tags if there is or not a traffic light. In the case with a traffic light, the Duckiebot detects the frequency at which the traffic light is blinking and acts based on the road rules. Otherwise, the Duckiebot detects the frequency at which the other bots are blinking and adjusts its emitted frequency depending on its state.

Two modules can be distinguished:

- LED emission and detection
- Coordination based on the detected signals


For the emission, three signals can be produced: each signal is encoded with a specific color and frequency. While Duckiebots are designed to recognise only frequencies, color are used to allow humans to easily understand the signals emitted by the Duckiebots. The signals represent the states: negotiation (and in which phase of the negotiation it is) or navigation of the intersection.

For the detection, the position and frequency of blinking LEDs are registered.

For the coordination, with the assumption that each vehicle can only see other vehicles on its right but not on its left, the Duckiebot yields its position if the only visible car is on the right, otherwise the Duckiebot waits (light green or red) or crosses (yellow light).



### Opportunity {#explicit-coord-final-opportunity}

The existing solution had essentially two drawbacks:

- The overall success rate was about 50%: the algorithm for LED-detection, and/or coordination failed, leading to a potential crash of the Duckiebots.
- In case of success, the LED-detection and/or coordination algorithms required an average of four minutes to clear an intersection with four Duckiebots. This results in an extremely slow process, which would block a city with dozens of Duckiebots.

Although the solution was problematic, it still gave us some important intuitions on how to solve the problem.

First of all, using a LED-communication protocol is a brilliant idea to let the Duckiebots communicate with each other. Since the previous communication algorithm had the only disadvantage of being slow (also because of several bugs), we started by re-thinking the coordination algorithm.
The existing implementation for the coordination was rather complex and articulated, resulting in confused strategies which led to the failure rate of 50%. In order to develop a simpler and lighter algorithm we took inspiration from an existing media access control protocol (MAC): the so called Carrier Sense Multiple Access (CSMA, https://en.wikipedia.org/wiki/Carrier-sense_multiple_access). This algorithm gave us the basic idea behind our strategy and allowed us to have a lighter protocol.

In the second place, we re-designed the LED-detection/-interpreter to be faster and more efficient based on the detection of blobs rather than frequencies.

Lastly, we wanted to have a demo that could deal with both intersections, i.e. with and without a traffic light, as opposed to the two available demos from MIT 2016's class.


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
- The Duckiebots do not crash during the navigation.

### Assumptions

####Functional assumptions

The following assumptions are made:

* The Duckiebot is of type DB17-l, i.e. has LEDs mounted on it.
* Camera works properly ( frequency 30Hz, resolution 64x48)
* LEDs work properly and emit the signals with the correct color and frequency.
* Duckiebots are able to see the vehicles in front and on the right with respect to their position: one cannot assume that the left visual is clear.
* The Duckiebots do not move while “waiting” at the intersection.
* One Duckiebot navigates the intersection at the time.
* The intersection is among one of the standard intersections of Duckietown, detected through april tags.

#### Assumptions on other groups

The following assumptions are made:

* The Controllers: one to four Duckiebots are at the intersection with a certain position and orientation with respect to the stop line. Responsible for this assumption are The Controllers, which should guide the Duckiebot at intersection and make it stops with following pose:
    - Min. 0 cm behind the stop red line;
    - Max. 6 cm behind the stop red line;
    - Max. +/- (left/right deviations) 2 cm from the center of the line;
    +/- 10° of rotation with respect to the perpendicular line of the red line.
* Navigators: as soon as coordination is decided, Duckiebots have to navigate through the intersection. This task is accomplished by The Navigators.
* Implicit coordination: it is assumed that explicit and implicit coordination are never running at the same time.



### Performance metrics

The metrics that are going to be used to judge the achievement of the goal are two:

1. Mean clearing time

2. Success rate

Our goal is to work out a procedure so that the Duckiebots cross an intersection efficiently, therefore the performance metrics follow naturally.
The clearing time should not exceed one minute and the success rate should be higher than 70% in all intersection configurations (1, 2, 3 vehicles, with or without a traffic light).


## Contribution / Added functionality {#explicit-coord-final-contribution}


_2 subsections 1) detector (Nicolas) 2)coordination (Gioele) 3)demo, 1 sola per il db + modifiche a quella del TL_

### LEDs Detection

LEDs are modeled as blobs in an image: A Blob is a group of connected pixels in an image that share some common property, which, in the case of LEDs, is the intensity of the pixel. Among the many ways to detect blobs, the one that turned out to be more robust for our purpose was the following algorithm (https://www.learnopencv.com/blob-detection-using-opencv-python-c/):
- Thresholding: Convert the source images to several binary images by thresholding the source image with thresholds.
- Grouping: In each binary image, connected white pixels are grouped together.  Let’s call these binary blobs.
- Merging: The centers of the binary blobs in the binary images are computed, and blobs located closer than a threshold are merged.
- Center and Radius Calculation:  The centers and radii of the new merged blobs are computed and returned.
- Filtering: The blobs are filtered by size and shape.

To increase the robustness of the detection, a sequence of $N$ images in analyzed. Each blob $b$ is characterized with a position vector $x_b\in\mathbb{R}^2$ and a signal $y_b\in\{0,1\}^N$ of dimension, indicating whether the blob was detected in the image (1) or not (0). The blobs found are collected in a set called $B$. The algorithm works as follows:
- Initialization: Initialize the set of blobs $B$ with the empty set.
- Recursion: For image $i$ and blob $j$ (with position $x_{ij}$):
   + If $\Vert x_{ij} – x_b\Vert > \mathrm{TOL}$ for all $b\in B$ the blob is added to $B$ with $x=x_{ij}$ and $y=e_i$, where $e_i$ is a vector of dimension $N$ whose $i$ entry is 1 and all other entries are 0. 
   + If $\Vert x_{ij} – x_b\Vert \leq \mathrm{TOL}$ for some $b\in B$. Then, we “merge” blob $j$ with the blob it was closest to, i.e., we set the $i$-th entry of $y_{\bar b}$ to 1, where $\bar b=\arg\min_{b\in B}\Vert x_{ij}-x_b\Vert$. That is, we store the information that the blob has been observed in the $i$-th image.
   
After this procedure, the user has full information about the blobs in the images. Then, one may identify the presence of another Duckiebot as follows:
- Analyzing the frequency spectrum of the signal y of each blob. If the known emission frequency and the detected frequency using the Fast Fourier Transform match, then we can conclude that a car has been detected.
- Using some heuristics. For instance, for each blob one may compute
\[
m_b=\frac{1}{N}\sum_{i=1}^N [y_b]_i
\]
and act upon this number.
This algorithms is run three times: To detect Duckiebots on the right, to detect Duckietbots on the left, and to detect traffic lights. To increase the robustness and reduce the computational demand, the image is cut accordingly. Hence, the output of the algorithm are three Booleans indicating the detection on the right, on the front, and for the traffic light respectively.  

### Coordination



### Implementation



Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_

## Formal performance evaluation / Results {#explicit-coord-final-formal}

<col4 figure-id="tab:Performance" class="labels-row1">
  <figcaption>Performance Evaluation</figcaption>

  <span>Situation</span>
  <span>Performance measure</span>
  <span>Required</span>
  <span>Obtained</span>


  <span>One Duckiebot at the intersection</span>
  <span>Clearing time</span>
  <span>60s</span>
  <span>!!!!missing </span>

  <span>One Duckiebot at the intersection</span>
  <span>Success rate</span>
  <span>90%</span>
  <span>!!!!missing </span>

  <span>Two Duckiebots at the intersection</span>
  <span>Clearing time</span>
  <span>60s</span>
  <span>!!!!missing </span>

  <span>Two Duckiebots at the intersection</span>
  <span>Success rate</span>
  <span>80%</span>
  <span>!!!missing </span>

  <span>Three Duckiebots at the intersection</span>
  <span>Clearing time</span>
  <span>60s</span>
  <span>!!!missing </span>

  <span>Three Duckiebot at the intersection</span>
  <span>Success rate</span>
  <span>70%</span>
  <span>!!!!missing </span>

  <span>One Duckiebot at a traffic light type intersection</span>
  <span>Clearing time</span>
  <span>60s</span>
  <span>!!!!missing </span>

  <span>One Duckiebot at a traffic light type intersection</span>
  <span>Success rate</span>
  <span>90%</span>
  <span>!!!!missing </span>
</col4>

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#explicit-coord-final-next-steps}

There is room for improvement for the coordination part of this project.
Our approach, in the case of an intersection without traffic light, prioritises robustness rather than efficiency (in some cases all the Duckiebots at an intersection could turn off and restart the whole protocol again) and it is easy to imagine a scenario with an improved efficiency (tradeoff with complexity).

An idea would be to encode in the signal also the intentions of the Duckiebot and, by doing so, allow multiple Duckiebots to navigate the intersection at the same time if their directions are compatible. In fact, if two Duckiebots wanted to go straight they could move at the same time.
The clearing time could also be reduced in the case of an intersection with traffic light if the latter was able to see where the vehicles are (prevent the light to turn green in a direction where no vehicle is waiting to cross).
