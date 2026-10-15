# Changelog

## Unreleased

- 按 controller 文档结构拆分架构、安装、CLI、配置和排障，新增贡献与 agent 指南。
- 将旧 GELLO 客户端示例更新为 controller 入口，明确 SDK 与应用的生命周期边界。
- 记录项目级许可尚未选定的状态；本次仅修改文档，不修改硬件实现。

## 0.1.0

以下汇总现有实现，不表示新增发布 tag。

- 统一包名 `agilex-sdk-python`、导入名 `agilex_sdk_python` 及两个 CLI，保留 `ag`、`ag-gello-server` 别名。
- 提供 SocketCAN 只读诊断、带反馈/限位检查的运动和 AGX 夹爪控制。
- 提供七自由度 ZMQ Robot 服务、JS 流式会话和命令限频。
- 使用 uv 管理环境，提供 pytest、Ruff 和 Pyright 开发检查。
