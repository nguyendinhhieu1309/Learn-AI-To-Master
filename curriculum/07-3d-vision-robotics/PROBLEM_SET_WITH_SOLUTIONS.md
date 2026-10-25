# 3D Vision and Robotics: Worked Problem Set

Use column vectors and homogeneous transforms unless stated otherwise. Write frame subscripts while solving; many transform errors are direction errors.

## 1. Transform direction

T_A_B maps points expressed in frame B to coordinates in frame A. Write the equation.

### Solution

p_A = T_A_B p_B. The right subscript is the source frame; the left is the destination. State this convention in code and diagrams.

## 2. Compose a transform chain

Given T_base_end and T_end_camera, find the transform from camera coordinates to base coordinates.

### Solution

T_base_camera = T_base_end T_end_camera. With column vectors, the rightmost transform acts first. Verify using a known point.

## 3. Invert a rigid transform

For T=[R t; 0 1], give its inverse.

### Solution

For a rigid rotation, R inverse = R transpose. Thus T inverse=[R transpose, -R transpose t; 0,1].

## 4. Stereo disparity to depth

A rectified stereo camera has f=400 pixels, baseline B=0.12 m, disparity d=24 pixels. Estimate depth Z.

### Solution

Z=fB/d=400×0.12/24=2 m. Calibration and disparity uncertainty matter; depth uncertainty grows rapidly at small disparity.

## 5. Voxel downsampling of a surface

A point cloud covers a 2 m by 2 m planar region. Cubic voxels have side 0.1 m. About how many are occupied, ignoring boundaries and gaps?

### Solution

Area is about 4 square meters. Each voxel intersects the plane over about 0.01 square meters, giving about 400 occupied cells. A volumetric estimate is wrong because points lie on a surface.

## 6. Estimate a surface normal

Local covariance eigenvalues are 0.02, 0.01, and 0.0001. Which eigenvector estimates the normal, and what does the spectrum suggest?

### Solution

The smallest-eigenvalue eigenvector estimates the normal because variation is smallest perpendicular to the local plane. The spectrum suggests a locally planar neighborhood. Normal orientation needs a viewpoint or propagation rule.

## 7. Kabsch reflection correction

An SVD alignment produces determinant(R)=-1. What correction is needed?

### Solution

The raw result is a reflection. Insert D=diag(1,1,det(V U transpose)) and compute R=V D U transpose for H=U S V transpose. This enforces determinant(R)=+1. Check covariance conventions in code.

## 8. ICP ambiguity

ICP reports a low residual for two repeated parallel shelves, but pose is wrong along the shelf direction. Why?

### Solution

Repeated geometry weakly constrains motion along the symmetry direction, so multiple poses have similar nearest-neighbor distances. Low residual does not prove unique global alignment. Add distinctive geometry, a pose prior, broader context, or independent verification.

## 9. RANSAC iteration count

A model needs 3 points per sample. Inlier fraction w=0.5 and desired probability p=0.99. Use N=log(1-p)/log(1-w^s).

### Solution

All-inlier sample probability is 0.5^3=0.125. N=log(0.01)/log(0.875)≈34.5, so use at least 35 iterations under the assumptions. If w is overestimated, actual success probability is lower.

## 10. Chamfer distance limitation

Two clouds have small symmetric Chamfer distance. Does this prove one-to-one correspondence or identical topology?

### Solution

No. Chamfer measures average nearest-neighbor proximity in both directions. It does not establish one-to-one correspondence, topology, semantic correctness, or absence of local errors hidden by an average. Also inspect coverage, outliers, and task-specific geometry.

## 11. Transform a point

A point is p_camera=[1,0,0] m. Camera translation in the base frame is [2,0,0] m, with no rotation. Find p_base.

### Solution

Using T_base_camera, p_base=[3,0,0] m. Confirm transform direction before applying translation.

## 12. Split point-cloud scans

You create 100 augmented clouds from each of 20 scans, then randomly split all clouds. Why is this invalid for testing unseen scans?

### Solution

Descendants of one scan can land in both partitions and share geometry and sensor artifacts. Split by original scan, scene, or capture session first; augment training only.

## 13. Depth uncertainty

For Z=fB/d, differentiate with respect to disparity d. How does sensitivity change as d becomes small?

### Solution

dZ/dd=-fB/d^2. Its magnitude increases as disparity shrinks, so distant points are more sensitive to the same disparity error. Report valid range and uncertainty.

## 14. Robot safety

A point-cloud detector reports high class probability for an obstacle. Is this enough to command a robot around it?

### Solution

No. Class confidence does not establish geometric accuracy, free-space safety, calibration, obstacle motion, or collision-free motion. Validate localization and perception, propagate uncertainty, use a separately checked planner, and define a safe stop.

## Review checklist

- Are transform names explicit about source and destination?
- Are calibration, synchronization, and coordinate conventions documented?
- Does registration use robust initialization and outlier handling?
- Are ambiguity, density, occlusion, and uncertainty measured?
- Are point-cloud splits grouped by original scene or scan?
- Is perception separated from validated planning and safety?
