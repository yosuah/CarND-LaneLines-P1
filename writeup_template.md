# **Finding Lane Lines on the Road** 

---

**Finding Lane Lines on the Road (using hand-engineered CV methods)**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

[//]: # (Image References)

[image1]: ./steps_plot.png "Major steps of the pipeline"

---

### Reflection

### 1. Pipeline description

I created a pipeline that consists of 3 main steps: creating a masking filter, selecting the relevant
edges that most likely make up the lane separators (segmentation), 
and calculating a single combined line for the two closest separators. 
In this implementation I only focused on the two separators that make up the ego lane.

The pipeline runs per-frame, there is no temporal tracking or any information carry-over between
frames (I took this as a challenge to create reasonably stable output on the video without
using tracking). This pipeline provides acceptable results on the sample videos, but
it is hand-tuned to these videos and would certainly fail in more complex scenarios. It already
fails on some parts of the challenge video.

The following steps are executed for each frame:
1. Masking
    1. Convert the image to HSV color space
    2. Select yellow-ish pixels from hue (H).<br>
       This is done to keep yellow separators,
       but it also keeps all other yellow objects, and some green vegetation.
    3. Select bright areas from saturation (S) and value (V), 
       regardless of their hue.<br>
       This selects white separators, plus other bright features
       of the image. White cars are the most problematic of these.
    4. Combine (OR) the two masks together.
    5. Select a manually defined region of interest (ROI), which is roughly a triangle
       from the bottom of the image to the center. <br>
       Cut the previous mask with this (AND them together).
       This is the most important filtering step right now, as it removes most of the 
       artifacts from other operations. However this is also the reason why most of the lanes
       end prematurely.
2. Segmentation of lane separators
    1. Convert to grayscale
    2. Blur.<br>
       A Bilateral filter is used in the final submission, as it blurs while preserving
       more of the edges. On the other hand it is slower than a regular Gaussian blur.
    3. Canny edge detection
    4. Hough transform
3. Calculation of a single line for the two closest separators
    1. Calculate line parameters for all line segments.<br>
       Much like in a regular Hough transform,
       I first calculate the parameters `(m, b)` for all lines. This is necessary beacuse OpenCV
       does not return the line parameters when using the `HoughP` function, which was used
       in the previous step.
       Additionally a third parameter is added, describing whether the line is in the left or right half of the image.
    2. Cluster line segments using DBSCAN.<br>
       Cluster line segments to group the ones that make up the whole lane separator while
       ignoring all other artifacts. 
       The left/right parameter was added in the previous step because in
       some cases DBSCAN added erroneous short line sections to otherwise correct groups,
       which were usually on the other side of the image (hence their m and b were very similar).
       This third parameter helped the algorithm to consider these as outliers.
    3. Calculate combined line parameters for all clusters.<br>
       Parameters `(m, b)` are calculated for each cluster of line segments. 
    4. Filter clusters based on their inclination.<br>
       Remove those groups where `-0.3 < m < 0.3`, essentially which are too close to horizontal.
    5. Select the closest ones for each side from the remaining.<br>
       From the remaining line groups, select one with positive and one with negative inclination
       that is the closest to the center of the image. These are considered to be the most likely
       lane separators.
    6. Draw.<br>
       The lines are drawn from the bottom of the image up to the last actual point that was
       found in the edge detection. This is done to avoid drawing too long lanes,
       as the two separators could be crossing each other.

The following image shows the major steps of the pipeline (but not all steps are visualized):
![alt text][image1]

I used Jupyter interact to create an environment where the effect of each parameter can be 
explored fairly easily. This is still not very efficient, as only a limited number of images
can be tested this way, so checking the final videos still proved to be the most
efficient method of evaluation.

For the third step (Calculation of a single line for the two closest separators),
I could have simply taken the filtered image and applied Hough transform on that, but I wanted
more control and wanted to explore the options for grouping together line segments.
DBSCAN might be a more complicated algorithm, but it worked well in this case.

### 2. Identify potential shortcomings with your current pipeline

There are several major shortcomings to this implementation.

1. Sensitive to white cars and other artifacts<br>
   One of the most important issue is that it is sensitive to artifacts, especially
   white cars and other objects close to the road with a strong gradient.
2. Aggressive and fixed ROI.<br>
   The previous issue is fixed by using setting the upper boundary of the ROI a bit lower
   than what would be ideal. This fixes the issue so the resulting videos look good, but of
   course this is more like cheating - it would break down if the road has strong inclination,
   if the pitch of the vehicle changes significantly (e.g. on a bump), when the vehicle gets
   closer than usual to the leading vehicle, etc.
3. Most of the parameters are tuned to accept lanes on straight roads or roads with
   large curvature radius.<br> This would break down when trying to take an exit or going into a
   road with smaller curvature radius.
4. Temporally unstable, no tracking.<br>
   All detections are performed per-frame, so the results can be totally different on successive
   frames.
5. Parameters hand-selected for these videos, no validation on larger set


### 3. Suggest possible improvements to your pipeline

1. Use the `rho`, `theta` parametrization for lines everywhere instead of `m`, `b`.<br>
   I should have used it in the first place.. Now the code is littered with edge cases
   handling m == 0.0
2. Improve criteria when selecting the final line segment group.<br>
   The criteria currently used when selecting be best ones from the possible line groups
   is very simplistic. It could be easily extended with the total length covered by the
   line segments, whic would favor longer line segments over shorter artifactual groups
   that could happen to be closer to the center of the image.
3. Approximate vanishing point, either from current or the previous frame.<br>
   The top of the ROI is an important parameter currently which is hard-coded. Ideally
   it should be the top point where a lane separator can still be seen, which is basically
   the vanishing point of the lanes. This could be estimated separately on each frame,
   using the already found 'bottom part' of the lanes, or it could be carried over from the
   previous frame. Doing the former would require a bigger change, as it would basically
   turn the algorithm into a two pass pipeline.
4. Use HSV for edge detection, then combine it with the grayscale results, instead of masking.
   Converting to HSV worked well, but using it only for masking has limited value. Maybe
   we could try to detect edges on the binary image that is currently used as the HSV mask,
   and combine this with the edges calculated from the grayscale image as a kind of voting.
5. Add temporal tracking.<br>
   Obviously some form of tracking across successive frames would be essential for any real
   application.
6. Get a labelled data set and automatically tune the parameters on that.
   These parameters were hand-tuned, so they are guaranteed to be non-optimal even on this 
   small test set. Ideally I should pick a data set which contains annotations for the lanes,
   then automatically optimize the parameters based on that. This would requre selecting
   an appropriate 'goodness function' to be used by the optimizer.
7. Performance optimization.<br>
   The current implementation does not run real-time (in my laptop), which would be a basic
   criteria for a real world system. There are numerous optimizations that could be made, from
   small changes like reducing the number of temporary images, to large ones like optimizing 
   for GPUs, etc.
   
Thanks for checking this out :)
