---
layout: post
title: Perspective Correction
header-img: img/projects/room.jpg
subtitle: Using OpenCV for correcting perspective warping.
tags:
    - opencv
    - homography
---

In the following image, we have an object which is slightly warped due to perspective.
What if we could come up with an algorithm to correct this perspective distortion and 
output a corrected image?

To achieve this, we will need some data.

- a **homography** matrix
- we need to check which object we are interested in (possibly a process to identify and
correctly apply a correction)

Opencv provides the functionality to compute a homography matrix given a set of 4 coressponding
features in 2 different images.

However, we will first try to implement this functionality on our own to improve our
understanding.

This leads us to the question: **What exactly is Homography matrix?**
// from wiki


In simpler terms, a Homography matrix is a transformation matrix which defines transformation of
a set of points in image A to same set of points in image B. These points are also called coressponding
features/points (which is analogous to a 3D transformation matrix which defines tranformation of a set
of points from space A to space B).

Opencv native functionality
----

- cv::findHomography:

        Finds Homography matrix given a set of source, destination points and images.


- cv::warpPerspective:

        Warps the specified image using a given matrix.


The generic procedure would be to detect lines which should be straight in the input image.
Then finding a homography matrix describing transformation of coressponding features in the two images.

**NOTE**

This technique requires us to have 2 images which have the same features from slightly different perspectives.
However, I am interested in fixing perspective distortion on a specified image without having another set of image.
Need to investigate this further.

Consider the following image, the green lines are what I know are parallell in the scene but appear
to be non parallell in the image. This is due to perspective distortion.
