# 3D Vision and Robotics

This module connects visual data to geometry and robot perception. Start with camera geometry before combining sensors or integrating a model into a robot workflow.

## Outcomes

- Transform points and poses between coordinate frames.
- Understand camera calibration, depth, point clouds, and geometric reconstruction.
- Design a basic perception pipeline that communicates with a robot system.

## Prerequisites

Python, computer vision basics, and linear algebra. Familiarity with robotics and ROS 2 helps but is introduced where needed.

## Suggested sequence

1. [Coordinate frames and camera geometry](01-coordinate-frames-and-camera-geometry.md)
2. [RGB-D, point clouds, and a robot pipeline](02-rgbd-point-clouds-and-robot-pipeline.md)
3. [ROS 2 vision workflow](03-ros2-vision-workflow.md)
4. [Stereo, visual odometry, SLAM, and point clouds](04-stereo-visual-odometry-slam-and-point-clouds.md)
5. [Point-cloud processing and deep learning](05-point-cloud-processing-and-deep-learning.md)
6. [Point-cloud registration lab](06-point-cloud-registration-lab.md)

## Practice

- Project a 3D point into an image using a calibrated camera model.
- Transform a point from a camera frame to a robot/world frame and verify units.
- Filter a noisy point cloud and compare before/after visualizations.
- Work through the [registration lab](06-point-cloud-registration-lab.md), writing every transform with its source and destination frames.
- Measure end-to-end delay and dropped frames in a small perception pipeline.
- Solve the [3D vision and robotics problem set](PROBLEM_SET_WITH_SOLUTIONS.md), including frame transforms, depth uncertainty, registration ambiguity, and point-cloud splits.

## Checkpoint project

Build a reproducible RGB-D or point-cloud pipeline that identifies a simple object or surface. Include coordinate-frame diagrams, sensor assumptions, calibration details, latency measurements, failure examples, and a simulation-first or otherwise safe evaluation setup.

## Common mistakes to watch for

- Mixing coordinate frames, units, or row/column conventions.
- Ignoring sensor timestamps and treating misaligned streams as simultaneous.
- Assuming a depth sensor is accurate at every distance and surface.
- Treating a perception output as a safe robot command without uncertainty checks.
