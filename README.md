# agilex-sdk-python

面向 AgileX PiPER-X 的 Python 硬件适配层，基于 pyAgxArm 与 SocketCAN，提供状态读取、运动/夹爪控制和七自由度 ZMQ Robot 服务。推荐 Python 3.11；包导入名为 `agilex_sdk_python`。

GELLO 主手、应用配置、客户端组合、跟随与 raw 录制由 agilex-controller 管理；时间对齐与数据转换由 lerobot-converter 管理。SDK 可以独立调试机械臂，不依赖这两个应用。

## 职责与目录

```text
src/agilex_sdk_python/    只读连接、安全驱动、ZMQ 服务与 CLI
scripts/config_can.sh    SocketCAN 配置工具
tests/                   模拟 SDK 的无硬件回归
docs/                    架构、安装、命令、配置与排障
```

## 安装与只读检查

准备 Linux、Python 3.11、Git、uv 和 `iproute2`，在本仓库根目录执行：

```bash
uv sync --frozen
uv run agilex-sdk-python --help
```

连接设备后配置默认 CAN，并检查反馈：

```bash
./scripts/config_can.sh
uv run agilex-sdk-python status
uv run agilex-sdk-python joints
```

`status` 不使能或移动机械臂。确认 JSON 中 `connected`、`communication_ok` 为 `true`，并核对反馈；仅打开 CAN 接口不代表机械臂通信正常。完整安装及 pyenv 流程见[安装指南](docs/installation.md)。

## ZMQ 服务与 controller

以下服务会进入硬件控制流程：连接 CAN、默认写入并确认速度/加速度限制、使能机械臂。启动前固定底座、清空工作区并确认急停可用。

```bash
uv run agilex-sdk-python-server
```

默认监听 `127.0.0.1:6001`，JS 命令最大发送频率为 50 Hz。服务不创建 GELLO 客户端、不记录 raw，也不负责应用退出回零。PiPER-X 的日常跟随与采集在 controller 根目录使用：

```bash
./scripts/start_gello_follow.sh
./scripts/start_data_record.sh
```

controller 默认管理 SDK 服务的启停，使用它时无需手动启动第二个服务。手动服务联调与协议说明见[架构文档](docs/index.md)。

SDK 的 CLI 和服务默认没有交互式运动确认；兼容确认开关不增加保护。运动前阅读 [CLI 参考](docs/reference/cli.md)。关闭连接不会自动失能或回零；失能可能导致下坠，必须先支撑机械臂。

## 开发与文档

```bash
uv run --frozen pytest
uv run --frozen ruff check --isolated .
uv run --frozen pyright
uv build
```

测试注入模拟 SDK，不访问 CAN。真实设备验收需单独执行并记录。

- [架构与 ZMQ 协议](docs/index.md)
- [安装指南](docs/installation.md)
- [CLI 参考](docs/reference/cli.md)
- [配置参考](docs/reference/config.md)
- [故障排查](docs/reference/errors.md)
- [贡献指南](CONTRIBUTING.md)
- [版本记录](CHANGELOG.md)
- [许可状态](LICENSE)

本仓库尚未选定项目级许可证；依赖保留各自许可，不能将依赖的许可视为本项目的许可。
