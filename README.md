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

Power on the Raspberry Pi and connect your computer to the same local network.
From a terminal on your computer (for example, Windows PowerShell), run:

```bash
ssh yg-fastbot@yg-fastbot.local
```

These are this robot's configured username and hostname; substitute your own if
different. If the `.local` name cannot be resolved, use the robot's current LAN IP
instead of `yg-fastbot.local`. On first connection, verify the host fingerprint
before accepting it, then enter your Raspberry Pi user's password when prompted.

A prompt such as `yg-fastbot@yg-fastbot:~$` means commands now run on the Raspberry
Pi. Use that terminal for the robot dependency, build, and launch steps below.
The rosject prompt `user:~$` belongs to the cloud computer, not the Raspberry Pi.
Keep the driver terminal open while using RViz2 or teleoperation; open another
SSH connection if you need a second robot terminal. Run `exit` when finished
with an idle SSH shell.

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

The configuration already includes LaserScan (`/fastbot/scan`), Image
(`/fastbot_camera/image_raw`), and the fixed frame `fastbot_laser_link`.
`bringup.launch.xml` starts the robot drivers; it does not open RViz2.

## Details

See [integration notes](docs/INTEGRATION.md) for device configuration, robot parameters,
package interfaces, preview mode, verification evidence, and known limitations.

## License

Student-authored code uses [Apache-2.0](LICENSE). The lidar driver retains its GPL
license. See [NOTICE](NOTICE) for third-party and template license scope.
