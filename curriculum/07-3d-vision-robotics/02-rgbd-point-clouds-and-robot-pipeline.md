# RGB-D, Point Clouds, and a Robot Pipeline

## Learning objectives

Relate depth images to 3D points, recognize common point-cloud operations, and design a robust perception path into a robot software system.

## 1. Depth images

An RGB-D camera pairs color with a depth measurement per pixel (or approximately aligned pixels). Given pixel `(u,v)` and depth `Z`, invert the pinhole equations:

\[
X=(u-c_x)Z/f_x, \qquad Y=(v-c_y)Z/f_y.
\]

The depth convention may be optical-axis depth or range along a ray; read the sensor documentation. Depth images often contain invalid values (zero, NaN, or a sentinel) and become less reliable on shiny, transparent, dark, or distant surfaces.

## 2. Point clouds

A point cloud is a set of 3D points, sometimes with color, normals, or confidence. Common operations:

- **Voxel downsampling:** replace nearby points in each 3D grid cell with a representative point.
- **Outlier filtering:** remove isolated points, while avoiding removal of valid sparse structures.
- **Normals:** estimate local surface directions; useful for geometry matching.
- **Registration:** align two clouds, often initialized with a rough pose and refined by a method such as ICP.

These methods can fail with low overlap, repeated geometry, motion, or poor initial alignment. Visualize before and after each step and track coordinate frames.

## 3. Sensor synchronization and ROS 2 concepts

In a robot pipeline, image and depth messages need compatible timestamps, frame IDs, and calibration. A transform tree records relationships between frames. A perception node should declare input topics, output type, units, frame, timestamp, and quality/confidence. Use a suitable QoS profile and monitor dropped or delayed messages; a correct algorithm on stale data can still produce a wrong robot action.

## 4. From perception to action

An object detection in pixels is not yet a robot pose. Convert pixels and depth into camera-frame coordinates, transform to robot/world coordinates, check confidence and valid workspace, and pass a timestamped result to planning. Keep safety checks and motion constraints in the robot-control layer. Start in simulation or offline playback before physical actuation.

Measure end-to-end latency: sensor capture → transport → synchronization → preprocessing → inference → transform → decision. Average time alone can hide long-tail delays; measure percentiles and dropped-frame rate.

## Mini exercise

Given `fx=500`, `cx=320`, a pixel `u=370`, and depth `Z=2 m`, the horizontal coordinate is `(370-320)*2/500 = 0.2 m`. Before using it, ask: is the depth in meters, is pixel 370 in the calibrated image, and which frame is the resulting point in?

## Checkpoint project

Build a recorded-data or simulated pipeline that produces a 3D point or object pose. Include a frame diagram, calibration and timestamp assumptions, invalid-depth handling, latency statistics, failure examples, and a clear non-actuating safety boundary.

## Common mistakes

- Assuming RGB and depth are already aligned or synchronized.
- Dropping a frame ID and later guessing the coordinate system.
- Treating all depth pixels as valid measurements.
- Sending perception output directly to motion control without workspace and confidence checks.
