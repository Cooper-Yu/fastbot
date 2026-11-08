# FastBot integration

Target: Raspberry Pi 4, Ubuntu 22.04 ARM64, ROS 2 Humble. The source repository
lives at `~/ros2_ws/src`. No driver starts automatically at boot.

## Install and build

The repository URL and instructor access are pending publication. Once the URL
is supplied, clone the **whole repository** under a clean workspace's `src/`.
Do not clone a second copy alongside the existing packages.

```bash
source /opt/ros/humble/setup.bash
bash ~/ros2_ws/src/fastbot_bringup/tools/prepare_dependencies.sh
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --executor sequential --parallel-workers 1
source install/setup.bash
```

`third_party/lslidar` contains the manufacturer N10 driver with a recorded graceful-shutdown patch, pinned in
`UPSTREAM.json`, including its original package license declarations and GPLv3
license text. It is built from source; no access to GitHub from the Pi is needed.
Licensing of the original student-authored packages remains undecided and must
be resolved before public redistribution. Do not infer a blanket repository
license from the manufacturer driver's license.

## Check without hardware

```bash
python3 -m pytest -q ~/ros2_ws/src/serial_motor/test/test_odometry.py
ros2 launch fastbot_bringup bringup.launch.xml \
  serial_enabled:=false lidar_enabled:=false camera_enabled:=false
```

Preview mode logs requested motor commands and timeout stops. It does not
fabricate encoder feedback or odometry messages. Publishing endpoints exist,
but measured odometry requires successful samples. Stop with Ctrl+C.

## Start the robot

Do not run this until the robot is secured, the ports are confirmed, and camera
configuration has been completed. The default bringup opens real motor/sensor
devices. It sends no nonzero motor command until a valid Twist is received.

```bash
ros2 launch fastbot_bringup bringup.launch.xml
```

Defaults are specific to the currently inspected robot:

| Item | Value / source |
| --- | --- |
| Namespace | `fastbot` |
| Nano | `/dev/serial/by-id/usb-1a86_USB_Serial-if00-port0`, 57600 baud |
| N10 candidate | `/dev/serial/by-id/usb-1a86_USB_Single_Serial_5B8E671883-if00`, 230400 baud; N10 protocol and actual LaserScan stream verified |
| Camera | `/dev/video0`, IMX219, verified with bm2835 mmal after the course configuration change |
| Wheel radius | 0.032 m, from user-measured diameter |
| Wheel separation | 0.12 m, user measurement |
| Encoder CPR | 2537, common initial estimate from repeated hand rotations |
| Firmware update | 30 Hz nominal |
| Command timeout | 0.5 s |
| Target limit | 50 counts/update, initial limit, not a calibrated ground speed |

Override `serial_port`, `lidar_port`, `camera_device`, or `robot_name` in the
top-level launch as needed. The motor-only launch additionally exposes geometry,
CPR, controller rate and command limits. For another robot, verify these instead
of assuming the current serial-number path or measurements apply.

## Interfaces and frames

| Interface | Type | Purpose |
| --- | --- | --- |
| `/fastbot/cmd_vel` | `geometry_msgs/msg/Twist` | Requested forward/yaw motion |
| `/fastbot/encoder_vals` | `serial_motor_msgs/msg/EncoderVals` | Cumulative signed int32 counts |
| `/fastbot/motor_vels` | `serial_motor_msgs/msg/MotorVels` | Measured wheel rad/s |
| `/fastbot/odom` | `nav_msgs/msg/Odometry` | Wheel-based pose/velocity estimate |
| `/fastbot/joint_states` | `sensor_msgs/msg/JointState` | Measured wheel positions/velocities |
| `/fastbot/scan` | `sensor_msgs/msg/LaserScan` | Required lidar output |
| `/fastbot_camera/image_raw` | `sensor_msgs/msg/Image` | Required camera output |

