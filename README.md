# FastBot

ROS 2 drivers for Checkpoint 21: Arduino Nano motors, N10 lidar, and an IMX219 camera.
Target: Raspberry Pi 4, Ubuntu 22.04 ARM64, ROS 2 Humble.

## Download and dependencies

Use a clean workspace without duplicate copies of these packages:

```bash
mkdir -p ~/ros2_ws/src
cd ~/ros2_ws/src
git clone https://github.com/Cooper-Yu/fastbot.git
source /opt/ros/humble/setup.bash
bash ~/ros2_ws/src/fastbot/fastbot_bringup/tools/prepare_dependencies.sh
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
```

The dependency script requires sudo. Initialize rosdep first if it has never been
configured. Complete the [camera setup](docs/INTEGRATION.md#camera-setup-boundary)
before launching; installing dependencies does not configure the camera boot settings.

## Build

```bash
cd ~/ros2_ws
source /opt/ros/humble/setup.bash
colcon build --executor sequential --parallel-workers 1
source install/setup.bash
```

## Launch

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
ros2 topic hz /fastbot/scan
ros2 topic hz /fastbot_camera/image_raw
```

For visual verification in RViz2, add LaserScan for `/fastbot/scan` and Image for
`/fastbot_camera/image_raw`.

## Details

See [integration notes](docs/INTEGRATION.md) for device configuration, robot parameters,
package interfaces, preview mode, verification evidence, and known limitations.

## License

Student-authored code uses [Apache-2.0](LICENSE). The lidar driver retains its GPL
license. See [NOTICE](NOTICE) for third-party and template license scope.
