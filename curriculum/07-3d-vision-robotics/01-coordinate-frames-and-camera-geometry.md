# Coordinate Frames and Camera Geometry

## Learning objectives

Describe a 3D point in a coordinate frame, transform it between frames, and project it into an image using a pinhole camera model.

## 1. A coordinate is meaningless without a frame

The point `(1, 0, 0)` may refer to one meter forward from a camera, robot base, or world origin. A frame identifies the origin and axis directions. Name frames explicitly: `p_camera`, `p_base`, `p_world`. Also state units (meters vs. millimeters) and right- or left-handed convention.

## 2. Rigid transforms

A rigid transform rotates and translates a point without changing its shape. Using homogeneous coordinates:

$$
\begin{bmatrix}p_A\\1\end{bmatrix} =
T_{A\leftarrow B}\begin{bmatrix}p_B\\1\end{bmatrix}, \qquad
T = \begin{bmatrix}R&t\\0&1\end{bmatrix}.
$$

`T_{A<-B}` means “convert coordinates expressed in frame B into frame A.” The rotation `R` is 3x3 and translation `t` is 3x1. Transform composition follows order: `T_{A<-C} = T_{A<-B} T_{B<-C}`. Matrix multiplication is not commutative; reversing transforms changes the result.

## 3. Pinhole camera projection

For a camera-frame point `(X,Y,Z)` with `Z > 0`, normalized coordinates are `x=X/Z`, `y=Y/Z`. Pixel coordinates are approximately

\[
u=f_x x+c_x, \qquad v=f_y y+c_y.
\]

`f_x,f_y` are focal lengths in pixel units; `(c_x,c_y)` is the principal point. In matrix form, `s[u,v,1]^T = K[X,Y,Z]^T`, with intrinsic matrix `K`. Lens distortion bends this ideal projection; calibration estimates intrinsics and distortion parameters for a particular camera/focus/resolution setup.

## 4. Calibration and uncertainty

Calibration estimates how a camera maps known 3D geometry to image points. Use a well-observed calibration target at varied positions and orientations. Inspect reprojection error, but remember that a low average can hide poor corners or a mismatched image resolution. Recalibrate after changing lens/focus or camera geometry.

Depth uncertainty grows with distance for many sensors. A projected point can shift substantially when depth or camera pose is inaccurate. Carry uncertainty through the system when decisions depend on precise placement.

## Worked frame transform

If a camera is translated 0.2 m forward of the robot base with no rotation, then a point at camera coordinates `(0, 0, 1)` is at base coordinates `(0.2, 0, 1)` under that stated convention. A different axis definition changes the signs/directions. Draw the frames before writing matrices.

## Small projection example

```python
import numpy as np

fx, fy = 500.0, 500.0
cx, cy = 320.0, 240.0
X, Y, Z = 0.2, -0.1, 2.0  # meters in the camera frame; Z must be positive
u = fx * X / Z + cx
v = fy * Y / Z + cy
print(u, v)  # (370.0, 215.0) pixels
```

This ideal pinhole calculation ignores lens distortion. Before comparing with a real image, confirm the calibration belongs to this camera and resolution and that the point is expressed in the camera optical frame.

## Practice

Draw camera and robot-base axes. Write the transform direction in words, compose two transforms, and verify with an easy point such as the origin. Projection check: if `X=0`, the point lies on the center vertical image line `u=c_x` (ignoring distortion).

## Common mistakes

- Using a transform in the wrong direction or multiplying in the wrong order.
- Mixing degrees and radians, millimeters and meters, or coordinate handedness.
- Treating pixel coordinates as metric coordinates.
- Applying calibration from one resolution to another without scaling/checking intrinsics.
