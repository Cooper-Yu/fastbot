# serial_motor

CP21 / Section21 5.4 当前驱动实现；环境为 Pi Ubuntu22.04 ARM64 / Humble。目标计算、协议、反馈及有界运行已有阶段证据，整课仍未完成。

## 职责与接口

| 接口 | 类型 | 含义 |
| --- | --- | --- |
| `/fastbot/cmd_vel` | `geometry_msgs/msg/Twist` | 输入 linear.x（m/s）和 angular.z（rad/s） |
| `/fastbot/encoder_vals` | `serial_motor_msgs/msg/EncoderVals` | 左右累计 int32 计数 |
| `/fastbot/motor_vels` | `serial_motor_msgs/msg/MotorVels` | 左右实际采样区间角速度（rad/s） |

该包将有效速度换成限幅后的整数 m 指令，通过串口要求完整写入/OK；周期 e 查询读取反馈。`serial_motor_msgs` 只定义接口，不控制设备。固件负责控制器侧调速，本包不替代固件内部控制。

## 运行与边界

入口为 `ros2 run serial_motor motor_driver`；构建及默认预览示例见[根 README](../README.md)。轮半径、轮距、CPR、控制器频率必须显式提供正有限值；源码零默认值用于拒绝未配置运行。

`serial_enabled` 默认 false。真实模式还需 serial_port、匹配的 baud_rate 与有限读写超时；启动等待后 b 查询确认就绪。只允许调用方有意启用硬件。默认 max_counts_per_loop=50 是初始软件限制，并非已标定安全速度；已审阅动力测试使用更低的15。

最近有效指令超过 command_timeout（默认0.5秒）后尝试零目标。通信故障会抑制后续命令并清计时，不声称已经物理停止，也不自动重试未确认命令。main 负责资源关闭；注入连接亦由它在退出时关闭。

反馈第一组只建立计数/时间起点，后续以实际 monotonic 间隔计算速度；成功后推进起点。回绕算法要求采样间真实变化小于2**31计数，不识别控制器重启。

## 当前阶段

活动 TODO86 的整车前进速度函数尚未实现/未调用；后续整车运动累计和 launch 继续由学习者逐片完成。此前生成代码骨架不等于当前所有工程工具均已验证。源码说明和协议已核，Pi 构建、ROS协议替身及有界实物记录分别保存在权威 verification.md。

## 工程导航

- [流程](docs/flow.md) · [工具](docs/tooling.md) · [样式](docs/style_profile.md)
- [注释约定](docs/comment_checks.md) · [测试说明](test/README.md)
- [模板来源与第三方许可](docs/TEMPLATE_SOURCES.md)

学习者当前题目与验证证据留在 WSL `training_notes/checkpoint21/fastbot_setup/`，这里记录当前接口，不复制逐题历史。Doxygen 配置已提供，当前 Pi 尚未安装生成工具，未声称 HTML 已生成或验收。
