---
id: "5fd40983-0e71-446d-82f4-f480228f0887"
title: "CodeBuddy runtime接口分型与模型基址覆盖核查"
description: "保存ACP/E2B链接、local-proxy通用转发与五个CLI基址覆盖输入的只读证据，限定Codex兼容结论及合法核验条件。"
status: "active"
created_at: "2026-11-16T09:38:23.960Z"
updated_at: "2026-11-16T09:38:23.960Z"
content_hash: "cba2e6f7af01ae526b5067f355a4eca25deb79e1cb7dc50522fddb69ad15f651"
source_paths:
  - "research/native-search-20261116/current-interface-scope.md"
  - "research/native-search-20261116/local-port-proxy-interface-review.md"
  - ".cache/websearch-upstream-20261116/runtime-link-endpoint-shapes.json"
  - ".cache/native-search-docs-20261116/codebuddy-agentos-api.yaml"
  - ".cache/local-proxy-interface-20261116/index.js"
  - ".cache/local-proxy-interface-20261116/pathProxy.js"
  - ".cache/local-proxy-interface-20261116/targetHost.js"
  - ".cache/websearch-upstream-20261116/final-model-base-overrides.json"
  - "research/gpt-codex-assessment-20261116/actual-codex.json"
  - ".cache/websearch-upstream-20261116/native-prefix-urllib-result.json"
session_ids:
  - "mnemon-run-v1-39ae5996-6a84-4f77-8ddc-5809b4969856"
memory_body_ids:
  []
---

# CodeBuddy runtime接口分型与模型基址覆盖核查

## 范围与关联记录

本记录保存服务器领取研究中心在2026-11-16完成的接口边界调查，依据检查点可见的源码摘录、脱敏只读回执与独立审查报告。它不是生产发布或完整Codex恢复验收，也不证明上游全面不存在其他模型接口。

相关既有Document：
- **6d3de3da-0226-4b1e-abac-70a2b365958e**：[动态目录与合并缓存的证据边界](<documents/active/codebuddy-12403-6d3de3da.md>)。本记录补充runtime连接用途与基址覆盖输入，不重复目录调查。
- **0fc2dea3-af67-4834-80c3-d4d8193753af**：[原生搜索入口与接入边界](<documents/active/miora-codebuddy-0fc2dea3.md>)。独立搜索成功不等于模型协议兼容。

主证据见[接口核查记录:5–22](<research/native-search-20261116/current-interface-scope.md#L5-L22>)。

## 1. Runtime链接：ACP连接与E2B沙箱

实际runtime接口字段已只读取回HTTP200，并与第一方AgentOS schema对应：

| 字段 | 已见形状与用途 | 不能据此认定 |
| --- | --- | --- |
| acpLink | 指向 `/acp`，用于ACP连接 | 新的OpenAI Responses模型路由 |
| sandboxLink.endpoint | `/v2/agentos/links/runtimes/[resource-id]`，E2B SDK API地址 | 同属runtime就等于模型入口 |
| dataPlaneEndpoint | 已知envd数据面主机，E2B数据面 | 数据面可达就等于模型协议、鉴权或工具归属兼容 |

来源：[脱敏runtime字段回执](<.cache/websearch-upstream-20261116/runtime-link-endpoint-shapes.json>)、[AgentOS schema:3781–3827](<.cache/native-search-docs-20261116/codebuddy-agentos-api.yaml#L3781-L3827>)。调查未携Miora凭据跨产品调用这些地址，未发ACP或DirectPrompt业务请求。

## 2. Local-proxy：数字端口到localhost的通用转发

已读实现将 `/proxy/{port}` 后的剩余路径转发到 `localhost:{port}`，支持HTTP与Upgrade/WebSocket。父代理读取相关实现并复核独立审查；已检查实现没有提供模型路由或鉴权注入，唯一显式头设置为 `x-forwarded-proto=https`。

这只证明通用端口转发，不能推导新的模型WebSocket入口或原生Responses兼容。结论限定独立审查覆盖的四个文件，不覆盖任意目标端口实际运行的服务。

来源：[入口:10–46](<.cache/local-proxy-interface-20261116/index.js#L10-L46>)、[HTTP发送:125–133](<.cache/local-proxy-interface-20261116/pathProxy.js#L125-L133>)、[目标主机:1–8](<.cache/local-proxy-interface-20261116/targetHost.js#L1-L8>)、[独立端口代理审查](<research/native-search-20261116/local-port-proxy-interface-review.md>)。

## 3. 最终模型基址：补查覆盖输入，未发现新路由

脱敏只读回执时间为 `2026-11-16T09:32:50.620719+00:00`（Asia/Shanghai为当日17:32:50）。核查对象是五个既有官方CLI进程及两份用户/origin设置，不创建新runtime、不修改配置、不发模型推理。

- 五个进程均未设置 `CODEBUDDY_BASE_URL`。
- 各进程已读 `ACC_PRODUCT_CONFIG_V3` 与 `ACC_PRODUCT_CONFIG_V2` 的endpoint均为 `http://auth.proxy/codebuddy`，未见模型URL覆盖。
- 两份已读设置 `/root/.codebuddy/settings.json` 与 `/root/.codebuddy-origin/settings.json` 均未提供该环境基址覆盖。

这些观察补足进程环境、产品配置与两份设置的覆盖项，不能声称读取了所有项目级设置、所有环境或服务器全部动态路由。结合已核对的官方工厂基址选择，在已读配置条件下，`resolveModelBaseURL` 使用endpoint加 `/v2`，Chat构造器追加 `/chat/completions`，现有确认通路仍是 `/codebuddy/v2/chat/completions`，不是缺少前缀。

来源：[最终基址输入回执](<.cache/websearch-upstream-20261116/final-model-base-overrides.json>)、[基址与接口分流记录:18–30](<research/native-search-20261116/current-interface-scope.md#L18-L30>)。

## 4. 判型与后续合法核验条件

截至该证据快照，正常Codex CLI 0.167.0七款模型的最后实测均为11128渠道拒绝；候选 `/codebuddy/v2/responses` 的空JSON路径探测返回403 `path not allowed`。这是既有证据的复用，本轮未重发；没有新的完整Codex成功证据。来源：[正常客户端结果](<research/gpt-codex-assessment-20261116/actual-codex.json>)、[原生路径回执](<.cache/websearch-upstream-20261116/native-prefix-urllib-result.json>)。

可复用核验顺序：先确认连接属于模型推理、独立搜索、Agent会话、E2B沙箱还是通用转发；再核对运行endpoint与模型级、环境及设置覆盖；最后取得具体官方调用依据和当前身份获准使用路由的证据，才开展针对性兼容验证。

继续验证需要同一上游具体新模型接口调用依据、实际路由配置变化或现有通路正式放行的证据。HTTP/WS传输成功、Agent最终文字或沙箱健康不能替代Codex模型协议与工具归属验收；不重复旧403/11128请求、不伪装渠道、不越过代理许可。缺少后端全部路由与准入规则，应保留未知范围，不能将本次排除项外推成供应商全面无接口。

本次runtime/代理/基址核查未修改生产、权限、渠道身份、账务或账号池，未发新的推理或Agent业务请求。
