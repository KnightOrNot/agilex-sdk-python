# 配置参考

SDK 没有 TOML 配置文件。CLI 参数、构造器参数和 CAN 脚本分别配置各自层级；controller TOML 由应用解析后传给 SDK 子进程，不会被 SDK 自动读取。

## CAN 与主 CLI

```bash
./scripts/config_can.sh can1 1000000 100
uv run agilex-sdk-python status --channel can1
```

脚本位置参数为接口、bitrate、txqueuelen，默认 `can0`、`1000000`、`100`。接口必须存在；bitrate 校验为正整数，txqueuelen 由 `ip` 校验。主 CLI 默认 `--channel can0`、`--interface socketcan`、`--firmware default`、`--wait 0.2`、`--timeout 1.0`；完整表见 [CLI](cli.md)。

## 服务参数

| 参数 | 默认值 / 含义 |
| --- | --- |
| `--channel` / `--interface` / `--firmware` | `can0` / `socketcan` / `default` |
| `--host` / `--port` | `127.0.0.1` / `6001`，ZMQ 监听地址 |
| `--hz` | `50`，JS 命令最大发送频率，必须为正有限数 |
| `--gripper-max-width-m` | `0.1`，全开宽度，正有限数，m |
| `--gripper-force-n` | `1.0`，夹持力，非负有限数，N |
| `--configure-motion-limits` | 默认开启；写入并确认关节速度/加速度限制 |
| `--no-configure-motion-limits` | 跳过限制写入，保留固件现有设置 |
| `--confirm-hardware-control` | 兼容旧脚本，无交互确认作用 |

默认限制写入值为 `3.0 rad/s`、`5.0 rad/s²`，可能持久改变固件配置。固件无法确认时默认启动失败；只有确认现有配置适用后才使用跳过选项。

服务将第七维裁剪到 `[0, 1]` 并按最大宽度正向映射。主手的开闭端点、校准与六轴方向由应用处理，不能据服务约定推定 GELLO 物理方向。

## Python 驱动默认值

| `PiperXDriver` 参数 | 默认值 |
| --- | --- |
| `feedback_timeout` | `0.5 s` |
| `motor_confirmation_timeout` / `motor_confirmation_interval` | `2.0 s` / `0.01 s` |
| `max_command_delta_rad` | `0.2 rad` |
| `joint_limit_margin_rad` | `5°`，以 rad 传入 |

主 CLI 构建驱动时将单步跨度设为 `90°`；七维服务适配器默认使用 `1.0 rad`。三者不是同一配置层，应用不得把较大的调试跨度当作连续跟随的默认值。
