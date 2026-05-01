# 关键类与关键路径

## Gateway 侧

### MessageEvent（归一化入站事件）

- 定义位置：[MessageEvent](file:///workspace/gateway/platforms/base.py#L869-L915)
- 核心字段：
  - `text` / `message_type`：正文与类型
  - `source: SessionSource`：来源（平台/会话/用户/线程）
  - `message_id`：原平台消息 ID（用于 reply）
  - `reply_to_message_id` / `reply_to_text`：被回复消息（用于上下文注入）
  - `media_urls` / `media_types`：附件的本地缓存路径与 MIME

### SessionSource（路由与会话维度）

- 定义位置：[SessionSource](file:///workspace/gateway/session.py#L71-L115)
- 关键用途：
  - `build_session_key()` 决定 session 粒度（群聊是否按用户隔离）
  - adapter.send 需要的 `chat_id` / `thread_id` 等路由字段

### BasePlatformAdapter（平台适配器基类）

- 定义位置：[BasePlatformAdapter](file:///workspace/gateway/platforms/base.py)
- 关键路径：
  - `handle_message()`：入站事件的 session guard、打断、队列等（见 [base.py](file:///workspace/gateway/platforms/base.py#L2505-L2620)）
  - `_process_message_background()`：后台跑 gateway handler 并统一做：
    - 发送 typing
    - 调用 gateway runner 的 message handler
    - 对 response 做媒体提取与拆分
    - 调用 `send()` 发回平台（见 [base.py](file:///workspace/gateway/platforms/base.py#L2640-L2912)）
  - `send(chat_id, content, reply_to, metadata)`：平台实现必须覆盖

### GatewayRunner（网关主调度器）

- 定义位置：[gateway/run.py](file:///workspace/gateway/run.py)
- 关注点：
  - `_prepare_inbound_message_text()`：把 reply_to_text 等上下文指针注入到用户消息（见 [run.py](file:///workspace/gateway/run.py#L5272-L5324)）
  - `_run_agent()`：创建/复用 AIAgent、处理流式输出、并发中断、失败恢复等（见 [run.py](file:///workspace/gateway/run.py)）

## Agent 侧

### AIAgent（工具调用循环核心）

- 定义位置：[AIAgent](file:///workspace/run_agent.py)
- 关键能力（高层）：
  - 组装系统提示（SOUL/skills/memory/上下文文件）
  - 触发模型调用（provider + transport）
  - 解析 tool_calls 并执行工具：`handle_function_call()`（见 [model_tools.py](file:///workspace/model_tools.py)）
  - 工具调用循环直到完成（final_response）
  - 上下文压缩与会话拆分（ContextCompressor）

### ToolRegistry（工具注册与可用性）

- 定义位置：[ToolRegistry](file:///workspace/tools/registry.py#L143-L173)
- 特点：
  - 工具文件模块级自注册：`registry.register(...)`
  - check_fn 结果 TTL 缓存（避免每次生成 schema 都探测外部依赖）
  - generation 版本号用于缓存失效

