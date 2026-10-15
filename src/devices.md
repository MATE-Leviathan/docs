# Devices

| Device | Measures | Connection | Package | Category |
| --- | --- | --- | --- | --- |
| DWE exploreHD cameras | Video | USB, `/dev/video*` | `fish_cam` | Common |
| Blue Robotics Bar02 (MS5837-02BA) | Depth (m), water temp (°C) | I2C bus 1 | `bar02_pub` | Research |
| Adafruit BNO055 9-axis IMU | Orientation, internal temp | I2C, `0x28` | `imu_publisher`, `internal_temp/` script | Common |
| ADS1115 ADC | pH, TDS, turbidity probe voltages | I2C, `0x48` | `sensor_bringup`, `ph_pub` | Research |
| Blue Robotics Sonoptix Echo | Imaging sonar | Ethernet (RTSP) | `sonoptix_sonar` (submodule) | Research |
