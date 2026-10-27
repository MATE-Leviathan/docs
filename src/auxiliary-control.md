# Auxiliary control (servos and actuator)

`auxiliary_control` drives the servos and the linear actuator from the gamepad.
It talks to a second Pico, separate from the thruster Pico. Run it on the
Jetson, with `joy` messages arriving from topside.

```bash
ros2 launch controller_node auxiliary_control_launch.py \
  port:=/dev/serial/by-id/YOUR_AUX_PICO
```

The default port is `/dev/ttyACM0` and the baud rate is `115200`.

The launch file starts its own `joy_linux_node`. If topside is already
publishing `joy` over the tether, run the executable directly instead so there
is a single joy source:

```bash
ros2 run controller_node auxiliary_control --ros-args -p port:=/dev/serial/by-id/YOUR_AUX_PICO
```

## Controls

| Input | Action |
| --- | --- |
| Left bumper (button 4) | Toggle servo on GP20 between its min and max |
| Right bumper (button 5) | Toggle servo on GP01 between its min and max |
| Right trigger | Actuator forward while held |
| Left trigger | Actuator back while held |

A trigger counts as pressed when its axis drops below `trigger_pressed_threshold`
(default `0.5`). Pressing both stops the actuator. The triggers also drive the
vertical thrusters through `controller_node`, so the robot moves vertically when
you move the actuator.

## Limits and wiring

| Output | Pico GPIO | Range sent |
| --- | --- | --- |
| Servo `20` | GP20 | 0.42 to 0.54 |
| Servo `01` | GP01 | 0.45 to 1.00 |
| Actuator direction | GP21 (`actuator_direction_pin`) | 1 forward, 0 back, 0.5 stop |
| Actuator speed | GP28 (`actuator_speed_pin`) | `actuator_speed` (default 1.0), 0 stop |

The servo limits are constants in `auxiliary_control.py`. Change them there if
the mechanism needs a different range.

Commands are sent as `z<pin><value>x` lines, for example `z200.54x`: a
two-digit GPIO number, then a value from 0 to 1. The actuator is stopped when the
node shuts down.