The motor node publishes `fastbot_odom -> fastbot_base_link`; robot state
publisher supplies the wheel and sensor transforms. Wheel estimates use actual
monotonic sample intervals, while output messages use the ROS clock. The first
encoder sample establishes a baseline and does not invent motion. Pose uses
the course's starting-heading approximation. Wheel slip and external movement
are not independently measured. Covariance values are initial estimates.

The URDF is a simplified visualization, not the course CAD export. Wheel radius
and track match the current measured parameters; chassis dimensions and sensor
mount translations are approximate and require measurement before spatial
sensor fusion or navigation. No collision/inertial simulation claim is made.

## Teleoperation and stop behavior

On a machine with ROS 2 Humble, the same ROS domain, reachable DDS network and
no competing velocity publisher:

```bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args \
  --remap cmd_vel:=/fastbot/cmd_vel
```

Start with bounded low speed and wheels clear of obstacles. Accepted commands
must arrive more often than the 0.5 s timeout. The node requests zero after a
command timeout and during normal exit. Communication failures latch a fault;
after a broken connection, a requested stop is **not proof of a physical stop**.
The firmware's independent command timeout is an additional layer; test the
whole system before ground movement. Do not depend on closing a terminal as an
emergency stop.

## Camera setup boundary

Current boot configuration has `camera_auto_detect=1` and the IMX219 uses
`unicam`. Course unit 9.4 uses the legacy mmal stack with
`camera_auto_detect=0` and `start_x=1`, followed by a reboot. Back up and review
`/boot/firmware/config.txt` before any such change. Do not copy the whole sample
config, blindly remove overlays, or reboot while package installation is running.
The user applied the reviewed settings and rebooted on 2026-11-08; video0 was then verified as bm2835 mmal. Other robots must review their own boot configuration.

## Acceptance still required

Software tests and preview launch cannot replace physical acceptance. Complete
assembly/wiring inspection, Twist/keyboard response, external rosject control,
real scan/image streaming, clone/build on the review environment, instructor
repository access, and a publicly shared demonstration video of at most three
minutes. Record each separately; no official score or learner PASS is inferred.


## Reviewed camera configuration helper
Run `python3 ~/ros2_ws/src/fastbot_bringup/tools/configure_legacy_camera.py` to inspect the exact proposed boot diff. It is read-only by default. Apply requires sudo, `--apply`, and `--expected-sha256` with the checksum printed by review. The helper refuses ambiguous settings and changed input, writes a timestamped backup, and does not reboot. Reboot manually after all builds/package installations have finished. Legacy firmware/module presence was verified on this Pi; successful image capture still requires a post-reboot test.


## Sensor streaming verification (2026-11-08)
On this robot, both required sensor topics produced real messages after the reviewed camera boot change: 320x240 rgb8 images at about 30 Hz and N10 LaserScan at about 10 Hz. Motor serial access was disabled. The original N10 SIGINT handler called abort; the recorded patch removes that override, cleans up its publisher thread and avoids constructing a fresh executor during context shutdown. Both original failures and the final clean-exit run are retained in the training verification log. The camera is not intrinsically calibrated; missing camera calibration and unsupported control-read warnings were not hidden or replaced with invented calibration data. This sensor check does not establish motor, external-computer or rosject acceptance.


## Keyboard release and bounded checks
The installed ROS 2 keyboard package exposes `speed` and `turn`; this version does not implement ROS 1-style repeat_rate/key_timeout parameters. Use `-p speed:=0.03 -p turn:=0.3`. Each key event sends a Twist; the motor driver retains its independent 0.5 s command timeout. Press `k` to request zero immediately and Ctrl+C to exit. Software PTY testing verified i/j/k/exit on a random topic with no hardware. A separately approved single suspended-wheel ROS pulse passed (one m12/12, about 0.510 s to zero, final feedback zero and serial released); this is not yet the actual keyboard-on-robot or external rosject acceptance. The bounded keyboard harness defaults to fake mode; never set real mode without a fresh explicit motion authorization. Its audited stimulus/watchdog/cleanup orchestrator has a documented statement-count exemption; no automatic lint-rule disabling was applied elsewhere.
