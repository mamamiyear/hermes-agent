# 运行与开发

## 快速运行（用户视角）

项目 README 给出了推荐路径（见 [README.md](file:///workspace/README.md#L30-L63)）：

- 安装后启动 CLI：
  - `hermes`
  - `hermes model`（选择 provider/model）
- 启动消息网关：
  - `hermes gateway`

## 源码运行（开发视角）

仓库内常见入口：

- CLI 入口：[hermes_cli/main.py](file:///workspace/hermes_cli/main.py)
- Agent runner（独立运行）：[run_agent.py](file:///workspace/run_agent.py)
- Gateway runner（平台消息接入）：[gateway/run.py](file:///workspace/gateway/run.py)

典型开发安装（README 中给了 uv 路径，见 [README.md#L144-L161](file:///workspace/README.md#L144-L161)）：

```bash
uv venv venv --python 3.11
source venv/bin/activate
uv pip install -e ".[all,dev]"
scripts/run_tests.sh
```

## 配置

- `~/.hermes/.env`：环境变量（如 FEISHU_APP_ID 等）
- `~/.hermes/config.yaml`：网关、模型、工具、显示、session_reset 等配置

Gateway 平台配置的结构由 [PlatformConfig](file:///workspace/gateway/config.py#L246-L289) 定义：

```yaml
platforms:
  feishu:
    enabled: true
    extra:
      app_id: "cli_xxx"
      connection_mode: "websocket"
```

## 测试

项目使用 pytest（见 [pyproject.toml](file:///workspace/pyproject.toml#L146-L152)）。

注意：仓库默认 addopts 包含 `-n auto`（xdist）。在缺少 xdist 的环境可用下面方式临时覆盖：

```bash
pytest -o addopts='' tests/...
```

