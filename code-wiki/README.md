# Hermes Agent Code Wiki

本目录是一套面向代码阅读与二次开发的 Wiki 文档，目标是帮助你在不通读全仓库的情况下，快速理解项目的核心架构、模块边界、关键类/函数、依赖关系与运行方式。

## 导航

- [项目概览与架构](file:///workspace/code-wiki/architecture.md)
- [模块与职责](file:///workspace/code-wiki/modules.md)
- [关键类与关键路径](file:///workspace/code-wiki/key-classes.md)
- [依赖与构建](file:///workspace/code-wiki/dependencies.md)
- [运行与开发](file:///workspace/code-wiki/running.md)
- [Feishu/Lark 会话通道（含自动 @ 改造）](file:///workspace/code-wiki/feishu-channel.md)

## 仓库入口速览

- CLI 入口：pyproject.toml#L132-L136 → [hermes_cli.main:main](file:///workspace/hermes_cli/main.py)
- Agent Runner：pyproject.toml#L132-L136 → [run_agent.py](file:///workspace/run_agent.py)
- Messaging Gateway：核心逻辑在 [gateway/run.py](file:///workspace/gateway/run.py)
- 平台适配器（Telegram/Slack/Feishu/…）：[gateway/platforms/](file:///workspace/gateway/platforms)
- 工具系统（Tools）：[tools/](file:///workspace/tools) + [tools/registry.py](file:///workspace/tools/registry.py)

