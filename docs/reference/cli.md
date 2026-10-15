# CLI 参考

以下命令在 SDK 仓库根目录执行。帮助不会连接硬件；读取与硬件命令仍需真实 CAN。


基本格式：

```bash
uv run agilex-sdk-python COMMAND [OPTIONS]
```

成功结果以 JSON 输出，返回 0。预期的连接、反馈或硬件操作错误返回 1；读取结果为 `None` 时返回 2，参数解析/校验错误也由 argparse 返回 2。`status` 的 JSON 仍可能表示通信不完整，必须检查字段，不能只看退出码。

## 参数

| 参数                        | 含义                        | 默认值          | 单位/范围                          |
| ------------------------- | ------------------------- | ------------ | ------------------------------ |
| `command`                 | 要执行的命令                    | 无，必填         | 见下方命令表                  |
| `--channel`               | SocketCAN 通道              | `can0`       | 有效 CAN 接口名                     |
| `--interface`             | python-can 后端             | `socketcan`  | 后端名称                           |
| `--firmware`              | 主控制器固件协议族                 | `default`    | `default/v183/v188/v189`       |
| `--wait`                  | 连接后等待首批周期反馈的时间            | `0.2`        | 秒，≥ 0                          |
| `--timeout`               | 查询或标定反馈的最长等待时间            | `1.0`        | 秒，≥ 0                          |
| `--speed-percent`         | 规划运动速度百分比                 | `20`         | `%`，整数 1～100                   |
| `--joint`                 | 指定单个关节                    | 全部或未指定       | J1～J6，对应整数 1～6                 |
| `--delta-deg`             | 单关节相对转动量                  | 无，使用时必填      | 度，非 0，范围 −90～90                |
| `--joints`                | J1～J6 的绝对目标角              | 无，使用时必填      | 6 个有限数，单位 rad                  |
| `--pose`                  | `X Y Z Roll Pitch Yaw` 位姿 | 无，使用时必填      | XYZ：m；RPY：rad                  |
| `--mid`                   | 圆弧途经点位姿                   | 无，使用时必填      | 6 个数，单位同 `--pose`              |
| `--end`                   | 圆弧终点位姿                    | 无，使用时必填      | 6 个数，单位同 `--pose`              |
| `--rating`                | 碰撞保护或助力等级                 | 省略时只查询       | `protect`：0～8<br>`assist`：0～10 |
| `--width`                 | AGX 夹爪目标开口宽度              | 无，使用时必填      | m，非负有限数                        |
| `--force`                 | AGX 夹爪目标夹持力               | `1.0`        | N，非负有限数                        |
| `--payload`               | 末端负载档位                    | `empty`      | `empty/half/full`              |
| `--position`              | 底座安装方向                    | `horizontal` | `horizontal/left/right`        |
| `--confirm-hardware-test` | 旧版脚本兼容开关                  | `False`      | 无实际控制作用                        |

参数可以写在命令前后，但推荐统一写在命令后。以下“完整示例”会显式写出该命令真正使用的全部参数；没有可调参数的命令只保留一个推荐示例。

## 读取命令

Read command 只读取反馈或查询配置，不使能机械臂、不修改控制器配置，也不发送运动命令。其中 `status`、`joints` 和 `firmware` 使用轻量只读连接，其余命令通过 `PiperXDriver` 完成更完整的反馈校验。

#### 常用指令

| 命令       | 功能                             | 最简示例               | 完整示例                                                                                                |
| -------- | ------------------------------ | ------------------ | --------------------------------------------------------------------------------------------------- |
| `status` | 汇总连接、通信、FPS、固件、六轴角度、末端位姿和控制器状态 | `uv run agilex-sdk-python status` | `uv run agilex-sdk-python status --channel can0 --interface socketcan --firmware default --wait 0.2 --timeout 1.0` |
| `joints` | 读取 J1～J6 当前反馈角度，单位 rad         | `uv run agilex-sdk-python joints` | `uv run agilex-sdk-python joints --channel can0 --interface socketcan --firmware default --wait 0.2`               |
| `fps`    | 查看 SDK 接收 CAN 周期反馈的频率，单位 Hz    | `uv run agilex-sdk-python fps`    | `uv run agilex-sdk-python fps --channel can0 --interface socketcan --firmware default --wait 0.2`                  |

#### 不常用指令

