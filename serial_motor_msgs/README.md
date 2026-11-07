# serial_motor_msgs

CP21 / Section21 5.3 消息接口包，ament_cmake + rosidl 生成；不运行电机或持有串口。

| 消息 | 字段 | 单位/用途 |
| --- | --- | --- |
| `EncoderVals` | int32 mot_1_enc_val / mot_2_enc_val | 左右累计计数 |
| `MotorVels` | float32 mot_1_rad_sec / mot_2_rad_sec | 左右角速度，rad/s |
| `MotorCommands` | bool is_pwm；float32 mot_1_req_rad_sec / mot_2_req_rad_sec | 命令接口定义；当前驱动速度输入使用 Twist |

已在目标 Pi/Humble 构建、显示接口并从 Python 导入；EncoderVals/MotorVels 的真实 ROS 发布在驱动阶段验证。接口存在本身不表示有节点运行。

## 构建与接口查看

```bash
cd ~/ros2_ws
colcon build --packages-select serial_motor_msgs
source install/setup.bash
ros2 interface show serial_motor_msgs/msg/EncoderVals
```

源文件为 msg/，CMakeLists.txt 列出三接口；package.xml 声明 rosidl 生成/运行依赖和接口包组。许可证仍为待确定项，未自行代选。

## 工程导航

- [流程](docs/flow.md) · [工具](docs/tooling.md) · [样式](docs/style_profile.md)
- [注释约定](docs/comment_checks.md) · [测试说明](test/README.md)
- [模板来源与第三方许可](docs/TEMPLATE_SOURCES.md)

学习者当前题目与验证证据留在 WSL `training_notes/checkpoint21/fastbot_setup/`，这里记录当前接口，不复制逐题历史。Doxygen 配置已提供，当前 Pi 尚未安装生成工具，未声称 HTML 已生成或验收。
