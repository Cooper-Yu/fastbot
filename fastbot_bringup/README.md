# fastbot_bringup

> Section/Checkpoint：CP21-DELEGATED-20261108 · 环境：humble · 模板：1.1.1
>
> 当前：工具骨架已准备，核心功能与行为验证尚未完成。

| 要查什么 | 入口 |
| --- | --- |
| 学习者当前动作与真实进度 | TODO：链接已有课程 code_lab/plan.md、attempt.md、verification.md |
| 包的职责、接口与运行约定 | 本页以下内容 |
| 数据/控制流程 | [flow.md](docs/flow.md) |
| 代码规范与工具入口 | [tooling.md](docs/tooling.md) |
| 统一排版的效果与规则说明 | [style_profile.md](docs/style_profile.md) |
| 类/方法文档 | [Doxygen 源说明](docs/api.dox)；生成后打开 docs/generated/html/index.html |
| 测试用例与接线 | [test/README.md](test/README.md) |
| 调试、构建与测试任务 | .vscode/launch.json、.vscode/tasks.json |
| 包的工程关系总览 | [engineering.canvas](docs/engineering.canvas) |
| 模板来源和许可 | [TEMPLATE_SOURCES.md](docs/TEMPLATE_SOURCES.md) |

## 职责与边界

TODO：由学习者说明本包负责什么、哪些职责属于其他包。

## 输入 输出与状态

| 接口 | 类型/单位 | 前置条件 | 输出或副作用 |
| --- | --- | --- | --- |
| TODO | TODO | TODO | TODO |

## 运行入口

TODO：实现入口确认后记录真实命令、参数、ROS 接口和停止条件。
模板不会自动启动节点、创建控制逻辑或初始化 Git 仓库。

## 本轮实现与验证

核心 TODO、学习者尝试和测试预期保存在对应课程权威记录中，不在本页重复完整作答。
每个有意义片段：保存排版 → 查看规范诊断 → 审阅 diff → 调试/测试行为 → 记录证据。
需要云端/官方/真机的 gate 分开记录，生成模板不等于课程 PASS。

## 文档生成

从本包根目录运行：

```bash
doxygen docs/Doxyfile
```

Doxygen 页使用固定版本的 Doxygen Awesome 样式。
修改源码注释后重新运行生成命令。
Markdown 中的 Mermaid 由支持 Mermaid 的预览器显示；Doxygen 的关联页显示对应源码/说明，不默认把 Mermaid 变成运行追踪。


## Current implementation
Entry: ros2 launch fastbot_bringup bringup.launch.xml. Default opens physical devices; use serial_enabled:=false lidar_enabled:=false camera_enabled:=false for preview. See ../docs/INTEGRATION.md.
