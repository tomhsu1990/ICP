# Iterative Closest Point (ICP)

The main concept of 2D ICP is to find the closest point for each source point. In addition, the key is that after determining the closest points' coorespondences, how do we calculate the rotation. Below is the big picture of computing the rotation part.

We have two points set: closest points set and source points set.<br>
After centering at each mean, every point lives in its own coordinate.<br>
Suppose the rotation we want to compute is r.<br>
Let r = r_closest - r_src.<br>
r_closest is the shifted closest points' coordinate relative to the global coordinate.<br>
r_src is the shifted source points' coordinate relative to the global coordinate.<br>
The formula below is an approximation to r.<br>
r = atan2(y'/x') - atan2(y/x), (x, y) is the source coordinate and (x', y') is the closest points' coordinate.<br>
app(r) = (atan2(y'/x') - atan2(y/x))/(1+yy'/xx')<br>
app(r) is proportional to r and up to scale by 0.5 to 1.0.<br>

Another big problem is the local minima. The way I deal with local minima is by generating multiple rotation hypothesis at local minima and doing ICP again to find the minimum error.

Here is my demonstration video: [2D Iterative Closest Point](https://youtu.be/m64E47uvPYc).

## Algorithm of the ICP Module

1. Get the closest points' coorespondences via [Approximate Nearest Neighbors (ANN)](https://www.cs.umd.edu/~mount/ANN/).

2. Compute delta translation and rotation.

3. Calculate the error function based on the transformed points set and their cooresponding closest points set.

4. Suppose we reach a local minima.

5. Generate rotation hypothesis and repeat steps 1 to 3.

## Discussion

To show the peroformace and accuracy, I randomly generat a pair of translation and rotation to move current points set to a target position.

In the video, red points set is the source points set and green points set is the target points set.

During the process, magenta points set is the local minima.

Yellow points set is the rotation hypothesis. (This part can be parallelized.)

Blue points set is our find result.

Meanwhile, in the terminal, you can see the tranlation and rotation result when reaching local minima.

At the end, you can see the final translation and rotation and the original randomly generated translation and rotation.

## How to run (MacOS)

* Please make sure you have installed OpenCV and ANN library.

mkdir build

cd build

cmake ..

make

1) ./ICP
(The default img is ../img/curve.png.)

2) ./ICP ../img/corner.png
(Change img.)

3) ./ICP ../img/corner.png 1000
(Uniformly add 1000 noise points to the target points set.)

4) ./ICP ../img/corner.png 10 2000
(Uniformly add 10 noise points and remove 2000 points from the target points set.)

## Question

1. How do we extend 2D ICP into 3D ICP? Is there a way to compute 3D rotation? RANSAC+SVD?

2. How do we speed up the performance by doing down-sampling?
    We can try uniform sampling first to realize down-sampling. If we try to keep the shape information when doing down-sampling, we need to have more samples agound the shape features such as corners.

3. Is there another way to solve local minima problem?

4. Is is reasonable to set the error difference as 1e-6?

5. Could we use fewer iterations as a termination condition?
    When the error difference is small, we can deduct the termintation iteration. In other words, an adaptive iteration bound can be used to improve this issue.

## Reference

[ICP（Iterative Closest Point）演算法實作 (C 語言)](http://ivory-cavern.blogspot.com/2009/11/icp-iterative-closest-point-c.html)
