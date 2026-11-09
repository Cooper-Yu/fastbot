# FastBot

ROS 2 drivers for Checkpoint 21: Arduino Nano motors, N10 lidar, and an IMX219 camera.
Target: Raspberry Pi 4, Ubuntu 22.04 ARM64, ROS 2 Humble.

## Where to run commands

- **Robot (Raspberry Pi, via SSH):** install dependencies, build, and launch the drivers.
- **Connected rosject (ROS 2 Humble):** run teleoperation and RViz2. If it already
  provides `teleop_twist_keyboard` and RViz2, only download this repository for
  the RViz configuration; skip the robot dependency, build, and launch sections.
  Connect the robot through The Construct before viewing data or sending commands.

## Connect to the robot via SSH

### Local computer (current workflow)

Connect your computer to the same LAN as the Raspberry Pi. From a local terminal
(for example, Windows PowerShell), use either format:

```bash
ssh USERNAME@HOSTNAME.local
ssh USERNAME@ROBOT_IP
```

Replace `USERNAME` with the Raspberry Pi login account, `HOSTNAME` with its configured
hostname, or `ROBOT_IP` with its current LAN IP. These names are not fixed by this
project. For example, this robot uses `yg-fastbot` for both its username and hostname:

```bash
ssh yg-fastbot@yg-fastbot.local
```

Use the IP form if the `.local` name cannot be resolved.
After login, run the robot dependency, build, and launch steps in this SSH
terminal. Keep bringup running there. In the connected rosject, use separate
terminals for topic checks, RViz2, and keyboard teleoperation.

### Rosject (course workflow)

Connect the robot through The Construct, then run the course command in a
rosject terminal:

```bash
ssh fastbot@master
```

This robot's configured account is `yg-fastbot`, so its corresponding command is:

```bash
ssh yg-fastbot@master
```

**Observed issue (2026-11-09, rosject 1016322):** after reconnecting, `master`
still resolved to `::1` (the cloud machine itself), and SSH returned
`Connection refused`. Direct access from the cloud to the robot's tunnel address
received an SSH server response. This identifies an incorrect `master` mapping;
the platform's underlying cause and a permanent fix remain unverified.

Until that mapping is corrected, use local SSH to start the robot drivers and
the connected rosject for topic checks, RViz2, and teleoperation. The SSH alias
failure alone does not establish whether ROS messages are passing: confirm live
sensor data before attempting motion. Use only one teleoperation command source.

## Download (robot or rosject)

Use a clean workspace without duplicate copies of these packages. Run this on
each machine that needs a copy; cloning on the robot does not download it into
the rosject:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/Cooper-Yu/fastbot.git
```

## Install dependencies (robot only)

Run in the Raspberry Pi SSH terminal. Skip this section in a rosject used only
for teleoperation and visualization:

```bash
source /opt/ros/humble/setup.bash
bash ~/ros2_ws/src/fastbot/fastbot_bringup/tools/prepare_dependencies.sh
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

The dependency script requires sudo. Initialize rosdep first if it has never been
configured. Complete the [camera setup](docs/INTEGRATION.md#camera-setup-boundary)
before launching; installing dependencies does not configure the camera boot settings.

## Build (robot only)

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
colcon build --executor sequential --parallel-workers 1
source install/setup.bash
```

## Launch (robot only)

With wiring, power, and robot parameters checked:

```bash
ros2 launch fastbot_bringup bringup.launch.xml
```

This starts the motor, lidar, and camera drivers. For initial motion checks, secure
the chassis with both wheels suspended and keep hands clear of the wheels.

## Keyboard teleoperation

In another SSH terminal on the robot, or in a ROS 2 Humble rosject connected to it:

```bash
source /opt/ros/humble/setup.bash
ros2 run teleop_twist_keyboard teleop_twist_keyboard --ros-args --remap cmd_vel:=/fastbot/cmd_vel
```

Before moving, press `z` repeatedly to reduce both displayed speeds for a low-speed
check. Press `k` to stop and `Ctrl+C` to exit. Use only one velocity-command source.

## Verify camera and lidar

With the drivers running, run each check in a separate ROS-enabled terminal on the
robot or connected rosject. Press `Ctrl+C` to finish each check:

```bash
ros2 topic echo /fastbot/scan
ros2 topic echo /fastbot_camera/image_raw
```

For visual verification, keep bringup running on the robot and open another
terminal in the connected rosject. With this repository cloned at
`~/ros2_ws/src/fastbot`, load the supplied RViz2 configuration:

```bash
source /opt/ros/humble/setup.bash
rviz2 -d ~/ros2_ws/src/fastbot/fastbot_bringup/rviz/fastbot_sensors.rviz
```

The configuration includes LaserScan (`/fastbot/scan`), Image
(`/fastbot_camera/image_raw`), and TF with coordinate axes and frame names enabled.
The fixed frame is `fastbot_laser_link`. For the video, briefly show the base,
wheel, lidar, and camera frames; confirm TF has no errors before recording.
`bringup.launch.xml` starts the robot drivers; it does not open RViz2.

## Details

See [integration notes](docs/INTEGRATION.md) for device configuration, robot parameters,
package interfaces, preview mode, verification evidence, and known limitations.

## License

Student-authored code uses [Apache-2.0](LICENSE). The lidar driver retains its GPL
license. See [NOTICE](NOTICE) for third-party and template license scope.
