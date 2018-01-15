# Demo template {#demo-template status=beta}

This is the description of a communication setup between multiple Duckiebots.

<div class='requirements' markdown="1">

Requires: At least two Duckiebot in configuration DB17-w or higher.

Requires: One additional wireless adapter per Duckiebot.

Requires: A laptop.

</div>


## Duckietown setup notes {#demo-template-duckietown-setup}

For this Demo, no Duckietown is needed.


## Duckiebot setup notes {#demo-template-duckiebot-setup}

For this demo, additional wireless adapters are needed that allow mesh networking (e.g. TP-Link TL-WN822N or TL-WN821N).


## Pre-flight checklist {#demo-template-pre-flight}

The pre-flight checklist describes the steps that are sufficient to
ensure that the demo will be correct:

Check: Additional wireless adapter is plugged in.

Check: Duckiebots have sufficient battery charge.

## Demo setup {#demo-template-run}

First, you need to install the wireless adapters. If the adapters are plugged into the Duckiebots, run the following command on each Duckiebot.

    duckiebot /catkin_ws/src/30-localization-and-planning/fleet_messaging/dependencies/install_meshnet


Three more packages are needed to enable the communication beween the Duckiebots, namely Protobuf, ZeroMQ and B.A.T.M.A.N. For this run the following code.

    duckiebot /catkin_ws/src/30-localization-and-planning/fleet_messaging/dependencies/install_fleet_messaging

Now you are ready to make your Duckiebots talk to each other.


## Demo instructions {#demo-template-run}

In your duckitown repository on your duckiebot, run

    duckiebot $ source environment.sh
    
... 

## Troubleshooting {#demo-template-troubleshooting}

Add here any troubleshooting / tips and tricks required.

## Demo failure demonstration {#demo-template-failure}

Finally, put here a video of how the demo can fail, when the assumptions are not respected.