| 命令                | 功能                            | 最简示例                      | 完整示例                                                                                                  |
| ----------------- | ----------------------------- | ------------------------- | ----------------------------------------------------------------------------------------------------- |
| `firmware` / `fw` | 查询主控制器固件信息                    | `uv run agilex-sdk-python firmware`      | `uv run agilex-sdk-python firmware --channel can0 --interface socketcan --firmware default --wait 0.2 --timeout 1.0` |
| `driver-status`   | 读取经过通信、格式、时效和错误检查的完整状态        | `uv run agilex-sdk-python driver-status` | `uv run agilex-sdk-python driver-status --channel can0 --interface socketcan --firmware default --wait 0.2`          |
| `motor-status`    | 读取六轴整体状态，以及逐电机的位置、电流、温度、电压和故障 | `uv run agilex-sdk-python motor-status`  | `uv run agilex-sdk-python motor-status --channel can0 --interface socketcan --firmware default --wait 0.2`           |
| `grip-status`     | 读取 AGX 夹爪开口、力、模式、使能、回零和错误状态   | `uv run agilex-sdk-python grip-status`   | `uv run agilex-sdk-python grip-status --channel can0 --interface socketcan --firmware default --wait 0.2`            |
| `pose`            | 读取法兰末端六维位姿和六轴角度               | `uv run agilex-sdk-python pose`          | `uv run agilex-sdk-python pose --channel can0 --interface socketcan --firmware default --wait 0.2`                   |
| `limits`          | 查询每个关节及末端当前运动限制               | `uv run agilex-sdk-python limits`        | `uv run agilex-sdk-python limits --channel can0 --interface socketcan --firmware default --wait 0.2 --timeout 1.0`   |
| `ratings`         | 查询各关节的碰撞保护和助力等级               | `uv run agilex-sdk-python ratings`       | `uv run agilex-sdk-python ratings --channel can0 --interface socketcan --firmware default --wait 0.2 --timeout 1.0`  |

## 硬件命令

Hardware command 使用带反馈校验和安全限制的 `PiperXDriver`。部分命令会使能机械臂、修改固件配置或产生真实运动，执行前必须确认机械臂周围没有人员和障碍物。

表格中的“SDK”表示该 `agilex-sdk-python` 命令最终调用的底层 pyAgxArm 运动接口；用户执行时必须使用第一行显示的 `agilex-sdk-python` 命令名。

#### 常用指令

| `agilex-sdk-python` 命令 / SDK 方法  | 功能                                 | 最简示例                                                                                                   | 完整示例                                                                                                                                                                                         |
| ----------------- | ---------------------------------- | ------------------------------------------------------------------------------------------------------ | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `enable` / `on`   | 使能 J1～J6，使机械臂可以接收运动命令；不会额外发送位置目标   | `uv run agilex-sdk-python enable`                                                                                     | `uv run agilex-sdk-python enable --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                                                     |
| `disable` / `off` | 失能 J1～J6；机械臂可能因重力下坠                | `uv run agilex-sdk-python disable`                                                                                    | `uv run agilex-sdk-python disable --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                       |
| `zero`            | 将 J1～J6 移动到当前坐标系的 `0 rad`；不会重新标定零点 | `uv run agilex-sdk-python zero`                                                                                       | `uv run agilex-sdk-python zero --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                                                       |
| `stop`            | 发送带阻尼电子急停并撤销驱动的已使能记录；不是物理断电        | `uv run agilex-sdk-python stop`                                                                                       | `uv run agilex-sdk-python stop --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                          |
| `reset`           | 恢复控制器运动状态；不是断电重启、恢复出厂或机械臂回零        | `uv run agilex-sdk-python reset`                                                                                      | `uv run agilex-sdk-python reset --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                         |
| `grip`            | 将 AGX 夹爪移动到指定开口宽度并设置夹持力            | `uv run agilex-sdk-python grip --width 0.035`                                                                         | `uv run agilex-sdk-python grip --width 0.035 --force 1.0 --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                |
| `grip-zero`       | 把 AGX 夹爪当前位置标定为零点，并等待夹爪确认          | `uv run agilex-sdk-python grip-zero`                                                                                  | `uv run agilex-sdk-python grip-zero --timeout 1.0 --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                       |
| `grip-reset`      | 复位 AGX 夹爪控制器；不会复位六轴或标定夹爪零点         | `uv run agilex-sdk-python grip-reset`                                                                                 | `uv run agilex-sdk-python grip-reset --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                    |
| `movej`           | 以普通关节模式移动到六轴绝对目标，并逐段等待到位           | `uv run agilex-sdk-python movej --joints 0 0 0 0 0 0`                                                                 | `uv run agilex-sdk-python movej --joints 0 0 0 0 0 0 --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                                 |
| `movejs`          | 以高响应、无平滑规划模式持续刷新六轴绝对目标直到到位，可能产生冲击  | `uv run agilex-sdk-python movejs --joints 0 0 0 0 0 0`                                                                | `uv run agilex-sdk-python movejs --joints 0 0 0 0 0 0 --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                                |
| `movep`           | 使用 P 模式移动到指定末端位姿                   | `uv run agilex-sdk-python movep --pose 0.20 0 0.30 0 1.57 0`                                                          | `uv run agilex-sdk-python movep --pose 0.20 0 0.30 0 1.57 0 --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                          |
| `movel`           | 使用直线 L 模式移动到指定末端位姿                 | `uv run agilex-sdk-python movel --pose 0.20 0 0.30 0 1.57 0`                                                          | `uv run agilex-sdk-python movel --pose 0.20 0 0.30 0 1.57 0 --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                          |
| `movec`           | 按起点、途经点和终点执行末端圆弧运动                 | `uv run agilex-sdk-python movec --pose 0.20 0 0.30 0 1.57 0 --mid 0.20 0.05 0.35 0 1.57 0 --end 0.20 0 0.40 0 1.57 0` | `uv run agilex-sdk-python movec --pose 0.20 0 0.30 0 1.57 0 --mid 0.20 0.05 0.35 0 1.57 0 --end 0.20 0 0.40 0 1.57 0 --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20` |

