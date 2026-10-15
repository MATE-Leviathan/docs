# Sensors

Every sensor on the ROV connects to the Jetson. Code lives in
`2026-27WaterCode/fishROS_ws/src`.

| Sensor | Measures | Connection | Package | Topic |
| --- | --- | --- | --- | --- |
| DWE exploreHD cameras (x2) | Video | USB, `/dev/video*` | `fish_cam` | `Image`, `Image2` |
| Blue Robotics Bar02 (MS5837) | Depth (m), water temp (°C) | I2C bus 1 | `bar02_pub` | `bar02/depth`, `bar02/temp` |
| Adafruit BNO055 | Orientation, internal temp | I2C, `0x28` | `imu_publisher`, `internal_temp/` script | `imu`, `InternalTempData` |
| ADS1115 ADC | TDS, turbidity, pH voltages | I2C, `0x48` | `sensor_bringup` | `/sensors/water_quality/voltages` |
| Sonoptix Echo | Imaging sonar | Ethernet (RTSP) | `sonoptix_sonar` (submodule) | see its README |
| Serial sensor board | Any JSON fields it sends | USB serial, `/dev/ttyACM0` | `sensor_bringup` | `/sensors/serial` |

## Notes

- **Cameras.** `jetson_combined_launch.py` starts both. `all_sensors.launch.py`
  uses `dual_stream_camera` instead, which publishes full frames on
  `/camera/image_raw` and a 640x480, 10 fps copy on
  `/camera/image_low/compressed` for the tether.
- **Bar02.** Fluid density is set to 1000 kg/m³ (fresh water). Depth hold in
  `stabilization_pub` reads `bar02/depth`.
- **BNO055.** Publishes Euler angles converted to a quaternion at 50 Hz. The
  internal temp reader is a bare script, not a ROS package. The `ping2_sonar`
  package also reads this IMU (on `IMUData`); there is no Ping2 driver despite
  the name.
- **ADS1115.** Channels are TDS on A0, turbidity on A1, pH on A2. It publishes
  raw counts and voltages only; convert to real units downstream. The older
  `ph_pub` node reads pH on A3 and converts with `pH = -5.98 * V + 16.1`.
- **Serial board.** Send one JSON object per line at 115200 baud. Numbers land
  in `numeric_keys`/`numeric_values`, everything else as strings, so new fields
  show up without code changes.

## Running them

`jetson_combined_launch.py` currently runs only the cameras and drivetrain.
`all_sensors.launch.py` runs the camera, Bar02, ADS1115, serial board, depth
hold and drivetrain, and records every topic to a bag in `~/rosbags/`. The
BNO055 is not in it; start it with `ros2 run imu_publisher imu_pub`
(`imu.launch.py` points at `imu_pub.py`, which doesn't exist).

```bash
ros2 launch sensor_bringup all_sensors.launch.py
```

Useful arguments: `enable_sonar:=true`, `enable_foxglove:=true`,
`enable_ads1115_water_quality:=false`, `serial_port:=...`,
`thruster_port:=...`.

Check a sensor is alive with `ros2 topic echo <topic>`.
