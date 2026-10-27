# Driving the robot

The topside laptop reads the gamepad (`joy_linux_node`), and `controller_node`
turns it into a `twist` message at 50 Hz. The Jetson's `drivetrain_node` turns
that twist into thruster commands for the Pico. See
[Topside Docker setup](docker-setup.md) and
[Jetson startup](jetson-setup.md) for launching both sides.

## Gamepad map (Xbox layout)

| Input | Action |
| --- | --- |
| Left stick up/down | Forward / backward |
| Left stick left/right | Strafe |
| Right stick left/right | Yaw |
| Left trigger | Ascend |
| Right trigger | Descend |
| A | Low sensitivity (inputs scaled to 0.65) |
| B | Full sensitivity |
| X | Toggle depth hold (publishes `stabilization_toggle`) |
| D-pad up/down | Claw open/close (`claw` topic, `Point.z`) |
| Left / right bumper | Claw rotate (`claw` topic, `Point.y`) |

Depth hold needs `bar02_pub` and `stabilization_pub`. The default
`jetson_combined_launch.py` leaves them out; `jetson_testing_launch.py` starts
them. Without them, X does nothing useful. The `claw` topic is published but nothing in the default launch files
uses it. The bumpers also toggle the servos if you run
[auxiliary control](auxiliary-control.md).

Trigger axes rest at `1` and read `-1` when fully pulled. `controller_node`
ignores a trigger until it has moved once, so a pad that reports `0` at rest does
not drive the robot on connect. If your gamepad numbers its triggers differently,
set `left_trigger_axis` and `right_trigger_axis` on `controller_node`.

## Thruster layout

Six thrusters on the thruster Pico:

| Thruster | Pico GPIO |
| --- | --- |
| Back left | 0 |
| Middle right | 1 |
| Front right | 2 |
| Middle left | 3 |
| Back right | 6 |
| Front left | 7 |

The four corner thrusters handle forward, strafe, and yaw. The two middle
thrusters handle vertical motion (and roll, if a twist asks for it).

## Safety behavior

- Stick inputs under 0.05 are treated as zero.
- Thruster output is scaled to 0.8 of full range. If several inputs combine to
  more than full power, the thrusters are scaled down together, keeping their ratio.
- If `/joy` goes quiet for 0.5 s, topside sends a zero twist.
- If the Jetson gets no twist for 0.5 s (`twist_timeout` parameter), it zeros
  every thruster. This covers a dropped tether or a crashed topside launch.
- On startup the drivetrain holds neutral for about 6 s while the ESCs arm. Keep
  the gamepad neutral and the area clear.
- If the Pico stops reading, the drivetrain drops frames instead of blocking. If
  it is unplugged, the drivetrain tries to reopen the port.
- On shutdown (Ctrl+C) the drivetrain sends neutral before closing the port.
