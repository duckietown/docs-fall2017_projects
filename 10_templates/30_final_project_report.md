#  Group name: final report {#fleet-communication-final-report status=draft}

<!--
General notes:
- REMEMBER to change the "template" in the chapter labels to your group label!
-->

_The objective of this report is to bring justice to your hard work during the semester and make so that future generations of Duckietown students may take full advantage of it. Some of the sections of this report are repetitions from the preliminary and intermediate design document (PDD, IDD respectively)._

## The final result {#fleet-communication-final-result}

_Let's start from a teaser._

* Post a video of your best results (e.g., your demo video)

Add as a caption: see the [operation manual](#demo-template) to reproduce these results.

## Mission and Scope {#fleet-communication-final-scope}

_Now tell your story:_

Enable Duckiebots to communicate with each other wirelessly.

### Motivation {#fleet-communication-final-result-motivation}

_Now step back and tell us how you got to that mission._

- What are we talking about? [Brief introduction / problem in general terms]

- Why is it important? [Relevance]

- In the previous state of Duckietown, Duckiebots wer individual autonomous agents roaming around Duckietown with no way to communicate with each other. To connect all Duckiebots to build one functioning system of fleets working together to pick up and drop customers in the optimal way, the Duckiebots need to be able to communicate with each other.

- One important part of this communication setup is that it needs to be decentralized and Duckiebots can join and leave the system without putting the whole network at risk to fail. This also allows for the network to be scaled to an arbitary size.

- Due to the current state of Duckietown, the communication is needed, but not limited to, fleet planning control and multi-SLAM.


### Existing solution {#fleet-communication-final-literature}

- There was no prior work to build a communication system upon. Everything was implemented from scratch.

### Opportunity {#fleet-communication-final-opportunity}

- We implemented a fleet-communication package from scratch that builds an ad-hoc mesh network and lets other teams define their messag types and sends them over the created network.


- Make sure to reference papers you used / took inspiration from

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

## Contribution / Added functionality {#template-final-contribution}

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

_Is there something you think still needs to be done or could be improved? List it here, and be specific!_
