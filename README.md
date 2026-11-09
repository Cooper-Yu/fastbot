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

**From the rosject (course workflow):** first connect the powered-on robot through
The Construct, then check the SSH destination in a rosject terminal:

```bash
getent hosts master
```

If it resolves to the robot's platform address, log in using the Raspberry Pi
account. The course example is `ssh fastbot@master`; this robot's configured
account is `yg-fastbot`, so use:

```bash
ssh yg-fastbot@master
```

If `master` resolves to loopback (`::1` or `127.x.x.x`), it points to the rosject
itself, not the robot. Recheck the platform connection and address mapping before
trying SSH again. `Connection refused` occurs before password authentication;
changing the username does not fix it. This result alone does not identify the
platform fault or prove ROS message transport is broken. Do not edit `hosts` or
restart the robot as a first response.

**Local-network fallback:** from your own computer on the same LAN as the robot
(for example, Windows PowerShell), use:

```bash
ssh yg-fastbot@yg-fastbot.local
```

Substitute your own username/hostname if different, or use the robot's current
LAN IP if `.local` cannot be resolved. This LAN name is not the rosject entry.
Windows and existing SSH sessions do not need to be closed to use the rosject.

On first login, verify the host fingerprint before accepting it, then enter the
Raspberry Pi user's password when prompted. A prompt such as
`yg-fastbot@yg-fastbot:~$` identifies the robot shell: use it for dependency,
build, and driver-launch commands. The prompt `user:~$` identifies the rosject
shell: use it for cloud teleoperation and RViz2. Connecting through the platform
enables communication; it does not turn an existing cloud shell into a robot shell.
Keep the driver SSH terminal open while using the robot; use another terminal
for teleoperation/RViz2, and `exit` only from an idle SSH shell when finished.

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
