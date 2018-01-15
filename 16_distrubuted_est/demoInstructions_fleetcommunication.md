# Demo template {#demo-template status=beta}

This is the description of a communication setup between multiple Duckiebots.

<div class='requirements' markdown="1">

Requires: At least two Duckiebot in configuration DB17-w or higher.

Requires: One additional wireless adapter per Duckiebot and laptop. (e.g. TP-Link TL-WN822N or TL-WN821N).

Requires: A laptop.

</div>


## Duckietown setup notes {#demo-template-duckietown-setup}

For this Demo, no Duckietown is needed.


## Duckiebot setup notes {#demo-template-duckiebot-setup}

For this demo, additional wireless adapters are needed that allow mesh networking (e.g. TP-Link TL-WN822N or TL-WN821N).

## Pre-flight checklist {#demo-template-pre-flight}

This pre-flight checklist describes the steps that are sufficient to
ensure that the demo will be correct:

Check: The additional Wifi adapter is installed and works.

    $ sudo apt-get install build-essential linux-headers-generic dkms git
    $ git clone https://github.com/Mange/rtl8192eu-linux-driver.git
    $ sudo dkms add ./rtl8192eu-linux-driver
    $ sudo dkms install rtl8192eu/1.0

Check: Duckiebots have sufficient battery charge.

## Demo setup {#demo-template-run}
Some packages are needed to enable the communication beween the Duckiebots, namely Protobuf, ZeroMQ and B.A.T.M.A.N.

ssh into the Duckiebots and source the environment

    $ cd duckietown
    $ source environment.sh

and find the name of the wifi interface you can use iwconfig. (eg. wlx7c8bca1120e0)

    $ iwconfig

Next specify a static IP adress and write it on a piece of paper, be carefull to not use the same IP on two bots. (eg. 192.168.15.38/24)

    
Then change to dependecie directory

    $ cd ~/duckietown/catkin_ws/src/30-localization-and-planning/fleet_messaging/dependencies
    
install!
  
    $ ./install_fleet_messaging <wifi-iface> <ipaddr>

After a reboot you are ready to make your Duckiebots talk to each other.


## Demo instructions {#demo-template-run}

ssh into the bots, then in your duckietown repository run:

    $ source environment.sh
    
    $ roslaunch fleet_messaging tester.launch
    
enjoy the show!

## Troubleshooting {#demo-template-troubleshooting}

Add here any troubleshooting / tips and tricks required.

## Demo failure demonstration {#demo-template-failure}

Finally, put here a video of how the demo can fail, when the assumptions are not respected.
