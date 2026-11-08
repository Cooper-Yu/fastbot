# CP21 FastBot 阶段源码

本仓库保存 `serial_motor` 与 `serial_motor_msgs` 的当前学习阶段实现。目标环境为 Raspberry Pi 4、Ubuntu 22.04 ARM64、ROS 2 Humble。

## 当前已验证范围

- 差速目标换算、按比例限幅、串口完整写入与应答检查。
- 编码器累计计数、int32 回绕差、实际时间间隔与左右角速度反馈。
- 有效指令超时零目标、故障抑制、启动查询及退出关闭。
- 三种自定义消息的生成、导入与 ROS 发布。
- 实物静止/手转反馈、单次短脉冲、约 2 秒和 10 秒的有界周期目标测试。

10 秒观察中右轮前段有停顿、后段连续；原因和独立速度精度仍未确定。这里的阶段验证不代表官方评分完成。

## 构建与预览入口

```bash
source /opt/ros/humble/setup.bash
cd ~/ros2_ws
colcon build --packages-select serial_motor_msgs serial_motor
source install/setup.bash
ros2 run serial_motor motor_driver --ros-args \
  -p wheel_radius:=0.032 -p wheel_separation:=0.12 \
  -p encoder_cpr:=2537 -p loop_rate:=30
```

默认 `serial_enabled=false`，上述入口只预览控制计算，不打开设备，也不产生真实编码器反馈。真实串口必须显式配置，运动验证使用训练记录中的已审阅有界脚本。

| 目录 | 职责 |
| --- | --- |
| `serial_motor` | ROS 速度输入、串口目标和左右计数/轮速反馈 |
| `serial_motor_msgs` | 三个消息接口与生成配置 |

## 当前待办

- 整车前进/转动速度纯函数已验证，尚未接节点；下一学习片段为根据实际运动累计位置与朝向。
- 课程 5.4 的后续整车运动/位置朝向累计、相应输出与 launch 配置。
- 后续遥操作时复查启动不平顺；雷达、相机和整体 bringup 仍待。
- 许可证声明与正式交付仓库/下载说明仍需完善。

详细学习、脚本和运行证据以 WSL `training_notes/checkpoint21/fastbot_setup/` 为权威记录；本仓库不复制完整学习历史。工程说明用中文，源码注释用英文；完成验证的教学 TODO 从源码移除，实际待办保留。


## Delegated integration (2026-11-08)
See [build, interfaces, safety and acceptance](docs/INTEGRATION.md). Software and physical acceptance are recorded separately.
