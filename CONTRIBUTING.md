# 贡献指南

## 开发环境与边界

推荐 Python 3.11，使用 `.python-version`、`pyproject.toml` 和 `uv.lock` 准备独立环境：

```bash
uv sync --frozen
uv run agilex-sdk-python --help
```

解释器和系统依赖见[安装指南](docs/installation.md)。SDK 负责 CAN、反馈、安全驱动、运动/夹爪与 ZMQ 服务。主手校准、主从方向映射、应用进程管理和录制属于 controller；数据对齐和 LeRobot 属于 converter。保持 SDK 可被其他应用独立使用。

## 模块与修改原则

| 模块 | 责任 |
| --- | --- |
| `reader.py` | 轻量只读连接、固件、关节与综合快照 |
| `driver.py` | 反馈时效、错误/使能检查、关节限制、运动与夹爪 |
| `gello_follower.py` | 六轴加夹爪的七维协议、JS 会话与限频 |
| `gello_server.py` | ZMQ REP 分派与进程信号 |
| `cli.py` | 参数校验、命令分派、JSON 与退出码 |
| `exceptions.py` | 可区分的通信、反馈、状态与命令异常 |

Python 使用四空格、`snake_case` 函数/模块和 `PascalCase` 类；公开接口添加类型标注。使用 Ruff 默认检查和 Pyright；没有独立 Ruff 配置、格式化配置或覆盖率门槛。`--isolated` 避免作为 submodule 时继承父仓库规则，结果与独立 checkout 保持一致。

保持读取命令无使能和运动副作用。新增运动路径必须保留反馈、关节限位、使能和单步跨度检查；写入配置后按现有协议确认。流式 JS 模式在会话中只设置一次。连接关闭不等于失能或回零，不应在清理阶段隐式增加这些动作。使用异常和有界等待报告失败，不能用缺失或陈旧反馈继续运动。

## 测试与验证

```bash
uv run --frozen pytest
uv run --frozen ruff check --isolated .
uv run --frozen pyright
bash -n scripts/config_can.sh
uv build
```

`tests/test_*.py` 使用 fake SDK、时间和反馈；覆盖命令分派、只读生命周期、运动保护、JS 限频、夹爪映射和清理。为行为变更增加针对性回归，避免测试只复述实现。pytest-cov 已安装，可按需运行覆盖率报告；没有最低覆盖率要求。

默认检查不接触真实硬件。真实 CAN/运动验证需操作员确认，并在 PR 中记录设备、固件、前置条件、步骤和结果。不要把模拟回归写成机械臂验收。

## 提交与多仓库协作

历史主题包含 `refactor:` 与 `update docs.`；新增提交优先采用 `feat:`、`fix:`、`docs:` 等前缀和简洁祈使句。PR 写明问题与最终行为、参数/协议兼容性、检查结果、硬件状态及相关 issue。

本仓库既可独立使用，也可作为 controller submodule；不同 checkout 不会自动同步。先提交、验证并发布子项目，再更新 controller 的 gitlink。开发期间保留本地改动，避免 setup 重置子模块。

参数或行为变更同步更新 README、`docs/reference/`、CHANGELOG 与测试。不要提交 `.venv`、缓存、构建产物、采集数据或个人设备凭据。许可证尚未选定，保留第三方来源与声明，不推定项目已获得依赖的同种许可。
