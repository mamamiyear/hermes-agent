# 模块与职责

## 顶层目录

- [agent/](file:///workspace/agent)：Agent 内部组件（prompt 构建、记忆、上下文压缩、provider 适配、工具守卫等）。
- [gateway/](file:///workspace/gateway)：消息网关（会话、平台适配器、消息路由、流式输出、跨平台投递）。
- [tools/](file:///workspace/tools)：工具实现与注册（终端、浏览器、文件、MCP、记忆、发送消息、cron 等）。
- [hermes_cli/](file:///workspace/hermes_cli)：CLI/TUI 入口、配置、安装、诊断、网关控制等。
- [cron/](file:///workspace/cron)：内置定时任务调度（croniter + delivery 到 gateway 平台）。
- [plugins/](file:///workspace/plugins)：插件与可选能力（memory、spotify、google_meet 等）。
- [environments/](file:///workspace/environments)：RL/基准/工具调用解析器等研究与训练相关环境。
- [web/](file:///workspace/web)、[ui-tui/](file:///workspace/ui-tui)、[website/](file:///workspace/website)：前端与文档站构建（非运行时核心，但影响发布与用户界面）。

## gateway 细分

- 平台适配器：[gateway/platforms/](file:///workspace/gateway/platforms)
  - 统一基类与归一化事件模型：[base.py](file:///workspace/gateway/platforms/base.py)
  - Feishu/Lark：[feishu.py](file:///workspace/gateway/platforms/feishu.py)
  - Telegram：[telegram.py](file:///workspace/gateway/platforms/telegram.py)
  - Slack：[slack.py](file:///workspace/gateway/platforms/slack.py)
  - Discord：[discord.py](file:///workspace/gateway/platforms/discord.py)
- 会话管理：[gateway/session.py](file:///workspace/gateway/session.py)
- 配置模型：[gateway/config.py](file:///workspace/gateway/config.py)
- 主循环与调度：[gateway/run.py](file:///workspace/gateway/run.py)
- 流式输出消费者：[gateway/stream_consumer.py](file:///workspace/gateway/stream_consumer.py)

## agent 细分（常用阅读入口）

- Prompt 组织与系统提示：`build_*_prompt` 在 [prompt_builder.py](file:///workspace/agent/prompt_builder.py)
- 上下文压缩：[context_compressor.py](file:///workspace/agent/context_compressor.py)
- 记忆管理：[memory_manager.py](file:///workspace/agent/memory_manager.py)
- 工具调用守卫（安全/预算/路径）：[tool_guardrails.py](file:///workspace/agent/tool_guardrails.py)
- Provider 适配（Anthropic/OpenAI/Bedrock/Gemini…）：[agent/transports/](file:///workspace/agent/transports)

## tools 细分

- 注册表：工具自注册入口 [tools/registry.py](file:///workspace/tools/registry.py)
- 工具编排与 OpenAI tool schema 适配：[model_tools.py](file:///workspace/model_tools.py)
- 终端执行：[terminal_tool.py](file:///workspace/tools/terminal_tool.py)
- 文件操作：[file_operations.py](file:///workspace/tools/file_operations.py)
- 浏览器与网页：[browser_tool.py](file:///workspace/tools/browser_tool.py)、[web_tools.py](file:///workspace/tools/web_tools.py)
- MCP：OAuth 与工具桥接 [mcp_tool.py](file:///workspace/tools/mcp_tool.py)

