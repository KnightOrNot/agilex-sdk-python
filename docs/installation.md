# 安装指南

## 系统与解释器

要求支持 SocketCAN 的 Linux、PiPER-X 和兼容 USB-CAN。推荐 Python 3.11，虽然包元数据允许更高版本。Ubuntu 系统工具：

```bash
sudo apt update
sudo apt install -y git curl iproute2
```

准备 uv 和 pyenv 后，在仓库根目录选择解释器并同步锁文件：

```bash
sdk_requested_version="$(<.python-version)"
sdk_resolved_version="$(pyenv latest -k "$sdk_requested_version")"
pyenv install -s "$sdk_resolved_version"
sdk_python="$(PYENV_VERSION="$sdk_resolved_version" pyenv which python)"
UV_NO_MANAGED_PYTHON=1 uv sync --frozen --python "$sdk_python"
```

已有 Python 3.11 环境时通常只需 `uv sync --frozen`。无需手动激活 `.venv`。pyAgxArm 从 `pyproject.toml` 指定的 Git 源安装，精确提交由 `uv.lock` 锁定，需要 Git 和 GitHub 网络连通。

## CAN 配置与只读验收

确认接线、终端电阻、设备电源和急停状态，并检查接口：

```bash
ip -brief link show can0
./scripts/config_can.sh
ip -details -statistics link show can0
uv run agilex-sdk-python status
uv run agilex-sdk-python joints
uv run agilex-sdk-python firmware
```

CAN 默认 `can0`、`1000000 bit/s`、发送队列 100。脚本已处于相同 UP/bitrate 时复用接口，否则通过 sudo 重配；不设置 Bus-Off 自动重启。自定义示例见[配置参考](reference/config.md)。

`status` 的连接与通信字段都应为 true。若没有完整反馈，先查 CAN RX、接线、电源和波特率，不要直接运行运动命令。硬件运动测试前固定底座、清空工作区并确认急停可用；失能前支撑机械臂。

## 与 controller 集成

作为 controller submodule 时，优先使用根项目的 `scripts/setup.sh` 准备环境。SDK 仍使用自己的 `.venv`，不需要安装 GELLO、controller 或 LeRobot。日常跟随由 controller 启动 SDK 子进程；独立服务见 [CLI 参考](reference/cli.md#zmq-服务)。
