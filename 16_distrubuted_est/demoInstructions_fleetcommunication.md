# Demo template {#demo-template status=beta}

This is the description of a communication setup between multiple Duckiebots.

<div class='requirements' markdown="1">

Requires: At least two Duckiebot in configuration DB17-w or higher.

Requires: A laptop.

</div>


## Duckietown setup notes {#demo-template-duckietown-setup}

For this Demo, no Duckietown is needed.


## Duckiebot setup notes {#demo-template-duckiebot-setup}

No modification on the Duckiebots are needed.

## Pre-flight checklist {#demo-template-pre-flight}

This pre-flight checklist describes the steps that are sufficient to
ensure that the demo will be correct:

Check: The Edimax adapter is installed and workes.

Check: Duckiebots have sufficient battery charge.

Check: Laptop is able to connect to Duckiebot networks.

## Demo setup {#demo-template-run}
Some packages are needed to enable the communication beween the Duckiebots, namely Protobuf, ZeroMQ and B.A.T.M.A.N.

ssh into the Duckiebots and source the environment

    $ cd duckietown
    $ source environment.sh

and find the name of the wifi interface you can use iwconfig. (eg. wlx7c8bca1120e0 or wlan0)

    $ iwconfig

Next specify a static IP adress and write it on a piece of paper, be carefull to not use the same IP on two bots. (eg. 192.168.15.38/24)

    
Then change to dependecie directory

    $ cd ~/duckietown/catkin_ws/src/30-localization-and-planning/fleet_messaging/dependencies
    
install!
  
    $ ./install_fleet_messaging <wifi-iface> <ipaddr>

Now you are ready to make your Duckiebots talk to each other.


## Demo instructions {#demo-template-run}

ssh into the bots, then in your duckietown repository run:

    $ source environment.sh
    
    $ roslaunch fleet_messaging tester.launch
    
enjoy the show!

## Troubleshooting {#demo-template-troubleshooting}

Add here any troubleshooting / tips and tricks required.

## Demo failure demonstration {#demo-template-failure}

Finally, put here a video of how the demo can fail, when the assumptions are not respected.
