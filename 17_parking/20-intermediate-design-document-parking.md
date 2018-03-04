#  Parking: intermediate report {#parking-int-report status=beta}

## Part 1: System interfaces

### Logical architecture

Description of the functionality. What happens when we click start?

- As soon as you arrive to the parking lot and see the corresponding entrance april tag, the Duckiebot switches from normal driving mode to parking mode. Parking mode is only allowed in the parking lot. If the bot exits the parking lot, it sees another april tag and it switches from parking mode back to normal driving mode (starting at a four way intersection).

- Inside the parking lot the robot estimates his pose (x, y and theta) using a bunch of april tags. There is one (maybe also two) april tag per parking space and some additional (entrance, exit, etc.) To do so, a new state estimation algorithm has to be implemented using the library 'AprilTags C++'. It estimates the relative position of the robot with respect to the april tag. The location of the april tag is encoded in the QR code. As soon as you see one (better two) tags, the pose can be calculated. We assume that we always see at least one tag.

- Given a prior information about the parking lot (where are the parking spaces, where can the robot drive etc) and real time vision information the robot chooses a parking space. At first we assume that the parking lot is empty or that other Duckiebots are static (do not move) and this is encoded in the parking map (places where the robot is not allowed to drive).

- We use RRT* to generate a path given the pose of the robot, the pose of the assigned parking space and the parking map. To do so we use the 'open motion planing library (OMPL)'.

- We control the robot to the optimal path using a sufficient controller using visual feedback.

- For driving to the exit, we generate a path and control our robot to this path which includes driving backwards to leave the parking space and turn to get to the exit in a forward motion.


Target values:

- accuracy: the error is a combination of localization accuracy and the offset due to the maximum allowable controller error. To park two Duckiebots next to each other within the space boundaries, the path planning accuracy has to be less (or equal) than 5 cm (which is the distance from the robot edge to the parking lane)

- the point of the robot which is the furthest away from the parking mid line should be less than half of the parking space width while the heading of the robot must be less than a constant (20 degrees) relative to the parking space boundary lines.

Assumptions about other modules:
- we assume that the robot finds itself at the entrance of the parking lot whenever it wants to get a parking space

- once in the parking lot: parking is decoupled from everything else


### Software architecture

```rosnode list```:

- someone
    - publishes: driving\_mode

- /vehicle/parking\_perception\_localization
    - subscribes: driving\_mode, camera\_image,
    - publishes: parking\_mode, space\_status, pose\_duckiebot, ,

- /vehicle/parking\_path\_planning
    - subscribes: parking\_mode, pose\_duckiebot,  space\_status
    - publishes: reference\_for\_control, (path)

- /vehicle/parking\_control
   - we copy this node from 'the controllers'
   - subscribes: reference\_for\_control
   - publishes: motor\_voltage

- /vehicle/parking\_LED
   - subscribes: parking\_mode, space\_status
   - publishes: -

```rostopic list```:
- /vehicle/driving\_mode
    - values = {driving, parking}
    - frequency: ~ 1 Hz

- /vehicle/parking\_mode
    - values = {parking, staying, leaving, observing}
    - frequency: ~ 1 Hz

- /vehicle/space\_status
    - 1xN array, N = number of parking space
    - values = {taken, free, not\_detectable, my\_parking\_space}
    - frequency: ~ 1 Hz

- /vehicle/pose\_duckiebot
    - x,y,theta
    - frequency: inherit from camera\_image (~30 Hz)

- /vehicle/path
    - x,y,theta array
    - frequency: very low - only updated once (if my\_parking\_space = {1:3}) or twice (for my\_parking\_space = {4:6}, first path is to go to the middle and observe which parking spaces are free, second path is to go go the associated parking space)
    - computation time ~ 10 s

- /vehicle/reference\_for\_control
    - d (orthogonal distance to path), c (curvature), phi (differential heading path and Duckiebot)
    - frequency: first step (path generation) uses a lot of time ~ 10 s, afterwards fast (~ 30 Hz) 

- /vehicle/motor\_voltage
    - two values for the two motors
    - frequency: fast (~ 30 Hz)


Introduced latency to other modules:

- we need some additional lines for the april tag detection in order to switch the driving_mode to parking (this needs a new publisher in this node) --> latency should be negligible

- otherwise we do not introduce delay to other modules since parking is decoupled

## Part 2: Demo and evaluation plan

### Demo plan

The parking feature including the design specification is new and will be implemented from scratch. Therefore, the desired functionality cannot be compared again a past version.
The demo will be split in multiple parts.

In a first demo just a single Duckiebot will enter the parking lot and take the first parking space, knowing beforehand that this space will be free. The Duckiebot is supposed to park within the marked parking space and be able to leave the parking space after a given terminal command.

In a second demo, the Duckiebot will park in space 5 or 6. These are the spaces which are not visible from the beginning. Its need a two stage path panning. 1) Driving from the entrance to the middle of the parking lot and observe if parking spaces 4 to 6 are free. 2) Driving from the middle of the parking lot to space 5 or 6. The demo is completed when the duckie successfully drives to the exit after a given terminal command.

In a later step multiple Duckiebots (one by one) will enter the parking lot. The Duckiebot will first search for an available parking space (out of max. six), undertake a parking maneuver like in the first demo. The Duckiebot will signal the other Duckiebots that the parking space is taken by outputting a sequence with the LEDs at the back.  After a given time each robot (one by one) will leave the parking lot again.

Needed hardware components:

 - parking lot setup including April tags
 - lanes marking the parking spaces
 - Duckiebots (obstacles at parking spaces)

### Plan for formal performance evaluation

The performance evaluation will be experimental. We will repeat the demo multiple times and measure the probability of a successful parking maneuver.


## Part 3: Data collection, annotation, and analysis

### Collection

We will need april tag data to understand at what distances and angles we can localize the Duckiebot from.

We will collect data at a distance up to 1.2m at increments of 10cm.
We will collect data at an angle up to 40 degrees at increments of 10 degrees.
We will do an "angle sweep" at each distance interval

Logs are taken manually
