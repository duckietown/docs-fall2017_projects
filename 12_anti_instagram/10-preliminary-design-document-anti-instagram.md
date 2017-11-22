# PDD - Anti-Instagram {#devel-anti-instagram status=beta}

## Part 1: Mission and scope

### Mission statement

Make the Duckiebot robust to illumination variation.

Line detection includes different colors of lines, we need to be able to detect these colors accurately.

### Motto

> SEMPER VIGILANS (Always vigilant)

### Project scope

#### What is in scope

* Improve software (anti instagram, line detection, ...) such that the line detection is robust to illumination variation
* Sample ground truth pictures
* Influence future changes on Duckietown (e.g. color of parking spots)

#### What is out of scope

* Geometric interpretation of the line detection (e.g. where is the middle of the road, distance to certain objects, …)
* Hardware modifications of the duckiebot
* Hardware modifications of the current Duckietown set up (colors of lanes, stop line, …)

#### Stakeholders

|  Team |  Reference Person | Interaction  |
|---|---|---|
| The heroes  | Sonja Brits  |  She helps us to interact with other groups. We talk with her if we change our project. |
|  The controllers | Andreas Aumiller  | He is the interaction person to confirm that we fulfil their requests regarding: frequency, latency, accuracy (resolution), maximum false positives, maximum misclassification error (confusion matrix). |
| The Parking  | Samuel Nyffenegger  |  We will determine together the best color for the parking lot lines if needed. |
| The Navigators  | Theodore Koutros  |  At the moment the duckiebot is controlled open loop at intersections. This should be improved. Probably they need line detection in a certain way. We can help and figure together out what procedure would be the best. |

## Part 2: Definition of the problem

### Problem statement

One of the central problems in autonomous mode of the duckiebot is the line detection. Line detection though is very sensitive to illumination variation. This problem has been solved by a color transformation called “Anti-Instagram”. The current illumination correction algorithm, however, is not working well enough. This affects the extraction of the line segments since the extract-line-segments algorithm is very sensitive to illumination changes (e.g. shadow, bright light, specular reflections).
There are several reasons why the current implementation fails:

1. Illumination correction is done only once by user input. So we don't do any online/automatic correction. This will obviously fail in an environment where illumination is changing frequently.
2. The algorithm works by detecting different clusters in RGB space for the colors of lines, thus it fails when it's not able to differentiate adequately. (e.g. in certain lighting conditions yellow and white look quite similar, in addition specular reflections distort all of the colors)
3. The color space is fixed to RGB, but it's unclear that this is best.
4. No geometric information is considered in differentiating colors of lines. The color information is completely decoupled from the place it's actually coming from, thus a red Duckiebot may be detected like a stop line, even though their shapes are quite different. (We also don't know whether a pixel is coming from the “street level” or the “sky level”)
5. It is a linear transformation (shift, scale) instead of a possible non-linear transformation, but there is nothing indicating this should be true.
6. Any previous anti-instagram transformation parameters are not taken into account when a new transformation is performed, thus no prior knowledge is leveraged.

### Assumptions

1. Lighting
    1. We assume normal office lighting, including any shadows that may occur because of occlusions, or spatial variance.
    2. We don't consider outdoor illumination (e.g. sunlight)
    3. We assume having illumination (no pitch black scenario)
2. Duckietown Condition
    1. We assume the normal colors of lane lines, plus one or two more (for the parking lot).
    2. We assume no variation in the shapes of the lines besides what is already constructed.
3. Training data
    1. We expect to have some pictures in different scenarios with correctly labeled segmentation (lines, street, …), where a polygon is drawn around each region.
    2. The pictures will be captured from a Duckiebot camera.
4. We also assume the current method of extracting lane pose from segments is accurate.


### Approach

1. Understand current system
    - Determine false positives, false negatives, true positives, true negatives of current line detection
    - Determine latency
    - Compare other color spaces than RGB to see how it affects performance (e.g. HSV or LAB).
2. Use geometric information to better determine the actual existing colors.
    - The optimal case would be that the system already knows beforehand which areas it should take into account. The relevant color areas are only the dashed lines area, the continuous line area, the stop line area, the parking line area and the street area. Everything else should not taken into account since we have no reference color for the other areas.
    - It should be possible to define a region in the picture where we can find these specific areas. For example the dashed line starts lower left and goes to direction top middle but it should stop at the “line of horizon” (= middle of the vertical length)
    - Distinguish between dashed and non-dashed lane lines to simplify identification of colors.
3. Use time information/parameters from earlier illumination corrections to improve robustness (Online learning)
    - In contrast to starting the color analysis from scratch every time, we could consider using the latest transformation parameters as an initial guess.
We could consider to update the color analysis every x-th frame during a session, to be more robust to changing light conditions/shadows.
4. Further improvements
    - We are using the color transformation to better estimate the lines. So we know after processing (color transformation, edge detection, …) where we can find the lines. With that information we could update the color transformation. The color transformation now should take into account only the “important” areas. As a result we should have a more accurate color transformation. We repeat until we converge to a minimum. 

### Functionality-resources trade-offs

### Functionality provided

We are assuming to have ground truth pictures. Then it is possible by processing the same picture with our algorithm and compare it to the ground truth to calculate an error. 

