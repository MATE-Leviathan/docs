# Sensors

| Sensor | Measures | Connection | Package |
| --- | --- | --- | --- |
| DWE exploreHD cameras | Video | USB, `/dev/video*` | `fish_cam` |
| Blue Robotics Bar02 (MS5837) | Depth (m), water temp (°C) | I2C bus 1 | `bar02_pub` |
| Adafruit BNO055 | Orientation, internal temp | I2C, `0x28` | `imu_publisher`, `internal_temp/` script |
| ADS1115 ADC | TDS, turbidity, pH voltages | I2C, `0x48` | `sensor_bringup` |
| Sonoptix Echo | Imaging sonar | Ethernet (RTSP) | `sonoptix_sonar` (submodule) |