> **当前未实现：** pyAgxArm 的单关节 MIT 控制方法 `move_mit` 尚未封装到
> `PiperXDriver` 和 `agilex-sdk-python` CLI，因此目前不能执行 `uv run agilex-sdk-python move_mit`。如需将它
> 作为常用指令，必须先补充驱动方法、CLI 参数与安全校验。

#### 不常用指令

| 命令           | 功能                                              | 最简示例                                           | 完整示例                                                                                                                                                                                                                   |
| ------------ | ----------------------------------------------- | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `hold`       | 使能后把当前实测六轴位置设为目标，保持当前位置                         | `uv run agilex-sdk-python hold`                               | `uv run agilex-sdk-python hold --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                                                                                 |
| `move-joint` | 让指定关节相对当前位置转动一次，其余关节保持当前目标                      | `uv run agilex-sdk-python move-joint --joint 1 --delta-deg 5` | `uv run agilex-sdk-python move-joint --joint 1 --delta-deg 5 --channel can0 --interface socketcan --firmware default --wait 0.2 --speed-percent 20`                                                                                   |
| `clear`      | 清除指定关节错误；省略 `--joint` 时清除全部关节错误                 | `uv run agilex-sdk-python clear`                              | `uv run agilex-sdk-python clear --joint 1 --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                                         |
| `max-limits` | 将 J1～J6 限值写为 `3.0 rad/s` 和 `5.0 rad/s²`，并要求固件确认 | `uv run agilex-sdk-python max-limits`                         | `uv run agilex-sdk-python max-limits --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                                              |
| `defaults`   | 只恢复关节角度、速度和加速度默认限制，不恢复全部出厂参数                    | `uv run agilex-sdk-python defaults`                           | `uv run agilex-sdk-python defaults --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                                                |
| `speed`      | 单独写入后续规划运动使用的速度百分比                              | `uv run agilex-sdk-python speed`                              | `uv run agilex-sdk-python speed --speed-percent 20 --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                                |
| `payload`    | 设置控制器使用的末端负载档位                                  | `uv run agilex-sdk-python payload`                            | `uv run agilex-sdk-python payload --payload full --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                                  |
| `install`    | 设置机械臂底座的实际安装方向                                  | `uv run agilex-sdk-python install`                            | `uv run agilex-sdk-python install --position right --channel can0 --interface socketcan --firmware default --wait 0.2`                                                                                                                |
| `protect`    | 不带 `--rating` 时查询等级；带参数时设置单轴或全部关节碰撞保护等级         | `uv run agilex-sdk-python protect`                            | 查询：`uv run agilex-sdk-python protect --channel can0 --interface socketcan --firmware default --wait 0.2 --timeout 1.0` 设置：`uv run agilex-sdk-python protect --joint 1 --rating 8 --channel can0 --interface socketcan --firmware default --wait 0.2` |
| `assist`     | 不带 `--rating` 时查询等级；带参数时设置单轴或全部关节助力等级           | `uv run agilex-sdk-python assist`                             | 查询：`uv run agilex-sdk-python assist --channel can0 --interface socketcan --firmware default --wait 0.2 --timeout 1.0` 设置：`uv run agilex-sdk-python assist --joint 1 --rating 10 --channel can0 --interface socketcan --firmware default --wait 0.2`  |

`zero`、`movej` 和 `movejs` 会按最大关节角变化自动拆分路径点，每个路径点最多等待 10 秒，并要求目标误差不超过 `0.1°`。命令结束后只断开主机连接，不会自动失能机械臂。

## ZMQ 服务

```bash
uv run agilex-sdk-python-server
uv run agilex-sdk-python-server --help
```

`ag-gello-server` 是服务别名；`ag` 是主 CLI 别名。服务参数和默认值见[配置参考](config.md#服务参数)，协议与手动 controller 联调见[架构文档](../index.md)。服务启动会使能并默认写入运动限制，无交互确认。它不提供 `--speed-percent`；`--confirm-hardware-control` 仅为兼容开关。
