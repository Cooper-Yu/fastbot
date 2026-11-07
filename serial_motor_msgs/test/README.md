# 测试入口与用例设计

此目录用于 serial_motor_msgs 的真实测试。
本模板不添加一个永远通过的占位测试，也不替学习者填写核心行为或预期。

| 用例字段 | 由学习者/当前任务补齐 |
| --- | --- |
| 稳定 id 与行为约定 | TODO |
| 输入、初始状态、前置条件 | TODO |
| 预期输出/副作用/容差 | TODO |
| 正常、边界或状态变化理由 | TODO |
| 目标环境与验证命令 | TODO |

C++ 使用 GoogleTest/ament_cmake_gtest 等实际项目框架；Python 使用 pytest。
ROS 集成测试按需要使用 launch_testing，具备就绪条件、有界等待和清理。
先完成一个功能片段的有效比较，再补相关边界和状态路径。
测试构建/安装/依赖接线属于课程要求时由学习者实现；Coach 可准备外围支撑并审阅。

原生 ros2 pkg create 可能已生成 lint 测试，保留它们。
Ruff 与 ROS/ament 的全部检查不是完全同一个规范；发现冲突时明确核对实际项目要求，不全局关闭原有检查。

权威尝试与证据：TODO：链接已有课程 code_lab/plan.md、attempt.md、verification.md。
