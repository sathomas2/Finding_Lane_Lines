# Finding Lane Lines
#### Project 1 of Udacity's Self-Driving Car Nanodegree


## Initial Pipeline
1) Convert RGB image to grayscale.

2) Apply Gaussian Blur.

3) Use Canny Edge Detection to detect edges.

4) Apply mask to detect edges only within the region where relevant lane lines could appear.

5) Apply Probabilistic Hough Transform to detect line segments.

6) Distinguish between left and right lane lines by calculating the sign of each line segment's slope.

7) Compute the average negative slope and its average bias and the positive slope and its bias of the lines.

8) Set y1 at the bottom of the image and y2 at a reasonable level. y1 and y2 are the same for both lines. Using the mean slopes and biases, derive x1 and x2 for both left and right lane lines. Draw both line segments on a blank image.

9) Blend the original image with the red-line image.


## Good News
For the easier videos, solidWhiteRight.mp4 and solidYellowLeft.mp4, it doesn't do a bad job.
<figure>
 <img src="examples/yellow_eg_good" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;">Smooth sailing...</p> 
 </figcaption>
</figure>
 <p></p> 
 
 <figure>
 <img src="examples/yellow_eg_bad" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;">... for the most part</p> 
 </figcaption>
</figure>
 <p></p> 
 
There are one or two single frame mess-ups, as above, that don't veer off the road or into the wrong lane but could slow the car's progress, but they instantly fix themselves. Overall, it works pretty well, so I tested it on the challenge video. However, the current pipeline has several issues:

<figure>
 <img src="examples/lines_frame_98" width="380" alt="Combined Image" />
  <img src="examples/lines_frame_139" width="380" alt="Combined Image" />
 <img src="examples/lines_frame_159" width="380" alt="Combined Image" />
 <figcaption>
 <p></p> 
 <p style="text-align: center;">In general, long spaces between dotted lines, different road colors, and shadows can be problematic for edge and line detection.</p> 
 </figcaption>
</figure>
 <p></p> 


## Issues and How to Solve Them
1) Challenge video has a different image shape, so needed a new image mask and new y-values for the endpoints in the draw_lines function.

2) The white dotted lines on the right often have large spaces, making it difficult for the hough_lines function to detect the lines.

3) The white dotted lines sometimes blend into the road, also making it hard for edge detection.

4) The solid yellow line on the left also sometimes blends in due to brightness and changing road color.

5) The solid yellow line is always long, but if the min_line_len is too short, Hough lines function will detect irrelevant lines, which can be caused by factors like tree shadows.

To fix everything I created two image masks, one to highlight the left lane and the other the right. I did this to control the threshold, min_line_len, and max_gap_len in the hough_lines function, allowing me to fine-tune the hyperparameters separately.

This helped, but there were still problems with poor contrast due to a change in road color or long spaces between the white dotted lines. For the most part, neither issue lasted for many frames, so I calculated the endpoints of the line segments with a moving average of the average slopes and biases over the previous five frames. This also helped to avoid the occasional frame when light conditions made useful Hough line detection with our current set-up impossible. Without something like a moving average, if no right line is detected, then there is no way to caculate x and y values for the right lane's endpoints, causing the pipeline to break.

Even the moving average can't help when light road color causes poor contrast for several frames, yet the hough_lines function detects at least one line per side. I did a test run with the current pipeline. Afterwards, uisng np.percentile, I calculated the 15th and 85th percentile of average slopes for the right and left lines. I changed the get_slopes_n_biases function so only those lines detected by the hough_lines function that had slopes within around the 15th and 85th percentile range were used. 
 
Adding these extra steps got the job done, at least with respect to the challenge video. In other videos, difficulties may arise in certain situations, like during sharp turns, on older roads with fading lane lines, or at intersections.


## Final Pipeline
1) Convert RGB image to grayscale.

2) Apply Gaussian Blur.

3) Use Canny Edge Detection to detect edges.

4) Apply masks to detect edges only within the region where relevant lane lines could appear, one mask for each lane.

5) Apply Probabilistic Hough Transform to detect line segments, once for both masked images.

6) Compute the average slope and bias of the lines that travel through each segment for both masks.

7) Set y1 at the bottom of the image and y2 at a reasonable level for both masked images. Using the mean of the average slopes and biases of the previous five frames, derive x1 and x2 for both images. Draw both line segments on a blank image.

8) Blend the original image with the red-line image.


**Run the notebook or watch test_videos_output/challenge.mp4 to see the results!**
