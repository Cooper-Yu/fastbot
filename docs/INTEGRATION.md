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

`third_party/lslidar` contains the unmodified manufacturer N10 driver, pinned in
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
| N10 candidate | `/dev/serial/by-id/usb-1a86_USB_Single_Serial_5B8E671883-if00`, 230400 baud; physical identity still to verify |
| Camera | `/dev/video0`, IMX219 currently detected via unicam; v4l2 compatibility not yet verified |
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
No boot edit or reboot has been performed by this integration deployment.

## Acceptance still required

Software tests and preview launch cannot replace physical acceptance. Complete
assembly/wiring inspection, Twist/keyboard response, external rosject control,
real scan/image streaming, clone/build on the review environment, instructor
repository access, and a publicly shared demonstration video of at most three
minutes. Record each separately; no official score or learner PASS is inferred.
