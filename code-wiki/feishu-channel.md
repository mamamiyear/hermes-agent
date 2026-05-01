# Feishu/Lark 会话通道（实现解析 + 自动 @ 改造）

本页聚焦 Feishu/Lark 平台适配器在 Hermes Gateway 中的实现：入站事件归一化、群聊准入（require_mention）、回复链路、以及“群聊回复时自动 @ 原消息人”的支持情况与改造方案。

## 代码入口

- 平台适配器： [feishu.py](file:///workspace/gateway/platforms/feishu.py)
- 归一化事件模型： [MessageEvent](file:///workspace/gateway/platforms/base.py#L869-L915)
- Gateway 对 reply_to 的上下文注入： [gateway/run.py](file:///workspace/gateway/run.py#L5272-L5281)

## 入站链路（从 Feishu 事件到 MessageEvent）

核心流程在 [FeishuAdapter._process_inbound_message](file:///workspace/gateway/platforms/feishu.py#L2733-L2811)：

- 提取内容：`_extract_message_content()` → (text/type/media_urls/media_types/mentions)
- 文本规范化：
  - 纯文本消息会把首尾对 bot 的自提及剥离：[_strip_edge_self_mentions](file:///workspace/gateway/platforms/feishu.py#L1234-L1277)
  - 把 `@_user_N` 占位符替换成 `@name`：[_normalize_feishu_text](file:///workspace/gateway/platforms/feishu.py#L1136-L1155)
  - 把“提及列表”以提示形式注入给 agent：[_build_mention_hint](file:///workspace/gateway/platforms/feishu.py#L1215-L1232)
- Reply 上下文：
  - 从 `parent_id / upper_message_id` 推断被回复消息 ID，并拉取其文本：见 [feishu.py](file:///workspace/gateway/platforms/feishu.py#L2760-L2766)
  - 写入 `MessageEvent.reply_to_message_id / reply_to_text`（最终被 gateway 注入到 prompt）
- 构造来源信息：
  - `SessionSource.user_id` 优先 tenant-scoped user_id，其次 open_id（见 [_resolve_sender_profile](file:///workspace/gateway/platforms/feishu.py#L3519-L3550)）

## 群聊准入（require_mention）

群聊默认只响应“@机器人”的消息，关键逻辑在：

- 入口：[_admit](file:///workspace/gateway/platforms/feishu.py#L3715-L3749)
- 是否要求 @：[_require_mention_for](file:///workspace/gateway/platforms/feishu.py#L3751-L3756)
- 是否提及 bot：[_mentions_self](file:///workspace/gateway/platforms/feishu.py#L3804-L3844)

配置文档参考：[website/docs/user-guide/messaging/feishu.md](file:///workspace/website/docs/user-guide/messaging/feishu.md)

## 出站链路（send / reply）

发送入口： [FeishuAdapter.send](file:///workspace/gateway/platforms/feishu.py#L1694-L1750)

- 自动把 markdown-ish 内容转为 Feishu `post`（`msg_type="post"`），否则发 `text`（见 [_build_outbound_payload](file:///workspace/gateway/platforms/feishu.py#L3987-L3992)）。
- 如果 `reply_to` 传入，则调用 `im.v1.message.reply`，否则 `im.v1.message.create`（见 [_send_raw_message](file:///workspace/gateway/platforms/feishu.py#L4057-L4085)）。
- reply 失败时降级为新消息（撤回/找不到等场景）：见 [feishu.py](file:///workspace/gateway/platforms/feishu.py#L4200-L4240)。

## 结论：是否支持“群聊回复时自动 @ 原消息人”？

在改造前，不支持。

原因是：

- Gateway 的通用发送路径只会 `reply_to=event.message_id`，但不会在正文里插入任何 @ 提及信息（见 [BasePlatformAdapter._process_message_background](file:///workspace/gateway/platforms/base.py#L2778-L2787)）。
- FeishuAdapter 的出站 payload 构造只在“用户输入提及机器人”时做解析/门禁，并没有“发消息时构造 at 标签”的逻辑。

## 改造：群聊回复时自动 @ 原消息人（已实现）

实现目标：当机器人在群聊里回复用户消息时，自动在回复内容开头加入一个 @，提及触发这次对话的用户（即 `event.source.user_id`）。

### 配置开关

在 Feishu 平台配置的 extra 中开启：

```yaml
platforms:
  feishu:
    enabled: true
    extra:
      auto_at_reply_author: true
```

### 实现要点

- 在 BasePlatformAdapter 统一发送路径中，为 Feishu 群聊构造 `metadata["mention_user_id"]`：
  - 代码： [base.py](file:///workspace/gateway/platforms/base.py#L2658-L2672)
- 在 FeishuAdapter.send 中检测 `reply_to + mention_user_id + auto_at_reply_author`，强制首段使用 `post` 并插入 `{"tag":"at","user_id": ...}`：
  - 代码： [feishu.py](file:///workspace/gateway/platforms/feishu.py#L1694-L1749)
  - payload 构造：[_build_markdown_post_payload_with_mention](file:///workspace/gateway/platforms/feishu.py#L541-L575)

### 单测

- 覆盖测试： [TestFeishuOutboundAutoMention](file:///workspace/tests/gateway/test_feishu.py#L4666-L4705)

## 兼容性与注意事项

- 该实现只对 Feishu 平台生效，且默认关闭（需要显式开启 `auto_at_reply_author`）。
- `mention_user_id` 使用的是 SessionSource.user_id（可能是 tenant-scoped user_id，也可能是 open_id，取决于租户权限与 Feishu 事件 sender_id_type）。如果你的 app 只接受特定 id_type 的 @，需要进一步把 id_type 一并传入 metadata 并在 send 端按 id_type 选择字段。

