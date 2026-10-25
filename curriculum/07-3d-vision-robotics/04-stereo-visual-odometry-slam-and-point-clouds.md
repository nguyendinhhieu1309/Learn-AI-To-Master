# Stereo, Visual Odometry, SLAM, and Point Clouds

Robots need more than image recognition. They estimate geometry, motion, and uncertainty while sensing from a moving platform. This lesson connects camera calibration, stereo depth, point-cloud alignment, visual odometry, and SLAM.

## 1. Camera calibration and image coordinates

The intrinsic matrix describes focal lengths and principal point:

$$
K = \begin{bmatrix}
f_x & 0 & c_x \\
0 & f_y & c_y \\
0 & 0 & 1
\end{bmatrix}
$$

Lens distortion bends the ideal pinhole projection, especially near image edges. Estimate intrinsics and distortion from calibration images with varied board positions and orientations. Reprojection error is useful, but inspect residuals across image locations and validate calibration after changing focus, resolution, or camera mounting.

## 2. Stereo depth

For a rectified stereo pair with baseline $B$, focal length $f$, and horizontal disparity $d=u_L-u_R$, approximate depth is:

$$
Z = \frac{fB}{d}
$$

Depth becomes unstable as disparity approaches zero. If pixel disparity has error $\sigma_d$, first-order uncertainty grows approximately as $\sigma_Z \approx (fB/d^2)\sigma_d$. This explains why distant stereo depth is less precise. Rectification, synchronized capture, known baseline, and matching quality are essential.

## 3. Point clouds and rigid transforms

A 3D point cloud contains points $p_i \in \mathbb{R}^3$, often with color or intensity. Transform a point from camera to robot coordinates with a rigid transform $T=[R\;t;0\;1]$ in homogeneous coordinates. Check the direction encoded by each transform: “camera in world” and “world in camera” are inverses.

Voxel downsampling reduces density; statistical or radius filtering can remove outliers, but may also erase thin structures. Preserve sensor timestamps and frame IDs. A point cloud without coordinate frame and unit metadata is ambiguous.

## 4. Point-cloud registration and ICP

Iterative Closest Point (ICP) alternates between finding correspondences and estimating a transform minimizing alignment error. Point-to-point ICP minimizes squared distances; point-to-plane often converges faster when reliable normals are available.

ICP is local: it needs a reasonable initial pose and overlapping geometry. It can align the wrong repeated structure, fail under large motion, or be distorted by moving objects. Use robust losses, correspondence distance limits, outlier handling, and convergence diagnostics. Compare alignment against known motion or independent sensors.

## 5. Visual odometry and SLAM

Visual odometry estimates motion between successive images. A typical pipeline detects or tracks features, rejects outliers with geometric checks, estimates relative pose, and integrates motion. Monocular scale is generally ambiguous without additional information; stereo or depth sensors provide metric scale.

SLAM jointly estimates robot trajectory and a map. Loop closure recognizes a previously visited place and adds constraints that correct accumulated drift. A factor graph represents poses and measurements as variables and constraints, then optimizes their consistency. Real systems fuse camera, IMU, wheel odometry, GNSS, or lidar, each with different failure modes.

## 6. Coordinate frames and time

Use a transform tree with named frames such as `map`, `odom`, `base_link`, `camera_link`, and `camera_optical_frame`. Specify handedness, axis directions, units, and transform convention. A correct matrix applied in the wrong direction produces plausible but wrong motion.

Sensors must be synchronized. During motion, a camera timestamp offset from an IMU or wheel sensor can look like a calibration error. For rolling-shutter cameras, rows are captured at different times, so fast motion can violate a global-shutter model.

## 7. Evaluate robotics perception in context

Measure trajectory error (absolute and relative), drift over distance/time, map consistency, depth error by range, and task-level success. Report sequence-level splits: frames from one trajectory must not be divided randomly between training and test. Evaluate on motion blur, low light, reflective surfaces, dynamic objects, and sensor dropouts.

Latency matters: a perception result arriving after the robot has moved may be less useful than a slightly noisier result delivered on time. Track end-to-end timestamp age, not just neural-network inference time.

## 8. Exercises

1. With $f=700$ pixels, $B=0.12$ m, and $d=42$ pixels, estimate depth. What happens to sensitivity if disparity is halved?
2. ICP converges to a low residual but the robot pose is clearly wrong in a hallway with repeated doors. Explain why.
3. A camera and IMU calibration looks poor only during fast turns. Which timing and rolling-shutter issues should be checked?
4. Why is randomly splitting video frames unsuitable for a SLAM benchmark?

## 9. Check your understanding

1. $Z=700(0.12)/42=2$ m. At half disparity the estimated depth doubles, and disparity error produces larger depth error because sensitivity scales as $1/d^2$.
2. Repeated structure creates incorrect correspondences and local minima; low residual alone does not prove the globally correct pose.
3. Check timestamp offsets, synchronization, IMU integration, rolling shutter, and motion-dependent calibration residuals.
4. Neighboring frames are highly correlated; the model and tuning process may see nearly the same scene in both partitions. Split by trajectory or environment.

## References

- [OpenCV camera calibration and 3D reconstruction](https://docs.opencv.org/4.x/d9/d0c/group__calib3d.html)
- [ORB-SLAM3 paper](https://arxiv.org/abs/2007.11898)
- [PCL registration tutorial](https://pointclouds.org/documentation/tutorials/registration_api.html)