We are going to consider true positives, true negatives, false positives and false negatives. This can be done either for only one color/one feature (dashed lines, continuous lines…) or for the whole process at once which means everything should be classified properly. In addition, we would like to measure the accuracy of the lane pose estimation for our algorithm vs. a ground truth, this can be a Euclidean distance.

### Resources required / dependencies / costs

Costs:

1. Computational cost
    - If the processing is done online we have to take care that it doesn't take too long.
2. Cost of producing ground truth pictures
    - Will be determined when we have some examples done by hand. (Week of 20th of November)
Resources:

1. Functional Duckiebot
    - Getting sample data for ground truth pictures
    - Try out the algorithm in real conditions

Dependencies:

1. Ground truth images
2. Lane pose estimator

### Performance measurement

1. Identify current run times of algorithms implemented at the moment and compare the algorithms intended to implement with the current ones. If the new ones are way more costly than the current ones and the current ones already use the Raspberry Pi to capacity it is probably not a good idea to implement ours. To sum up: Estimate run time of new implementation and compare to old. See whether implementation is feasible.
2. We can compute percentage of success of identification of a line segment, as well as correct color classification.
3. Measure euclidean distance of lane pose estimation using our algorithm and lane pose estimation without to the ground truth.

The performance measurement procedure for the algorithm is described in the section *Functionality provided*.

## Part 3: Preliminary design

### Modules

1. Anti-Instagram module: Takes raw picture from camera and estimates a color transformation. The transformation details are returned.
2. Module to classify geometries (e.g., distinguish between dashed lines, continuous lines and stop lines): This could be a standalone routine, which gets called by the Anti-Instagram, we could also combine this with the anti-instagram as described in Approach point 4.
3. Online learning: Takes picture from camera, does Anti-Instagram procedure and transformation. The error (e.g. cluster error of k-means) is estimated and the procedure is repeated until the optimal transformation parameters are found (transformation with the lowest error). The procedure returns the optimal parameters. They are saved and used for the future image processing.

### Interfaces

1. Anti-Instragram: input: Raw camera image. Output: Transformation parameters
2. Geometry Classifier: Input: Raw camera image. Output: List of the different line segments.
3. Online learning module: input: Multiple camera images/continuous stream. Output: Optimal transformation parameters

### Preliminary plan of deliverables

1. Take the current algorithm and find best color space for it, estimate the errors and accuracies discussed previously. 
2. Search for other clustering method and optimize current version. (Without considering geometry)
3. Consider geometry (as a first step indicate considerable areas by hand) and see what difference it makes compared to the current optimal implementation (Maybe after 1.) and 2.) are done).
4. Distinguish relevant and non-relevant areas (street surface vs. rest of world).
5. Distinguish dashed and continuous lines.
6. Implement and test a online learning system.

### Specifications

As stated above we are only involved in determining the best fitting color of the (not yet installed) parking lot lines. All the other colors and the environment are assumed to be given.

### Software modules

1. Anti-Instagram Node: Already exists, will most likely be updated or even changed completely according to our approach. The online-learning (if accomplished) will be folded into this node.
2. Geometry Classifier Node: Needs to be written according to the approach.

### Infrastructure modules

None.

## Part 4: Project planning

|  Date [MM/DD/YYYY] |  Task | Who | Target Deliverables |
|---|:--|---|:--|
|11/20/2017|Finish the Preliminary Document, Peer Reading of other team members |Milan, Christoph|Preliminary Document|
|11/27/2017|Investigate why current algorithm fails.|Milan, Christoph|Create detailed description when the algorithm fails and when it works. Make it understandable why for everyone|
|11/27/2017|Create data annotation, check how website works|David, Shengjie|Get 1000 annotated pictures or have a specific date when these images are delivered.|
|12/1/2017|Find out what colorspace is the best for the current algorithm||best color space, performance analysis|
|12/4/2017|Find out what's the best clustering method based on best color space, is the best color space still the best?||Best clustering method, best color space, performance analysis|
|12/4/2017|Include geometry in the current color transformation algorithm||Performance analysis|
|1/8/2018|Implement an online system||Performance analysis of supervised system|

### Data collection

Around 1000 pictures with the duckiebot camera from a duckiebot perspective in Duckietown. The pictures have to be from different environment conditions (illumination, specular light)

### Data annotation

The data collected above has to be annotated. The annotations should state what type and what color it is.

1. Dashed lines: Yellow
2. Continuous lines: White
3. Stopping lines: Red
4. Street: Black
5. (Parking Lot: TBD)

#### Relevant Duckietown resources to investigate

The whole sum of nodes within the ‘10-lane-control' folder will be within the scope of this project. They are:

- Anti_instagram
- Ground_projection
- Lane_control
- Lane_filter
- Line_detector
- complete_image_pipeline

#### Other relevant resources to investigate

1. Color differentiation, like this https://arxiv.org/pdf/1506.01472.pdf
2. Properties of different color spaces
3. OpenCV

### Risk analysis

1. Computationally too expensive algorithms
    - We have to estimate our algorithms carefully and compare them to the existing solutions.
2. No annotated data delivered
    - Build annotated data by ourselves/by hand.
3. Not enough time
    - Create good tasks list to be done. Try to specify time exact for every task.
