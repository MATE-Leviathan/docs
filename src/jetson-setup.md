# Jetson startup and Pico control

The topside laptop runs the gamepad code in Docker. The Jetson runs the cameras
and thruster-control ROS node. These commands run on the Jetson, outside the
laptop's container.

## Requirements

- ROS 2 Humble and colcon installed on the Jetson.
- A copy of this repository, with its ROS and Python dependencies installed
  (the drivetrain uses `pyserial`).
- The laptop and Jetson connected through the tether network.
- The thruster Pico connected to the Jetson by USB, with compatible firmware
  already flashed, and the cameras connected.
- The Jetson user has permission to access the Pico's serial device.

For firmware upload and standalone hardware tests, see
[Pico W thruster control](pico-w-thruster-control.md).

## Connect and build

From the laptop, use the Jetson's current address and username:

```bash
ssh ubuntu@JETSON_IP
```

On the Jetson, replace the repository path with its actual location:

```bash
source /opt/ros/humble/setup.bash
cd /path/to/2026-27WaterCode/fishROS_ws
colcon build --symlink-install
source install/setup.bash
```

If you use the sonar, run `git submodule update --init` in the repository first;
`sonoptix_sonar` is a submodule.

Skip the build on later starts unless the code has changed. Source ROS and
`install/setup.bash` in each new terminal.

## Start the hardware code

On the Jetson, identify the thruster Pico's USB serial path:

```bash
ls -l /dev/serial/by-id/
```

Use its actual path in the launch command:

```bash
ros2 launch controller_node jetson_combined_launch.py \
  thruster_port:=/dev/serial/by-id/YOUR_THRUSTER_PICO
```

This launch starts two cameras and the drivetrain. It sends neutral commands to
arm the thrusters. Keep the gamepad neutral and the thruster area clear.
Depth, stabilization, pH, and Foxglove are currently disabled in this launch.

To run the sensors (depth, ADC probes, serial JSON, camera) and record a bag, use
`sensor_bringup`'s `all_sensors.launch.py` instead. It also starts the drivetrain,
so don't run both. See the [ROS reference](ros-reference.md) for its arguments.
For the servos and actuator, see [Auxiliary control](auxiliary-control.md). If
something doesn't work, see [Troubleshooting](troubleshooting.md).

Start the [topside code](docker-setup.md) on the laptop to send driving commands.
The [gamepad map](driving.md) lists the controls. The Jetson stops the thrusters
if commands stop arriving for 0.5 s.
Press `Ctrl+C` in the Jetson launch terminal to stop.
