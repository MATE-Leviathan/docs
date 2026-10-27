# Ideas not implemented

> **Status: not built.** Nothing here has been tested on the robot.

## 3D reconstruction with imaging sonar and cameras in HoloOcean

The goal was to build 3D reconstructions of underwater scenes in simulation, by
combining an imaging sonar with camera images.

The rough plan:

1. Define a HoloOcean scenario with an imaging sonar, a camera, an IMU and a DVL,
   and record the data to ROS 2 bags.
2. Convert the simulated sonar output to the message format of an existing sonar
   SLAM package. `controller_node/sonar_cam_message_converter.py` is a start on
   this. It publishes Oculus-style sonar pings and odometry.
3. Generate a sonar point cloud, then color it with camera data using odometry.
4. Compare against HoloOcean ground truth.

Approaches considered: use an existing sonar SLAM package, implement a
visual-inertial-sonar method, or add sonar to a visual SLAM pipeline. Neural
rendering methods that fuse camera and sonar data were also looked at, but were
judged too heavy for this project.

The `sonar_cam_message_converter` entry point exists, but the converter also
needs `sonar_oculus`, which the [simulation setup](simulation-setup.md) does not
install. The full original plan is in this file's git history.
