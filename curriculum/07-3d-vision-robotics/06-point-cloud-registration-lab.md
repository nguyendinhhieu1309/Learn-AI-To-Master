# Point-Cloud Registration Lab: Frames, Sampling, Normals, and ICP

This lesson connects geometry to a point-cloud workflow. Registration estimates how two partial observations relate; the central challenge is not only solving for a transform, but also choosing valid correspondences and coordinate frames.

## Learning objectives

- Track source and destination frames in a rigid transform.
- Explain voxel sampling, neighborhood normals, and outlier filtering.
- Derive the closed-form rigid alignment step used in point-to-point ICP.
- Evaluate registration with overlap-aware metrics and inspect failure modes.

## Prerequisites

Study [coordinate frames and camera geometry](01-coordinate-frames-and-camera-geometry.md), [RGB-D and point clouds](02-rgbd-point-clouds-and-robot-pipeline.md), and [point-cloud processing](05-point-cloud-processing-and-deep-learning.md).

## 1. A transform always has a direction

Let $p_B$ be a point expressed in frame B. A transform $T_{A\leftarrow B}$ maps it into frame A:

$$
\tilde p_A=T_{A\leftarrow B}\tilde p_B,\qquad
T_{A\leftarrow B}=
\begin{bmatrix}R_{A\leftarrow B}&t_{A\leftarrow B}\\0&1\end{bmatrix}.
$$

The tilde denotes homogeneous coordinates $[x,y,z,1]^\top$. The rotation should satisfy $R^\top R=I$ and $\det(R)=+1$.

### Worked transform

Rotate 90 degrees counter-clockwise around z and translate by $(1,0,0)$:

$$
T_{A\leftarrow B}=
\begin{bmatrix}
0&-1&0&1\\
1&0&0&0\\
0&0&1&0\\
0&0&0&1
\end{bmatrix}.
$$

For $p_B=(1,0,2)$, first rotate to $(0,1,2)$, then translate to $p_A=(1,1,2)$. Write the direction in the variable name; a naked symbol $T$ is easy to invert accidentally.

## 2. Build a stable preprocessing sequence

1. Verify frame, unit, timestamp, and sensor metadata.
2. Remove invalid points and sensor-specific artifacts using documented rules.
3. Voxel-downsample to reduce density while preserving approximate geometry.
4. Estimate normals from local neighborhoods if the later method requires them.
5. Find candidate correspondences using a geometric or learned descriptor.
6. Reject outliers before solving a rigid transform.
7. Refine the transform and evaluate on held-out or independently sampled points.

Voxel size is expressed in the same units as the cloud. A value of 0.05 means 5 cm only if coordinates are in meters.

### Neighborhood normals

For a point and its neighborhood, center the neighbors and compute covariance:

$$
C=\frac1k\sum_{j=1}^{k}(q_j-\bar q)(q_j-\bar q)^\top.
$$

The eigenvector associated with the smallest eigenvalue estimates the local surface normal. This relies on a neighborhood that is locally planar enough and not dominated by noise or mixed surfaces.

Too small a neighborhood gives unstable normals; too large a neighborhood smooths away corners and thin structures. Inspect neighborhoods at multiple locations and scales.

## 3. Point-to-point rigid alignment

Given corresponding source points $p_i$ and destination points $q_i$, solve:

$$
\min_{R,t}\sum_i\|Rp_i+t-q_i\|_2^2
\quad\text{subject to}\quad
R^\top R=I,\ \det(R)=1.
$$

Compute centroids $\bar p$ and $\bar q$, and centered coordinates $x_i=p_i-\bar p$, $y_i=q_i-\bar q$. Form:

$$
H=\sum_i x_i y_i^\top,\qquad H=U\Sigma V^\top.
$$

Then:

$$
R=V\operatorname{diag}(1,1,\det(VU^\top))U^\top,\qquad
t=\bar q-R\bar p.
$$

The determinant correction prevents a reflection. This solves the least-squares transform for the supplied correspondences; it does not find correct correspondences by itself.

## 4. Iterative Closest Point (ICP)

ICP alternates:

1. transform source points using the current estimate;
2. find nearest target points;
3. reject invalid or distant matches;
4. solve the rigid alignment for the remaining pairs;
5. compose the update with the current transform;
6. stop when the update or objective changes little, or a limit is reached.

Point-to-plane ICP minimizes:

$$
\sum_i\left(n_i^\top(Rp_i+t-q_i)\right)^2
$$

where $n_i$ is a target surface normal. It can converge faster near a good pose on smooth surfaces, but is more sensitive to bad normals and initialization.

ICP is local: poor initialization, low overlap, symmetry, repetitive geometry, dynamic objects, and unit mismatches can lead to a plausible but wrong alignment. A small residual over a tiny overlap is not proof of global correctness.

## 5. Metrics and what they do not say

For two point sets $P$ and $Q$, a symmetric Chamfer-style distance is:

$$
d_{\mathrm{ch}}(P,Q)=
\frac1{|P|}\sum_{p\in P}\min_{q\in Q}\|p-q\|_2^2+
\frac1{|Q|}\sum_{q\in Q}\min_{p\in P}\|q-p\|_2^2.
$$

This metric depends on sampling density and can hide a local failure in a large cloud. Also report overlap or inlier ratio, transform error when ground truth exists, and failure rate across scenes. Use thresholds in documented physical units.

For point-cloud learning, report the task metric as well: per-class IoU for segmentation, average precision for detection, and split boundaries by scene or sequence. Random point-level splits can leak nearly identical surfaces.

## Practice with answer checks

1. A LiDAR cloud is in millimeters but voxel size is set to 0.05. What likely happens?
2. Why is the determinant correction required in the SVD alignment?
3. Two clouds overlap by only 10%. Why can nearest-neighbor ICP fail despite a correct mathematical update?
4. How does point-to-plane ICP differ from point-to-point ICP?
5. Why is a low Chamfer distance not enough to declare a robotics pose estimate safe?

### Answers

1. The voxel size is tiny relative to millimeter coordinates and may barely downsample. Convert units or choose a consistent scale.
2. An unconstrained SVD solution may include a reflection with determinant $-1$, which is not a proper rotation.
3. Most nearest-neighbor pairs may be incorrect. The optimizer minimizes error for those pairs and can converge to the wrong local alignment.
4. Point-to-point minimizes Euclidean displacement; point-to-plane minimizes displacement projected onto a target normal.
5. It may hide low overlap, symmetry, local errors, a wrong transform direction, or a dangerous failure in the robot's region of interest.

## Mini-project

Use two partial scans from the same scene. Record the coordinate frames and units; visualize before and after; compare at least two initialization or correspondence strategies; report overlap, residuals, transform plausibility, and failure cases. Include a reproducible sequence for preprocessing and parameter choices.

## Common mistakes

- Applying a transform in the reverse direction.
- Mixing meters and millimeters.
- Recomputing nearest neighbors and correspondences without rejecting outliers.
- Treating ICP convergence as evidence of a globally correct pose.
- Reporting only an average distance without overlap or failure rate.
