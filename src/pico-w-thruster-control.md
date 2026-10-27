# Pico W thruster control

Use [pico-w-thruster-control](https://github.com/MATE-Leviathan/pico-w-thruster-control)
to upload Pico firmware and test thrusters, servos,
and an actuator from the Jetson over USB. Run these commands on the Jetson,
outside Docker. Close any ROS launch or serial tool using the Pico first.

## Setup

1. Install Python 3.10 or newer and `uv` on the Jetson.
2. With the Pico W disconnected, hold BOOTSEL while plugging it into USB.
   Copy the Pico W `.uf2` from the repository's `assets/firmware/` folder onto
   the `RPI-RP2` drive. The Pico will restart with MicroPython installed.
3. Edit `config/rov.json` to match your wiring and output limits. Pin numbers
   are GPIO numbers: `12` means GP12. The current configuration has one
   thruster on GP12; servos and the actuator are optional.

## Configure outputs

Edit `config/rov.json` in `pico-w-thruster-control`. The same file configures
the diagnostic menu and, after uploading, the Pico. `config/rov.schema.json`
defines the allowed fields; you normally only need to edit `rov.json`.

The current configuration is:

```json
{
  "thruster_neutral": 0.5,
  "thruster_pulse_ms": { "min": 1.1, "max": 1.9 },
  "servo_pulse_ms": { "min": 0.7, "max": 2.2 },
  "thrusters": {
    "thruster_1": { "pin": 12 }
  }
}
```

| Field | Meaning |
| --- | --- |
| `thruster_neutral` | Stop value sent by arming and neutralizing. Between `0` and `1`; currently `0.5`. Match this to your ESC. |
| `thruster_pulse_ms` | Pulse widths in milliseconds for thruster values `0` (`min`) and `1` (`max`). With the current limits, `0.5` gives a 1.5 ms pulse. |
| `servo_pulse_ms` | Pulse widths in milliseconds for auxiliary servo values `0` and `1`. Required even when no auxiliary servos are configured. |
| `thrusters` | Named thrusters, each with a GPIO `pin`. Add one entry per thruster. An optional `position` string is a label; it does not change motor behavior. |

Names such as `thruster_1` are your choice and appear in the menu. GPIO numbers
are integers from `0` to `29` in the schema; choose pins available on your board.
Output values between `0` and `1` are relative settings, not milliseconds.
Choose pulse limits supported by your ESCs and servos, with `min` below `max`.

### Optional servos and actuator

Add these fields alongside `thrusters` only if you use them. These are example
pins and limits; adjust them for your hardware:

```json
"aux_servos": {
  "gripper": { "pin": 14, "min": 0.2, "max": 0.8, "end": 0.5 }
},
"actuator": {
  "direction_pin": 10,
  "speed_pin": 11,
  "pwm_freq": 1000
}
```

Add a comma after the existing `thrusters` object before inserting this fragment.

| Field | Meaning |
| --- | --- |
| Servo `pin` | GPIO connected to the servo signal. |
| Servo `min`, `max` | Allowed movement range, each between `0` and `1`. Firmware clamps servo commands to these limits. |
| Servo `end` | Position used at the end of the servo test. Keep it within `min` and `max`. |
| Actuator `direction_pin` | Digital direction signal to the motor driver; the test switches it between `1` and `0`. |
| Actuator `speed_pin` | PWM speed signal to the motor driver; `0` stops it and `1` requests full power. |
| Actuator `pwm_freq` | Speed PWM frequency in Hz, a positive integer supported by your driver. |

Leave unused `aux_servos` and `actuator` fields out, or set them to `null`.
`aux_servos` can also be `{}`. Use distinct GPIOs for each signal. PWM outputs
that share a hardware channel conflict, and outputs sharing a PWM slice must
use the same frequency. Thrusters and servos use 50 Hz; an actuator speed output
at another frequency needs a separate slice.

The upload and diagnostics commands check the schema and PWM conflicts before
running. Unknown fields are rejected. Keep servo `min <= end <= max` and pulse
`min < max`; these ordering rules are not checked by the schema.

## Robot wiring

The `drivetrain_node` expects six thrusters on these GPIOs. A `rov.json` for the
robot, in the same format as above:

```json
{
  "thruster_neutral": 0.5,
  "thruster_pulse_ms": { "min": 1.1, "max": 1.9 },
  "servo_pulse_ms": { "min": 0.7, "max": 2.2 },
  "thrusters": {
    "back_left": { "pin": 0 },
    "middle_right": { "pin": 1 },
    "front_right": { "pin": 2 },
    "middle_left": { "pin": 3 },
    "back_right": { "pin": 6 },
    "front_left": { "pin": 7 }
  }
}
```

The drivetrain sends one line per thruster in the form `z<GPIO><value>x`, for
example `z0000.50x` for GP0 at neutral, with the value running from `0.00` to
`1.00`. Your firmware must accept this format. Servos and the actuator use the
same format from [auxiliary control](auxiliary-control.md), which runs on a
separate Pico, so its GPIOs can overlap with the thruster Pico's.

## Upload and test

From the `pico-w-thruster-control` repository folder:

```bash
uv run upload --port /dev/ttyACM0
uv run diagnostics --port /dev/ttyACM0
```

Dependencies install automatically. Change the port to match your Pico.
On Linux, `ls -l /dev/serial/by-id/` can help identify it. Close diagnostics before
uploading. Upload again after changing the configuration.

Use **Arm / sound test** to send neutral to the selected thrusters, then
**Spin test** for a short forward/reverse test. Servo and actuator tests appear
only when configured. Use **Neutralize thrusters** (or
**Neutralize thrusters / stop actuator**) to stop those outputs.

To try the menu without hardware:

```bash
uv run diagnostics --dry-run
```

Keep the thruster area clear during tests. Disconnecting USB does not guarantee
the motors stop.
