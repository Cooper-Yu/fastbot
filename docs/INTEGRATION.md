# FastBot integration

Target: Raspberry Pi 4, Ubuntu 22.04 ARM64, ROS 2 Humble. The source repository
lives at `~/ros2_ws/src`. No driver starts automatically at boot.

## Install and build

Repository: https://github.com/Cooper-Yu/fastbot. Clone the **whole repository**
under a clean workspace's `src/fastbot/`.
Do not clone a second copy alongside the existing packages.

```bash
source /opt/ros/humble/setup.bash
bash ~/ros2_ws/src/fastbot/fastbot_bringup/tools/prepare_dependencies.sh
cd ~/ros2_ws
rosdep install --from-paths src --ignore-src -r -y
colcon build --executor sequential --parallel-workers 1
source install/setup.bash
```

`third_party/lslidar` contains the manufacturer N10 driver with a recorded graceful-shutdown patch, pinned in
`UPSTREAM.json`, including its original package license declarations and GPLv3
license text. It is built from source; no access to GitHub from the Pi is needed.
The owner selected Apache-2.0 for the student-authored packages. See the root
LICENSE and NOTICE for scope; the manufacturer driver retains its upstream GPL terms.

## Check without hardware

```bash
python3 -m pytest -q ~/ros2_ws/src/fastbot/serial_motor/test/test_odometry.py
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

Geometry defaults use the currently measured robot. Empty motor/lidar port arguments prefer the course udev aliases, then select exactly one matching USB adapter:

| Item | Value / source |
| --- | --- |
| Namespace | `fastbot` |
| Nano | Auto: `/dev/arduino_nano`, otherwise a unique CH340/FTDI adapter; 57600 baud |
| N10 | Auto: `/dev/lslidar`, otherwise a unique current QinHeng or course CP2102 adapter; 230400 baud |
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
Run `python3 ~/ros2_ws/src/fastbot/fastbot_bringup/tools/configure_legacy_camera.py` to inspect the exact proposed boot diff. It is read-only by default. Apply requires sudo, `--apply`, and `--expected-sha256` with the checksum printed by review. The helper refuses ambiguous settings and changed input, writes a timestamped backup, and does not reboot. Reboot manually after all builds/package installations have finished. Legacy firmware/module presence was verified on this Pi; successful image capture still requires a post-reboot test.


## Sensor streaming verification (2026-11-08)
On this robot, both required sensor topics produced real messages after the reviewed camera boot change: 320x240 rgb8 images at about 30 Hz and N10 LaserScan at about 10 Hz. Motor serial access was disabled. The original N10 SIGINT handler called abort; the recorded patch removes that override, cleans up its publisher thread and avoids constructing a fresh executor during context shutdown. Both original failures and the final clean-exit run are retained in the training verification log. The camera is not intrinsically calibrated; missing camera calibration and unsupported control-read warnings were not hidden or replaced with invented calibration data. This sensor check does not establish motor, external-computer or rosject acceptance.


## Keyboard release and bounded checks
The installed ROS 2 keyboard package exposes `speed` and `turn`; this version does not implement ROS 1-style repeat_rate/key_timeout parameters. Use `-p speed:=0.03 -p turn:=0.3`. Each key event sends a Twist; the motor driver retains its independent 0.5 s command timeout. Press `k` to request zero immediately and Ctrl+C to exit. Software PTY testing verified i/j/k/exit on a random topic with no hardware. A separately approved single suspended-wheel ROS pulse passed (one m12/12, about 0.510 s to zero, final feedback zero and serial released); this is not yet the actual keyboard-on-robot or external rosject acceptance. The bounded keyboard harness defaults to fake mode; never set real mode without a fresh explicit motion authorization. Its audited stimulus/watchdog/cleanup orchestrator has a documented statement-count exemption; no automatic lint-rule disabling was applied elsewhere.


## Local hardware keyboard check (2026-11-08)
With separately confirmed suspended wheels, the official teleop_twist_keyboard app received one i key and one k key through a PTY. It produced one nonzero motor frame, 80/78 encoder increments, a stop acknowledgment after about 0.322 s, stable final counts and zero wheel feedback. App/controller resources were closed and the Nano port released. This verifies the local keyboard-to-ROS-to-Nano path; rosject networking and remote teleoperation are still pending. No further motion is authorized by that one-test confirmation.


## Port portability
No hardware serial number is hardcoded in the default launch. Explicit `serial_port` or `lidar_port` overrides take precedence; otherwise existing course udev aliases are preferred, then a unique known USB identity is selected without opening the device. The resolver deduplicates tty/by-id names and rejects zero or multiple matches. Nano IDs are CH340 1a86:7523 and FTDI 0403:6001; lidar IDs are this robot's 1a86:55d4 and the course example CP2102 10c4:ea60. Another adapter requires explicit configuration or a reviewed udev alias; identity selection is not proof of firmware/model compatibility. Seven filesystem-only boundary tests, read-only identification of the two actual ports, six-package build, full owned-package tests and real sensor streaming passed. No additional motor motion was performed for this change.


## Independent clone audit
A clean local Git clone of 93fb3f4 built all six packages from scratch in its own Linux workspace under a cleared environment with only the Humble underlay. The same isolated clone was then updated to d679a11 (including portable device resolution), rebuilt and checked with all four owned-package test suites and an actual hardware-disabled preview launch. There were zero failures, one existing copyright-test skip, and no configured tests for the description-only package. This checks independence from the original workspace build/install overlay; it does not claim a fresh operating system, another physical robot, or official evaluator acceptance.


## The Construct Real Robot Lab connection
Register this robot under the owner account, select ROS 2 Humble and Active, then run the account-specific official installation command on the Pi. Do not commit that command, its credentials, generated configuration files or full container inspection/log output. The setup installs Docker and the theconstruct.rrl systemd service and adds Husarnet DDS profile exports. Reload the shell environment and stop the old ROS daemon before using the new profiles. The robot username here is yg-fastbot; from a connected rosject the course hostname is master, so use ssh yg-fastbot@master, adapting if the platform explicitly shows another endpoint.

On this robot the service is active, hnet0 and both DDS profile files exist, and a host-only ROS String pub/sub probe passes with the platform CycloneDDS profile. The ordinary user cannot access the Docker socket, so the official doctor cannot inspect its container and reports Docker/Husarnet/DDS issues despite the active service. Do not treat its exit code 0 as all checks passing, and do not infer remote rosject connectivity from a host probe. No Docker-group or sudoers change was made by this project. The optional check_rrl.sh helper retains only sanitized diagnostic output. Next connect the robot from the chosen rosject and verify topics before any separately approved remote motion.

## 2026-11-08 — Connected rosject transport and real sensors
The owner connected rosject 1016322 to the robot. A uniquely named String request/reply probe passed in both directions with the configured RRL CycloneDDS profile: the Pi acknowledged five requests and the rosject received its reply. This functionally verifies the network path despite the ordinary user's inability to inspect the Docker container with the official doctor.

The rosject then received 9 real RGB8 images (320x240, 230400 bytes, fastbot_camera_optical_frame) and 5 real LaserScan messages (451 rays in the reported sample, fastbot_laser_link). No motion command was published and the motor serial interface stayed disabled. The concurrent Pi check received 1429 images and 483 scans at approximately 30.01 Hz and 10.01 Hz. Launch exited 0, no abnormal child exits were recorded, and the Nano, lidar and camera device ports were released. Ray counts can vary by revolution; these are observed samples, not a fixed-size requirement.

Remote keyboard-driven physical motion has not yet been tested. It needs a separately reviewed bounded stimulus and fresh onsite confirmation. Repository publication, the downloadable rosject README and the required video remain pending. This milestone does not grant learner or official grading PASS.
## Rosject keyboard-to-motor verification (2026-11-08)
With the owner's new one-test approval and suspended wheels, the official keyboard app in rosject 1016322 sent one forward key and a stop key about 0.3 s later. The host accepted exactly one nonzero serial frame (`m 12 12`); actual encoder increments were 40/24. The first zero frame followed after 0.231603 s, final five samples had stable counts and zero wheel velocities, and the Nano port was released. This demonstrates one remote teleoperation/stop path, not matched wheel speeds or ground navigation accuracy.

The cloud keyboard version hardcodes its initial speed to 0.5 m/s and ignores speed/turn ROS parameter overrides. An initial fake-serial trial rejected its larger frame before any hardware opened. The reviewed cloud helper uses the app's own `z` speed keys 27 times and verifies its printed speed (0.0290748685 m/s) before sending the single forward key. Its command observer also checks the actual one nonzero Twist. Another fake trial corrected a readiness/observer assumption, and a further trial removed an invalid watchdog-state assertion after delayed zero messages. The final fake network check passed (118/118 counts, 0.329008 s stop), then exactly one real trial passed. No repeat real pulse was performed.

Tools: `fastbot_bringup/tools/check_remote_keyboard_host.py`, `remote_keyboard_host.sh`, and `remote_keyboard_cloud.py`. Host mode defaults to fake; the wrapper requires explicit mode/topic arguments. Replace the cloud topic placeholder with a fresh shared `/cp21_remote_...` topic, inspect the script and run fake mode first. Real mode opens the Nano and needs fresh onsite motion authorization. The whitelist, one-nonzero limit, 0.5 s driver timeout, feedback checks and 0.65 s fallback remain active. Do not run this beside another controller. The shell wrapper uses the existing RRL DDS profile and Humble workspace.

Evidence caveat: the original result's `stop_path=watchdog_before_remote_zero` was an incorrect inference from observer callback order. Driver logs show receipt of the remote zero and its serial send before the later 0.5 s timeout; the 0.231603 s first stop was from the remote zero. Raw results are preserved; the helper now requires log review rather than inferring stop cause from callback order. This reporting-only repair did not trigger another motion test.

The course explicitly names the submission repository `fastbot`. Upload/publication and the rosject download README are still pending; no GitHub repository was created and no code was pushed by these checks.
## RViz visual check (2026-11-08)
Both displays were opened in actual rosject RViz: an Image display showed the real camera scene and a LaserScan display showed the surrounding range points. Global Status was OK; two screenshots taken about one minute apart showed changing scan points. This supplements, rather than replaces, the earlier message-level image/scan checks. The camera and lidar ran with `serial_enabled:=false`; no motor port was opened or Twist sent. Screenshots stay in the local verification record and are not published with the source.

A reusable configuration is provided at `fastbot_bringup/rviz/fastbot_sensors.rviz`. With the sensor streams available and the RRL connection active when using a rosject, start it using the path in your clone:

```bash
rviz2 -d ~/ros2_ws/src/fastbot/fastbot_bringup/rviz/fastbot_sensors.rviz
```

It uses `/fastbot_camera/image_raw` with the Image display, `/fastbot/scan` with LaserScan, best-effort/volatile topic QoS and `fastbot_laser_link` as the fixed frame. An Image display checks raw pictures without requiring camera calibration; this is not a calibrated Camera projection or verified sensor extrinsic model.
## Grading-path integration check (2026-11-08)
The public `ab65dbd` revision was downloaded afresh and built in an independent Pi
workspace (all six packages, 4 min 37 s). The Pi's direct GitHub fetch was too slow,
so a fresh anonymous GitHub download on the control host was relayed as a Git
bundle. This was not a second operating system or a successful direct Pi download.

The actual runtime command was `ros2 launch fastbot_bringup bringup.launch.xml`,
without hardware-disable overrides. A rosject ran the official keyboard executable
with `--remap cmd_vel:=/fastbot/cmd_vel`. Camera, lidar and motor feedback were active
together. Default DDS transport was verified after the owner reconnected the robot;
no temporary peer profile was used for the final run.

Four nonzero keys were sent. The first three steps obtained settled encoder deltas:
forward 164/218, backward -182/-144, and left -190/148. The right-turn command also
reached the driver and produced positive-left/negative-right measured wheel speeds.
The Pi received 606 images and 209 scans; the rosject received 240 images and 85
scans during this run.

The complete automated run returned **FAIL**, not PASS: its 0.7 s motion/settling
protection aborted the right-turn step before collecting that step's final stable
samples. Driver logs show a right-turn command at 1794138050.1586, a timeout zero
at 1794138050.7148, and subsequent near-zero feedback. The owner independently
confirmed both wheels stopped. All launch children exited cleanly and all three
hardware device ports were released. The result demonstrates actual four-direction
responses on the grading topic, but does not certify uninterrupted automated
completion, exact stopping latency, matched wheel speeds, or official grading.
The 0.7 s guard is a test protection, not a published grading-guide requirement.

Earlier attempts and failures are retained in the local evidence: stale connection
state prevented default DDS traffic; a temporary peer profile passed small-message
checks but not integration; an early forward pulse arrived only about 54 ms before
its zero and produced no encoder changes. The harness was then changed to wait for
receiver acknowledgment before timing the short hold, and that workflow passed
four-direction fake-feedback checks before the final real run. No controller
implementation or physical speed limit was changed to make the result pass.

All nine delivery READMEs are now English. A tracked-file scan found no Chinese in
those READMEs or in student-authored Python comments/docstrings. Third-party source
and license text retain their original provenance. The required video and official
submission remain separate unfinished delivery steps.