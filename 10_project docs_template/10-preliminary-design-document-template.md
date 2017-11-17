#  Group name: preliminary design document {#project-name-preliminary-design-doc status=draft}

<div class='requirements' markdown="1">

Requires: Group Name

Result: Group-Name-preliminary-design-document

</div>

## Part 1: Mission and scope

### Mission statement

What is the overarching mission of this team? You should write in one sentence.

What is the need that is being addressed? Do not focus on technical specifics yet.

### Motto

Your rallying cry into battle. Traditionally, Duckietown uses Latin mottos.

<div class='check' markdown="1">

Per aspera sic itur ad astra

</div>

### Project scope

Are you going to rewrite Duckietown from scratch? Probably not. You need to decide what are the boundaries in which you want to move.

#### What is in scope

What do you consider in scope? (e.g. having a different calibration pattern)

#### What is out of scope

What do you consider out of scope? (e.g. hardware modifications)

#### Stakeholders

What other pieces of Duckietown interact with your piece?

List here the teams, and a possible contact person for each team.


## Part 2: Definition of the problem

### Problem statement

Time to define the particular problem that you choose to solve.

Suppose that we need to free our prince/princess from a dragon. So the mission is clear:
Mission = we must recover the prince/princess.

Now, are we going to battle the dragon, or use diplomacy?

If the first, then the problem statement becomes:
Problem statement = We need to slain a dragon.

Otherwise:
    Problem statement = We need to convince the dragon to give us the prince/princess.

Suppose we choose to slain the dragon.

### Assumptions

At this point, you might need to make some assumptions before proceeding.

Does the dragon breath fire?
What color is the dragon? Does the color matter?
How big is this dragon, exactly?

### Approach

All right. We are going to kill the dragon. How? Are we going to battle the dragon? Are we trying to poison him? Are we going to hire an army of mercenaries to kill the dragon for us?

### Functionality-resources trade-offs

The space of possible implementations / battle plans is infinite.
We need to understand what will be the trade-offs.

### Functionality provided

How do you measure the functionality (what this module provides)?
What are the "metrics"?

Example: numbers of dragons killed per hour

Note that this is already tricky. In fact, the above is not a good metric. Maybe we kill the dragon with an explosion, and also the prince/princess is killed. A better one might be:

Example: numbers of royals freed per hour
Example: probability of freeing a royal per attempt

It works better if you can choose the quantities so that functionality is something that you maximize to maximize. (so that you can "maximize performance", and "minimize resources").

### Resources required / dependencies / costs

How do you measure the resources (what this module requires)?

Example: numbers of knights to hire
Example: total salary for the mercenaries.
Example: liters of poison to buy.
Example: average duration of the battle.

It works better if you think of these resources as something to minimize.

### Performance measurement

How would you measure the performance/resources above? If you don't know how to measure it, it is not a good quantity to choose.

Example: we dress up Brian as a Dragon and see how long it takes to kill him.
