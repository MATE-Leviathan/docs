# Troubleshooting

| Symptom | Likely cause and fix |
| --- | --- |
| `drivetrain_node` fails with a serial error at startup | Wrong port. `ttyACM0`/`ttyACM1` swap with USB order. Run `ls -l /dev/serial/by-id/` and pass that path as `thruster_port`. |
| Permission denied on the serial port | Add the Jetson user to the `dialout` group, then log out and back in. |
| Thrusters never respond | Check the Jetson log for `Receiving twist, thrusters live`. If it only says `No twist`, topside isn't reaching the Jetson. Check the tether, subnet, and that `topside_combined_launch.py` is running. |
| Thrusters stop after about half a second | The watchdog fired. Either `/joy` or `twist` stopped arriving. Check the gamepad and the network. |
| `controller_node` warns `No joy input` | Gamepad unplugged, or `joy_linux_node` can't see it. Check `ros2 topic echo /joy`. |
| Robot moves vertically on connect | Trigger axes differ on your gamepad. Set `left_trigger_axis` and `right_trigger_axis`; see [Driving the robot](driving.md). |
| X button does nothing | Depth hold needs `bar02_pub` and `stabilization_pub`, which `jetson_combined_launch.py` doesn't start. Use `jetson_testing_launch.py`. |
| Thrusters don't arm | Wait the full 6 s after launch. Verify the Pico has compatible firmware and the ESCs have power. |
| `auxiliary_control` not found | Rebuild: `colcon build --symlink-install --packages-select controller_node`, then source `install/setup.bash`. |
| Sonar launch fails | The `sonoptix_sonar` submodule is empty. Run `git submodule update --init` in the repository. |
| ADC readings missing | Check I2C wiring and that the address matches (`i2cdetect -y 1`; default `0x48`). |
| Rebuild doesn't pick up changes | Source `install/setup.bash` in each new terminal, and rebuild after adding an entry point. |
