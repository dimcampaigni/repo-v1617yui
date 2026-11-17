---
id: "c1c69850-1eae-4dee-863a-d45f042353bf"
title: "CodeBuddy官方SDK无推理目录读取：12403后续核验与路由边界"
description: "补充官方2.183.0仅初始化成功刷新目录的实证、七款GPT新缓存默认Chat的限定结论，以及2.184.0已见差异与Codex恢复的区分。"
status: "active"
created_at: "2026-11-16T08:28:50.384Z"
updated_at: "2026-11-16T08:28:50.384Z"
content_hash: "6048a1911fdbd5dc235fb4e3cab87a39d3d8713b4236a4f3f1d251754f76c02e"
source_paths:
  - "research/native-search-20261116/current-interface-scope.md"
  - ".cache/websearch-upstream-20261116/current_official_metadata_init.py"
  - ".cache/websearch-upstream-20261116/current-official-metadata-init.json"
  - ".cache/websearch-upstream-20261116/current-target-interface-keys-after-init.json"
  - ".cache/websearch-upstream-20261116/current-config-read-summary.json"
  - ".cache/interface-delta-20261116/package/CHANGELOG.md"
session_ids:
  - "mnemon-run-v1-b88e1602-3989-4fcf-97c5-8b2244d6c40a"
memory_body_ids:
  []
---

# CodeBuddy官方SDK无推理目录读取：12403后续核验与路由边界

## 范围与既有记录

本记录保存2026-11-16官方客户端目录读取的已完成调查步骤，不是生产发布、模型推理或完整Codex恢复验收。

相关既有Document **6d3de3da-0226-4b1e-abac-70a2b365958e**（[动态目录12403与合并缓存的证据边界](<documents/active/codebuddy-12403-6d3de3da.md>)）覆盖此前手写GET被12403拒绝及上午缓存。本记录补充其后的成功读取证据；旧请求失败仍是历史事实，但不能再据此判断官方目录持续不可读。原生独立搜索入口另见Document **0fc2dea3-af67-4834-80c3-d4d8193753af**（[原生搜索入口与接入边界](<documents/active/miora-codebuddy-0fc2dea3.md>)），不重复搜索契约，也不将独立搜索服务等同于Codex模型接口。

主证据：[接口核查记录:31–38](<research/native-search-20261116/current-interface-scope.md#L31-L38>)。

## 可复用方法：真实官方客户端初始化，不伪装手写请求

已有环境实际安装的官方 `@tencent-ai/codebuddy-code` **2.183.0** 仅执行SDK初始化：

- 使用 `--print`、输入/输出 `stream-json`、`--no-session-persistence`。
- 工具列表为空（`--tools`），MCP配置为 `{"mcpServers":{}}`，启用 `--strict-mcp-config`。
- 发送 `control_request`，请求 `subtype="initialize"`、`hasPrompt=false`；不发送用户提示词。
- 启动环境禁用自动更新、遥测相关开关和内置marketplace加载；这里记录传入开关，不宣称已独立审计全部后台行为。
- 不手工替换UA冒充渠道，不升级运行客户端、不启动替代Agent服务器、不修改生产配置。

实际回执：**退出0、事件类型control_response、初始化返回32条可选模型、stderr为0字节、耗时2.45秒**。报告记录 `userPromptSent=false`、`modelCalls=0`、`newAgentServer=false`。初始化成功不依赖重新发送此前失败的七款推理请求。

方法依据：[初始化脚本](<.cache/websearch-upstream-20261116/current_official_metadata_init.py>)、[官方初始化回执](<.cache/websearch-upstream-20261116/current-official-metadata-init.json>)。脚本在检查点中只有部分源码可见；本记录保留已见参数和成功回执，不补全未见实现。

## 新缓存与模型路由读回

缓存时间应与来源一起记录，不能把旧快照说成当前能力，也不能将不同采样合并为一个原子快照。

| 证据来源 | 现场结果 | 时间锚点（UTC） |
| --- | --- | --- |
| 官方初始化所观察的主产品缓存 | mtime从上午快照推进到本轮 | 之前2026-11-16T01:11:10.370255+00:00；之后2026-11-16T08:16:58.197246+00:00 |
| 后续读取的 cloud_product_config_cache | 34模型，目标七款齐全，api/url/labels覆盖均为空 | 2026-11-16T08:16:54.723506+00:00 |
| 后续读取的 CodeBuddy-Product-Cache:2.183.0 | 34模型，目标七款齐全，api/url/labels覆盖均为空 | 2026-11-16T08:16:55.296500+00:00 |

这些时间对应同日北京时间16:16附近。初始化返回的32条可选模型与缓存中的34条是不同来源的观察，未据此推测差异原因或声称所有环境配置一致。

目标七款为：`gpt-6-astra`、`gpt-6-sol`、`gpt-6-luna`、`gpt-6.1-sol`、`gpt-5.6-sol`、`gpt-5.6-terra`、`gpt-5.6-luna`。

依检查点已记录的官方工厂核查，这些采样配置仍选择默认Chat接口，不选择Responses或AICC。**这是当前所读配置的路由结论，不是后端所有入口的穷尽证明，也不证明当前身份能够承接完整Codex请求。**

字段证据：[远端与合并缓存读回](<.cache/websearch-upstream-20261116/current-target-interface-keys-after-init.json>)、[精简摘要](<.cache/websearch-upstream-20261116/current-config-read-summary.json>)。

## 版本差异只能按已见证据解释

官方npm latest在本轮返回 **2.184.0**，与运行版本2.183.0不同；本轮没有安装或升级生产客户端。2.184.0的[更新日志:29](<.cache/interface-delta-20261116/package/CHANGELOG.md#L29>)列出新增 `X-Feature-Module` 与 `X-Agent-Core: agent-cli` 归属标记；[更新日志:45](<.cache/interface-delta-20261116/package/CHANGELOG.md#L45>)描述模型原生拒答的保留与不自动重试。检查点也显示拒答透传相关源码片段，但完整接口对比报告未包含在有界证据内。

因此只保存上述已见差异，不认定新版新增了适用模型接口，不伪造归属头，不将拒答展示调整解释为11128已解除。

## 结论与验收分层

| 已验证观察 | 可以成立的结论 | 不可外推 |
| --- | --- | --- |
| 真实官方SDK初始化成功并刷新缓存 | 旧手写GET的12403不是官方目录持续不可读的证明；可无推理读取目录 | 所有目录读取方式均可用；12403解释Codex11128 |
| 新远端及合并缓存七款无api/url/labels覆盖 | 这些采样配置仍默认Chat | 服务器绝无其他模型入口；完整Codex兼容已证实 |
| 新版有归属头和拒答处理变化 | 存在可核对的客户端实现差异 | 新接口已找到；升级即可恢复 |

本轮**零新模型推理、零生产配置修改**。没有新的完整Codex成功回执，不能宣布11128解除或Codex恢复。后续判断仍须分别取得目录/路由证据、当前身份的合法调用依据以及真正的端到端协议验收，不能用模型名称、缓存刷新或初始化成功替代推理与搜索验收。
