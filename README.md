# FastBot

Checkpoint 21 的真实机器人驱动。目标环境：Raspberry Pi 4、Ubuntu 22.04 ARM64、ROS 2 Humble；上位控制器为 Arduino Nano，雷达为 N10，相机为 IMX219。

## 下载、依赖与构建

在新的 Linux 工作空间中执行；已有同名包时不要再克隆一份。

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

依赖脚本需要 sudo，仅安装软件；不修改相机启动配置、不重启、不启动电机。首次使用 rosdep 的主机需要先完成其初始化。相机启动配置及硬件差异见 [集成说明](docs/INTEGRATION.md)。

## 启动

先检查不连接硬件的预览模式：

```bash
ros2 launch fastbot_bringup bringup.launch.xml \
  serial_enabled:=false lidar_enabled:=false camera_enabled:=false
```

实物接线、供电和参数确认后，课程统一入口为：

```bash
ros2 launch fastbot_bringup bringup.launch.xml
```

该入口默认打开真实设备；不收到有效速度指令时不发送非零目标。首次运动应固定车架、两轮悬空，并使用有界低速测试。没有配置开机自动启动。

## 包与接口

| 路径 | 职责 |
| --- | --- |
| `serial_motor` | Twist 转换、串口目标、编码器反馈、里程计、TF 与轮关节状态 |
| `serial_motor_msgs` | 自定义电机指令与反馈消息 |
| `fastbot_bringup` | `bringup.launch.xml`、设备配置、诊断与测试工具 |
| `fastbot_description` | 简化机器人模型；传感器安装位置尚非精确标定 |
| `third_party/lslidar` | 固定上游版本的 N10 源码及退出修复补丁 |

主要话题：`/fastbot/cmd_vel`（Twist）、`/fastbot/scan`（LaserScan）、`/fastbot_camera/image_raw`（Image）、`/fastbot/odom`（Odometry）、`/fastbot/joint_states`（JointState）。

空串口参数会优先使用课程 udev 别名，再匹配唯一已知 USB 设备；缺失或歧义时拒绝猜测。可显式覆盖 `serial_port`、`lidar_port` 与 `camera_device`。本车轮半径 0.032 m、轮距 0.12 m、CPR 2537；其他机器人必须重新核对。

## 验证与边界

已完成六包构建、自有包测试、独立克隆构建、真实相机/雷达、本机及 rosject 的单次有界键盘遥控和停车检查。测试结束后设备释放。详见 [实际证据及复现方式](docs/INTEGRATION.md)。

这些证据不代表官方评分、地面导航精度或左右轮跟踪精度已经通过。短测试两轮增量不同，旧右轮启动停顿仍没有确定根因。发布仓库不自动启动任何硬件。

## 许可

自有代码使用 [Apache-2.0](LICENSE)。第三方雷达驱动保留其 GPL 许可，模板附带资产保留各自许可；范围与来源见 [NOTICE](NOTICE)。
