# Structure-From-Motion
A simple implementation of classical Structure From Motion  
This project was done in collaboration with [@Dushyant Patil]() and under the guidance of [Prof. Nitin J. Sanket](https://nitinjsanket.github.io/index.html)

## What is SfM?
Structure from Motion refers to reconstruction of a 3D scene and simultaneously obtain the camera poses of a monocular camera w.r.t. the given scene.

## Why SfM?
SfM makes it possible to simultaneously determine the camera settings and the 3D structure of a scene by combining 2D images taken from various perspectives. SfM approaches may create a dense point cloud from a series of partially overlapping photos even using a cheap digital camera.   
It is use extensively in geosciences to provide hyperscale landform models n remote or rugged environments where terrestrial laser scanning is limited by equipment portability and airborne laser scanning is limited by terrain roughness causing loss of data and image foreshortening.  
SfM is used in order to properly estimate situations as well as planning and maintenance efforts and costs, control and restoration of Cultural heritage sites.

## What we did?
#### Dataset
We were given 5 distortion corrected images of Unity hall at WPI campus resized from *4000 x 3000 px* to *800 x 600 px*, the Camera Intrinsic Matrix, *K*, and *SIFT* keypoints and descriptors for each image.

#### Fundamental Matrix Estimate
The fundamental matrix represents the epipolar geometry between 2 images. The fundamental matrix between 2 cameras with corresponding image points x and x’ of same world point X is governed by equation:  $x'F^Tx = 0$  
Let the fundamental matrix be defined as below:  
![Fundamental Matrix Description](Assets/Images/FundamentalMatrix.png)  
To find the fundamental matrix between 2 cameras using 2 images of similar scene, we need minimum 8 point matches. To solve for the elements of the fundamental matrix, we need to use the SVD of the A matrix formed using the 8 or more correspondences as shown below:  
![SVD of ieght Correspondences](Assets/Images/EightCorrespondances.png)  
The fundamental matrix derived by solving above equation is supposed to be a rank 2 matrix for perfect matches. But, due to the noise in the image data the fundamental matrix derived might be rank 3 and needs to be set to rank 2 matrix by changing the last diagonal element of the covariance matrix of the SVD of F matrix.  

#### RANSAC Outlier Rejection
To obtain a better estimate of fundamental matrix and remove outliers in the initially found matches (using SIFT feature descriptor here), we perform a RANSAC algorithm. The F matrix with maximum number of inliers and the inliers found using the RANSAC are used further for pose estimation and 3D reconstruction.  

#### Essential Matrix Estimation
The essential matrix between 2 cameras represents the epipolar geometry irrespective of the camera matrix between 2 images capturing same scene at the same time. The essential matrix is the specialization of the fundamental matrix to the case of normalized image coordinates. The relation between the fundamental matrix and the essential matrix is as follows: $K^TFK = 0$  

#### Camera Pose Estimation
We can find the relative camera pose of second camera with respect to the first camera with the help of the essential matrix. The underlying assumption in this pose estimation is that the pose matrix for camera 1 is assumed to be $[I | 0]$. Due to projective ambiguities, we find 4 possible camera poses for the second camera. Let the essential matrix be rewritten as E = UDV T . Let us define a matrix W as follows:  
![W](Assets/Images/W.png)  
Here the W matrix is derived as multiplication of a diagonal matrix with diagonal elements (1, 1, 0) and a skew symmetric matrix. The camera pose can be written as: $P = K*[R | t]$  
Here we can find 4 ambiguous poses of second camera as follows:  
t<sub>1</sub> = $U(:,3)$ and R<sub>1</sub> = $UWV^T$  
t<sub>1</sub> = $-U(:,3)$ and R<sub>1</sub> = $UWV^T$  
t<sub>1</sub> = $U(:,3)$ and R<sub>1</sub> = $UW^TV^T$  
t<sub>1</sub> = $-U(:,3)$ and R<sub>1</sub> = $UW^TV^T$  
If $det(R) = -1$, then the pose is to be corrected by setting $R = -R$, $t = -t$.  

#### Linear Triangulation
Linear triangulation is used to estimate the world points using the camera pose and image points-feature matches. Let the image points using 2 camera describing similar scene $X$ simultaneously be $x$ and $x’$. Let the camera poses of both the cameras be $P$ and $P’$. The pinhole geometry between 2 cameras is as follows:  
$x = PX$, $x' = P'X$  
Using above equations, the cross product between vector $x$ and $PX$ is 0. Using this identity of cross product of 2 parallel vector being 0, we can find following equations:  
![LinearTriangulation](Assets/Images/LinearTriangulation.png)  
Here, $p^{iT}$ are the rows of $P$ matrix. Using both $x$ and $x’$, and $P$ and $P’$, we can form a linear equation $AX = 0$ as follows:  
![LinearTriangulation2](Assets/Images/LinearTriangulation2.png)  
By solving for $X$ using SVD of $A$ matrix, a linear world point estimate using 2 image points $x$ and $x’$ can be found.  

#### Triangulation and cheirality check
The ambiguity for the correct pose of second camera is solved using the cheirality check condition. For cheirality check, the world points are calculated for each possible camera pose of camera 2 and camera 1 pose using linear triangulation as explained above. For the world points X and camera center C the depth positivity/ cheirality condition is checked by multiplying the difference between the world point and camera center by the third column of the rotation matrix for the given camera pose, i.e, r<sub>3</sub>$(X - C) = 0$ where r<sub>3</sub> is the third row of transpose of rotation matrix. Using this condition, the camera 2 pose with most number of points satisfying cheirality condition is selected as the real camera pose.  

#### Non Linear triangulation
The non linear triangulation is performed to optimize the output of linear triangulation between camera 1 and true camera pose of camera 2. The non linear triangulation treats the world points found using linear triangulation as input and optimizes the reprojection error using the pinhole camera model as shown below:  
![NonLinearTriangulation](Assets/Images/NonLinearTriangulation.png)  
The error minimization is done using non linear least squares method with the help of scipy.optimize library.  

#### Perspective n Points Methodology
The objective of the perspective n points problem is to find the camera poses of camera 3, 4, 5 based on camera1, camera2 poses, and image3 points. The first step to solve the PnP problem is to create required input data for image1 and image3. Using RANSAC for fundamental matrix, the inliers between image1 and 3 are found. Using [the inliers output corresponding to image 1 after RANSAC between image 1 and 3] and [the inliers output corresponding to image 1 after RANSAC between image 1 and 2], we can find the common inliers corresponding to image 1. This gives us a set of common world points corresponding to the image 1 and image3. We also get a set of unique image points which solely represent the matches between image 1 and 3 and are not common with other images.  

#### Linear PnP & PnP RANSAC
Using the image points and world points, camera3 pose can be found by solving following linear equations:  
![LinearPnP](Assets/Images/LinearPnP.png)  
As $K$ is already known, the pose of camera i.e. $R$ and $t$ can be found as follows:  
$K[R|t] = γ [p$<sub>1</sub> $p$<sub>2</sub> $p$<sub>3</sub> $p$<sub>4</sub>$]$  
$γR = K^{-1} [p$<sub>1</sub> $p$<sub>2</sub> $p$<sub>3</sub>$]$  
Let $UVD^T$ be the SVD decomposition of the RHS of above equation. Let $D = diag(d$<sub>1</sub> $d$<sub>2</sub> $d$<sub>2</sub>$)$. The SVD cleanup to convert the rotation matrix into an orthogonal matrix, and to recover the scale of translation vector is as follows:  
$R = UV^T$  
$T = (p$<sub>4</sub> $*K^{-1}) / d$<sub>1</sub>  
To get a better estimate of the PnP matrix against the noise present in the input data, we run the RANSAC algorithm. The RANSAC gives The $P3$ matrix which gives maximum number of inliers.  

#### Non Linear PnP
The non linear optimization is used to minimize the re-projection error for image 3 onward. The reprojection error minimization is as shown below:  
![NonLinearPnP](Assets/Images/NonLinearPnP.png)  
The optimized PnP matrices for
camera 3,4,5, etc are further used to find the world points corresponding to the image 3,4,5 etc. The world point estimates are found using the linear triangulation followed by non linear triangulation.  

#### Bundle Adjustment
After computing all the camera poses and 3D points, the bundle adjustment is used to refine the camera pose and world point estimates of all cameras. To improve the speed of the implementation of bundle adjustment, a visibility matrix is used to minimize the reprojection error for the 3D points which are visible to particular cameras only. The optimization problem for visibility matrix can be shown as below:  
![BundleAdjustment](Assets/Images/BundleAdjustment.png)
