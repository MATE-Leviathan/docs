# ROS reference

## Launch files

| Launch file | Runs on | Starts |
| --- | --- | --- |
| `controller_node topside_combined_launch.py` | Topside | `joy_linux_node` (20 Hz autorepeat), `controller_node` |
| `controller_node jetson_combined_launch.py` | Jetson | Two DWE cameras, `drivetrain_node` |
| `controller_node auxiliary_control_launch.py` | Jetson | `joy_linux_node`, `auxiliary_control` |
| `sensor_bringup all_sensors.launch.py` | Jetson | Everything below, plus a rosbag |
| `controller_node jetson_testing_launch.py` | Jetson | Everything in `jetson_combined_launch.py` plus `bar02_pub`, `stabilization_pub`, `ph_pub` and the Foxglove bridge |

`jetson_combined_launch.py` takes `thruster_port` (default `/dev/ttyACM1`).
Pass a `/dev/serial/by-id/` path, since `ttyACM` numbers change with USB order.

### `all_sensors.launch.py`

Starts the serial JSON publisher, the dual-stream camera, `bar02_pub`, the
ADS1115 water-quality publisher, `stabilization_pub`, and `drivetrain_node`. It
also records a bag with `ros2 bag record -a`.

| Argument | Default | Meaning |
| --- | --- | --- |
| `serial_port`, `serial_baud`, `serial_timeout` | `/dev/ttyACM0`, `115200`, `1.0` | Sensor Pico serial link |
| `thruster_port` | `/dev/ttyACM1` | Thruster Pico |
| `camera_device` | `0` | Video device index for `dual_stream_camera` |
| `enable_ads1115_water_quality` | `true` | Run the ADS1115 publisher |
| `ads1115_i2c_address` | `72` (`0x48`) | ADC address, as a decimal integer |
| `ads1115_gain`, `ads1115_sample_period` | `1.0`, `0.5` | ADC gain, seconds between samples |
| `tds_channel`, `turbidity_channel`, `ph_channel` | `0`, `1`, `2` | ADC input for each probe |
| `enable_sonar` | `false` | Also launch `sonoptix_sonar echo.launch.py` |
| `enable_foxglove` | `false` | Also launch the Foxglove bridge |
| `bag_output` | `~/rosbags/leviathan_<timestamp>` | Bag directory |

The bag records every topic, including raw camera images, so it grows fast. Check
free disk space on the Jetson before a long run, and delete old bags you don't need.

## Topics

| Topic | Type | Publisher | Used by |
| --- | --- | --- | --- |
| `joy` | `sensor_msgs/Joy` | `joy_linux_node` | `controller_node`, `auxiliary_control` |
| `twist` | `geometry_msgs/Twist` | `controller_node` | `drivetrain_node`, `stabilization_pub` |
| `claw` | `geometry_msgs/Point` | `controller_node` | Nothing by default |
| `stabilization_toggle` | `std_msgs/Bool` | `controller_node` | `stabilization_pub` |
| `stabilization` | `geometry_msgs/Twist` | `stabilization_pub` | `drivetrain_node` |
| `stabilization_status` | `std_msgs/Int32` | `stabilization_pub` | |
| `bar02/depth` | `std_msgs/Float32` | `bar02_pub` | `stabilization_pub` |
| `bar02/temp` | `std_msgs/Float32` | `bar02_pub` | |
| `imu` | `sensor_msgs/Imu` | `imu_pub` | |
| `InternalTempData` | `std_msgs/Int32` | `imu_pub` | |
| `pH` | `std_msgs/Float32` | `ph_pub` | |
| `Image`, `Image2` | `sensor_msgs/Image` | `DWE_exploreHD_pub`, `DWE_exploreHD_pub2` | |
| `/camera/image_raw` | `sensor_msgs/Image` | `dual_stream_camera` | |
| `/camera/image_low/compressed` | `sensor_msgs/CompressedImage` | `dual_stream_camera` (640x480, 10 fps) | |
| `/sensors/serial` | `leviathan_sensor_msgs/SerialReading` | `serial_json_publisher` | |
| `/sensors/water_quality/voltages` | `leviathan_sensor_msgs/WaterQualityVoltages` | `ads1115_water_quality_publisher` | |

`WaterQualityVoltages` carries `tds_voltage`, `turbidity_voltage` and `ph_voltage`
(float32), plus the matching `_raw` ADC counts (int32). `SerialReading` carries
the raw line and its parsed JSON: `numeric_keys`/`numeric_values`,
`string_keys`/`string_values`, `parse_ok` and `error`.

## Network

- The Jetson Orin Nano's address is `10.50.2.103` (`ssh ubuntu@10.50.2.103`).
  Confirm it before a run, since it can change.
- The laptop and Jetson talk over the tether network. ROS 2 discovery needs both
  on the same subnet, and the topside container uses host networking.
- To view data, run the Foxglove bridge on the Jetson (it was faster there) and
  connect with Foxglove Studio from the laptop.
