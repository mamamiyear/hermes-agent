# 依赖与构建

## Python 依赖（核心）

项目以 Python 为主，依赖在 [pyproject.toml](file:///workspace/pyproject.toml) 中声明：

- 核心：openai、anthropic、httpx、requests、pydantic、tenacity、pyyaml、python-dotenv、rich、jinja2、prompt_toolkit 等
- 可选 extra：
  - `messaging`：telegram/discord/slack 等平台 SDK
  - `feishu`：`lark-oapi`
  - `mcp`：`mcp`
  - `voice` / `tts-premium`：语音能力相关
  - `web`：fastapi/uvicorn（dashboard）

建议阅读：

- 入口脚本与可执行命令：[pyproject.toml#L132-L145](file:///workspace/pyproject.toml#L132-L145)

## Node/前端依赖

- [web/](file:///workspace/web)：Vite 前端（dashboard），由 Python 包打包分发
- [ui-tui/](file:///workspace/ui-tui)：终端 UI 相关构建资产
- [website/](file:///workspace/website)：Docusaurus 文档站

这部分通常不影响“网关 + agent”核心运行，但影响发行包与文档站构建。

## 外部系统依赖（按功能）

- 终端/沙盒：部分 terminal backend 需要 Docker / SSH / Daytona / Modal 等
- 浏览器：browser 工具可能依赖 playwright 或 CDP 相关组件（取决于配置与工具链）
- 平台：Telegram/Slack/Feishu 等需要各自 bot/app 凭据与回调配置

