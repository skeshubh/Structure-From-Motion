# Structure-From-Motion
A simple implementation of classical Structure From Motion  
This project was done in collaboration with [@Dushyant Patil]() and under the guidance of [Prof. Nitin J. Sanket](https://nitinjsanket.github.io/index.html)

## What is SfM?
Structure from Motion refers to reconstruction of a 3D scene and simultaneously obtain the camera poses of a monocular camera w.r.t. the given scene.

## Why SfM?
SfM makes it possible to simultaneously determine the camera settings and the 3D structure of a scene by combining 2D images taken from various 
perspectives. SfM approaches may create a dense point cloud from a series of partially overlapping photos even using a cheap digital camera.   
It is use extensively in geosciences to provide hyperscale landform models n remote or rugged environments where terrestrial laser scanning is 
limited by equipment portability and airborne laser scanning is limited by terrain roughness causing loss of data and image foreshortening.  
SfM is used in order to properly estimate situations as well as planning and maintenance efforts and costs, control and restoration of Cultural heritage sites.

## What we did?
#### Dataset
We were given 5 distortion corrected images of Unity hall at WPI campus resized from *4000 x 3000 px* to *800 x 600 px*, the Camera Intrinsic Matrix, *K*,
and *SIFT* keypoints and descriptors for each image.

#### Fundamental Matrix Estimate
The fundamental matrix represents the epipolar geometry between 2 images. The fundamental matrix between 2 cameras with corresponding image points 
x and x’ of same world point X is governed by equation: $x'F^Tx = 0$  
Let the fundamental matrix be defined as below:  
![Fundamental Matrix Description](Assets/Images/FundamentalMatrix.png)  
To find the fundamental matrix between 2 cameras using 2 images of similar scene, we need minimum 8 point matches. To solve for the elements of the fundamental matrix, we need to use the SVD of the A matrix formed using the 8 or more correspondences as shown below:

