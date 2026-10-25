# Point Cloud Processing and Deep Learning

This lesson is a dedicated introduction to 3D point clouds. It covers representation, preprocessing, geometric operations, learning architectures, tasks, metrics, and common failure modes.

## 1. What a point cloud contains

A point cloud is a set of 3D samples, usually $p_i=(x_i,y_i,z_i)$, optionally with color, intensity/reflectance, timestamp, ring index, semantic label, or instance ID. Unlike an image, points are unordered and density is nonuniform. Nearby surfaces may be dense while distant LiDAR returns are sparse.

Always document coordinate frame, axis convention, units, sensor origin, and timestamp. A LiDAR frame, camera frame, robot base frame, and world frame are different coordinate systems. A visually plausible cloud may still have the wrong handedness or transform direction.

## 2. Sensors and sampling geometry

- **Spinning LiDAR** produces range returns across azimuth/elevation angles; density and spacing vary with range and beam pattern.
- **Depth cameras** produce organized depth maps with missing values, multipath errors, and limited range.
- **Stereo** infers depth from disparity and loses precision with distance.
- **Photogrammetry/MVS** reconstructs points from overlapping images and may be scale-ambiguous without references.

Sensor noise is not uniform. Model uncertainty as a function of range and incidence angle when downstream tasks depend on metric accuracy. Synchronize sensors before fusion.

## 3. Preprocessing pipeline

Typical operations include:

1. Transform points to a documented reference frame.
2. Remove invalid, non-finite, and out-of-range returns.
3. Crop a region of interest or apply ground filtering when appropriate.
4. Downsample for compute, often using a voxel grid.
5. Estimate normals or local descriptors for geometric methods.
6. Preserve source IDs or indices when labels must map back to raw points.

Voxel-grid downsampling replaces points within a 3D cell with a representative point. It reduces density but can erase thin objects and boundaries. Statistical outlier removal relies on neighbor-distance distributions; it may remove valid isolated returns from distant objects. Tune with visual inspection and task metrics.

### Open3D mini-lab: inspect and clean a cloud

```python
import open3d as o3d

cloud = o3d.io.read_point_cloud("scene.pcd")
if cloud.is_empty():
    raise ValueError("No points loaded; check the path and file format")

# These values are in the same units as the cloud; 0.05 means 5 cm only if units are meters.
down = cloud.voxel_down_sample(voxel_size=0.05)
filtered, inlier_indices = down.remove_statistical_outlier(
    nb_neighbors=20,
    std_ratio=2.0,
)
print(f"raw={len(cloud.points)}, voxel={len(down.points)}, kept={len(filtered.points)}")
o3d.visualization.draw_geometries([filtered])
```

Change the voxel size and filter settings gradually, then compare the raw and filtered clouds. Do not treat outlier removal as automatically correct: sparse distant returns may be valid. Open3D's voxel operation averages points within each occupied voxel, and its statistical filter removes points based on mean neighbor distance; preserve the indices if annotations must remain aligned.

## 4. Neighborhoods, normals, and geometric descriptors

For each point, neighborhood search finds k nearest neighbors or all points within radius $r$. Radius neighborhoods reflect physical scale but vary in count with density; k-NN fixes count but changes physical extent. Normal estimation fits a local plane, often using PCA of neighborhood covariance. The smallest-eigenvalue eigenvector estimates the surface normal, but orientation may be ambiguous.

Classic descriptors such as FPFH summarize local geometry and can support registration. They are useful baselines and remain relevant when data or compute is limited.

## 5. Registration: ICP in more detail

Given source points $p_i$ and target matches $q_i$, rigid alignment estimates rotation $R$ and translation $t$ minimizing:

$$
\min_{R,t}\sum_i \|Rp_i+t-q_i\|_2^2,
\qquad R^\top R=I,\;\det(R)=1
$$

ICP alternates correspondence search and transform estimation. Point-to-plane variants minimize distance along target normals. Use coarse-to-fine alignment, robust losses, rejection thresholds, and a good initial pose. Check overlap and spatial distribution: a low residual on a small flat patch can still leave motion poorly constrained.

## 6. Point-cloud neural networks

- **PointNet** applies shared pointwise MLPs and symmetric pooling (such as max) to obtain permutation-invariant global features. It handles unordered points directly but may miss fine local structure.
- **PointNet++** builds hierarchical local neighborhoods and aggregates features at multiple scales, improving local geometry modeling.
- **Voxel networks** rasterize space into 3D grids and apply 3D convolutions. They are regular and simple conceptually but memory grows rapidly with spatial resolution.
- **Sparse convolution networks** compute only active voxels, making high-resolution 3D scenes more tractable when occupancy is sparse.
- **Pillar/BEV networks** collapse vertical information into columns or bird’s-eye-view features, enabling efficient LiDAR detection while losing some height structure.
- **Point transformers** use attention over local or global point neighborhoods to model contextual relations, with greater compute and memory cost.

The choice depends on task, point count, resolution, hardware, and latency budget. Compare against a classical or simple voxel baseline.

## 7. Core 3D tasks and metrics

- Classification: one label for an object or cloud; report accuracy and class-balanced metrics.
- Part segmentation: label object parts; report per-class IoU and mean IoU.
- Semantic segmentation: label every point; report class IoU, mean IoU, and point accuracy with care for imbalance.
- 3D detection: predict 3D boxes with class and confidence; evaluate 3D IoU at declared thresholds, AP by class and range.
- Registration: report translation and rotation error, inlier ratio, and success under a defined threshold.

3D box conventions differ: center, size ordering, yaw axis, coordinate frame, and whether dimensions refer to length-width-height. Declare conventions before comparing metrics.

## 8. Data augmentation and sampling

Common augmentations include rotation around the gravity axis, translation, scaling, jitter, point dropout, and beam/range simulation. Preserve gravity when the task depends on upright objects; arbitrary 3D rotation can create impossible scenes. Sampling strategy affects rare objects: uniform random point sampling may omit small objects, while class-aware sampling changes the training distribution and requires deployment-distribution evaluation.

## 9. Evaluation pitfalls

Split by scene, route, drive, building, or sensor session rather than random points or crops. Check train/test overlap, class maps, coordinate transforms, range truncation, and label projection. Evaluate by distance, occlusion, point count, weather, sensor, and object size. For robotics, measure end-to-end delay and stale-data behavior as well as geometric accuracy.

## 10. Exercises

1. Why does voxel downsampling risk removing a thin pole even when average point density remains high?
2. When would radius neighborhoods be preferable to k-NN, and what density problem can radius search create?
3. A point transformer scores well but misses the latency budget. Name two representation/model alternatives and the information tradeoff of each.
4. A detector has good 3D IoU nearby but poor performance far away. What sensor and data effects should be investigated?
5. Why must train/test splits be grouped by route or scene?

## References

- [PointNet paper](https://arxiv.org/abs/1612.00593)
- [PointNet++ paper](https://arxiv.org/abs/1706.02413)
- [Open3D point-cloud tutorial](https://www.open3d.org/docs/latest/tutorial/geometry/pointcloud.html)
- [Point Cloud Library tutorials](https://pointclouds.org/documentation/tutorials/)
- [PointPillars paper](https://openaccess.thecvf.com/content_CVPR_2019/html/Lang_PointPillars_Fast_Encoders_for_Object_Detection_From_Point_Clouds_CVPR_2019_paper.html)
