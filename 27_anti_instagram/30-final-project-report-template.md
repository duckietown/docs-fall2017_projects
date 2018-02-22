#  Anti Instagram: final report {#template-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

_The objective of this report is to bring justice to your hard work during the semester and make so that future generations of Duckietown students may take full advantage of it. Some of the sections of this report are repetitions from the preliminary and intermediate design document (PDD, IDD respectively)._

## The final result {#template-final-result}

_Let's start from a teaser._

* Post a video of your best results (e.g., your demo video)

Add as a caption: see the [operation manual](#demo-template) to reproduce these results.

## Mission and Scope {#template-final-scope}

_Now tell your story:_

 --> Define what is your mission here.  




### Motivation {#template-final-result-motivation}

_Now step back and tell us how you got to that mission._

- What are we talking about? [Brief introduction / problem in general terms]

- Why is it important? [Relevance]

During the process of autonomous driving the Duckiebot has to know where to drive and where not. For example the road marking or obstacles on the road give constraints where the allowable area to drive is.  
Let's take the road marking example. For Duckietown it is true that the road is rather black, the stopping lines are red, the side lines are white and the dashed middle lines are yellow.  
Knowing about this color information the Duckiebot is theoretically able to know whether it is on the correct lane position or not. By detecting the lines and estimating the position the Duckiebot can know whether it should drive more left or right or stay in the same position.  
This is only possible if we are assuming the line detection is done perfectly.  
Bus the line detection and especially the classification is done with the color information.  
The problem now is that the color can vary due to illumination variation. So the classification can fail and therefore the performance suffers.  
To sum up the lower the variance in the color of the lines the better the classification and the better the autonomous driving performance of the Duckiebot.

### Existing solution {#template-final-literature}

- Was there a baseline implementation in Duckietown which you improved upon, or did you implemented from scratch? Describe the "prior work"  


Before we started our project a solution was already implemented. The existing approach was estimating a color transformation by using k-Means algorithm with 3 centers.  
First "true colors" were defined. These "true colors" should represent the best colors for the lines. So if the road marking lines would be in these predefined red, yellow and white the color classification would work the best.  
So this approach took all the pixels of the camera image and estimated three centers with the k-Means algorithm.  
Afterwards the three found centers were compared with the previously defined "true colors". The difference of the centers found by k-Means and the "true colors" lead to the color transformation.  
Maybe you can better imagine the procedure as follows:  
You take all the pixels and their RGB values. Then you plot each pixel in your imaginary R,G,B coordinate system.  
The k-Means algorithm now tries to detect clusters in this RGB space and estimates the center of these clusters. The centers of these clusters are now compared to the "true centers" which is the location of the optimal red for example in the RGB space. This leads to a transformation which is applied to every image from the camera of the Duckiebot.



### Opportunity {#template-final-opportunity}

#### Advantages of existing solution

The idea of estimating a color transformation from a captured image based on estimated and true centers is very promising since it really focuses on transforming the colors.  
Often other image transformations focus on white balance. But we are concerned the most of the colors. So this clustering approach is a good idea here.

#### Disadvantages of existing solution

1. The k-Means clustering was initialized only with 3 centers. This is a very rough guess. By analyzing several sample images one sees that there are distinct white, red and yellow clusters. They can indeed be represented by three distinct centers. But the problem is that all the other pixels have to assigned to a cluster as well. This distorts the color transformation.
2. The existing solution was not online. The color transformation had to estimated explicitly by pressing a button on the joystick. Firstly the system is not fully autonomous anymore since it needs user input (pressing the button). And secondly the user doesn't or cannot really know when the optimal moment is for a color transformation.

--> What didn't work out with the existing solution? Why did it need improvement?

Examples:
- there wasn't a previous implementation
- the previous performance, evaluated according to some specific metrics, was not satisfactory
- it was not robust / reliable
- somebody told me to do so (/s)

* How did you go about improving the existing solution / approaching the problem? [contribution]
- We used method / algorithm xyz to fix the gap in knowledge (don't go in the details here)
- Make sure to reference papers you used / took inspiration from

### Preliminaries (optional) {#template-final-preliminaries}

We thought that a clustering approach is a very promising approach because of the following points:
- clustering algorithms are able to focus on our road marking colors since these colors are distinct.
- clustering can be done unsupervised and can learn. (?)
- clustering is stable (?)

--> Is there some particular theorem / "mathy" thing you require your readers to know before delving in the actual problem? Add links here.

Definition of link:
- could be the reference to a paper / textbook (check [here](#bibliography-support) how to add citations)
- (bonus points) it is best if it is a link to Duckiebook chapter (in the dedicated "Preliminaries" section)

## Definition of the problem {#template-final-problem-def}

_Up to now it was all fun and giggles. This is the most important part of your report: a crisp mathematical definition of the problem you tackled. You can use part of the preliminary design document to fill this section._

Make sure you include your:
- final objective / goal
- assumptions made (including contracts with "neighbors")
- quantitative performance metrics to judge the achievement of the goal

## Contribution / Added functionality {#template-final-contribution}


### k-Means Approach
We were convinced that a clustering approach is a good solution because of the following points:
1. The clustering and the consequently determined centers lead to a color transformation focusing only on the problem relevant colors.
2. can be implemented unsupervised

### Histogram Equalization


Describe here, in technical detail, what you have done. Make sure you include:
- a theoretical description of the algorithm(s) you implemented
- logical architecture (refer to [IDD template](#template-int-report) for description)
- software architecture (refer to [IDD template](#template-int-report) for description)
- details on the actual implementation where relevant (how does the implementation differ from the theory?)
- any infrastructure you had to develop in order to implement your algorithm
- If you have collected a number of logs, add link to where you stored them

_Feel free to create subsections when useful to ease the flow_

## Formal performance evaluation / Results {#template-final-formal}

_Be rigorous!_

- For each of the tasks you defined in you problem formulation, provide quantitative results (i.e., the evaluation of the previously introduced performance metrics)
- Compare your results to the success targets. Explain successes or failures.
- Compare your results to the "state of the art" / previous implementation where relevant. Explain failure / success.
- Include an explanation / discussion of the results. Where things (as / better than / worst than) you expected? What were the biggest challenges?

## Future avenues of development {#template-final-next-steps}



### Two step k-Means
Do first transformation with n iterations and k centers. Remember the k centers. For the next 2, ..., z images only start from the \[z_{i-1}\] to compute image $$z_{i}$  

### Other clustering method
Gaussian mixture models, the cluster

### white paper reference
include a white paper as a reference to estimate the best moment for executing the color correction algorithm.

### polarize camera
remove reflections. often a problem

_Is there something you think still needs to be done or could be improved? List it here, and be specific!_
