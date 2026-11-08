# FastBot

Real-robot drivers for Checkpoint 21. Target: Raspberry Pi 4, Ubuntu 22.04 ARM64,
and ROS 2 Humble. Hardware: Arduino Nano motor controller, N10 lidar, and IMX219 camera.

## Download, dependencies, and build

Use a clean Linux workspace. Do not clone another copy alongside existing packages
with the same names.

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/Cooper-Yu/fastbot.git
source /opt/ros/humble/setup.bash
bash ~/ros2_ws/src/fastbot/fastbot_bringup/tools/prepare_dependencies.sh
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --executor sequential --parallel-workers 1
source install/setup.bash
```

The dependency script needs sudo and only installs software. It does not change
camera boot configuration, reboot, or start motors. Initialize rosdep first on a
host where it has never been configured. See [integration notes](docs/INTEGRATION.md)
for camera configuration and hardware differences.

## Launch

First check the hardware-disabled preview:

```bash
ros2 launch fastbot_bringup bringup.launch.xml \
  serial_enabled:=false lidar_enabled:=false camera_enabled:=false
```

After checking wiring, power, and robot parameters, the course entry point is:

```bash
ros2 launch fastbot_bringup bringup.launch.xml
```

The default entry opens physical devices. It sends no nonzero motor target until
a valid velocity command arrives. Secure the chassis, suspend both wheels, and
use bounded low-speed checks for initial motion. Drivers do not start at boot.

## Packages and interfaces

| Path | Responsibility |
| --- | --- |
| `serial_motor` | Twist conversion, serial targets, encoder feedback, odometry, TF, and wheel joint states |
| `serial_motor_msgs` | Custom motor-command and feedback message definitions |
| `fastbot_bringup` | `bringup.launch.xml`, device configuration, diagnostics, and verification tools |
| `fastbot_description` | Simplified robot model; sensor mounting transforms are approximate |
| `third_party/lslidar` | Pinned upstream N10 source and a recorded shutdown patch |

Main topics: `/fastbot/cmd_vel` (Twist), `/fastbot/scan` (LaserScan),
`/fastbot_camera/image_raw` (Image), `/fastbot/odom` (Odometry), and
`/fastbot/joint_states` (JointState).

Empty serial-port arguments prefer the course udev aliases, then a unique known
USB device. Missing or ambiguous matches are rejected. Override `serial_port`,
`lidar_port`, or `camera_device` when needed. This robot uses a 0.032 m wheel radius,
0.12 m wheel separation, and 2537 encoder counts per revolution. Verify these
measurements for another robot.

## Verification and limits

Evidence includes a six-package build, owned-package tests, an independent clone
build, actual camera/lidar streams, bounded local and rosject keyboard pulses,
stop feedback, device release, and an actual RViz Image/LaserScan visual check.
See [evidence and reproduction details](docs/INTEGRATION.md).

These checks do not certify official grading, ground-navigation accuracy, matched
wheel tracking, or a complete final grading-guide run. The short tests produced
different wheel increments, and the earlier right-wheel startup pauses have no
confirmed root cause. Publishing this repository starts no hardware.

## License

Student-authored code uses [Apache-2.0](LICENSE). The lidar driver retains its GPL
license, and template assets retain their own licenses. See [NOTICE](NOTICE) for scope.
