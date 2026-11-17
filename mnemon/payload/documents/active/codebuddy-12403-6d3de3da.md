---
id: "6d3de3da-0226-4b1e-abac-70a2b365958e"
title: "CodeBuddy模型接口定位：动态目录12403与合并缓存的证据边界"
description: "保存2026-11-16现有上游模型路由调查：动态目录实际拒绝、七款GPT缓存无路由覆盖、与Codex11128的证据区分及后续合法核验条件。"
status: "active"
created_at: "2026-11-16T08:12:03.412Z"
updated_at: "2026-11-16T08:12:03.412Z"
content_hash: "59353b24dc24aa6157d1c09c0ef2330fd9e1733c4d2afde7361dbfb8a9e90404"
source_paths:
  - "research/native-search-20261116/current-interface-scope.md"
  - ".cache/websearch-upstream-20261116/current-remote-model-config.json"
  - ".cache/websearch-upstream-20261116/current-target-interface-keys.json"
  - ".cache/websearch-upstream-20261116/current-model-cache-routes.json"
session_ids:
  - "mnemon-run-v1-da97fe10-1ebc-4b05-95f9-2e780ca8510c"
memory_body_ids:
  []
---

# CodeBuddy模型接口定位：动态目录12403与合并缓存的证据边界

## 调查范围与关联记录

本记录保存2026-11-16对现有上游模型接口的调查，不是生产发布、实时目录取得或完整Codex恢复验收。本次用户明确纠正："错了！不是寻找供应商，而是寻找接口"。因此调查目标是找到有官方调用依据、当前身份获准使用的模型接口，而不是将换供应商或联系供应商当作接口定位成果。

相关既有Document为 **0fc2dea3-af67-4834-80c3-d4d8193753af**，见[原生搜索入口与接入边界](<documents/active/miora-codebuddy-0fc2dea3.md>)。该文档保存独立搜索执行侧的入口调查；本记录补充模型目录与路由选择证据，不重复其搜索契约，也不将独立搜索服务等同于Codex模型推理接口。

主证据是[接口核查记录:21–29](<research/native-search-20261116/current-interface-scope.md#L21-L29>)；下文保留其明确结论与限制，不补全检查点中缺失的源码或接口字段。

## 动态目录：找到了读取入口，未取得实时内容

按同版CloudProductProvider的配置读取逻辑，先读取当前 `config.url`，未覆盖时使用GET `/codebuddy/v3/config`。本轮确实执行了目录GET，返回 **HTTP400 / code12403**，错误原文为 `check ua, get coding copilot version error`。见[目录读取回执](<.cache/websearch-upstream-20261116/current-remote-model-config.json>)。

这一结果只证明此次目录读取被客户端版本校验拒绝：

- 尚未取得实时模型目录与最新模型路由字段。
- 12403不是此前正常官方Codex 0.167请求返回11128的原因证据，两类请求与错误必须分开解释。
- 本轮没有修改UA冒充客户端或绕过校验；不能把找到目录路径写成取得可用模型接口。

## 已有合并缓存：弥补静态配置缺口，但不是实时目录

本轮按官方存储键名及MD5本地路径读取已有 `cloud_product_config_cache` 与 `CodeBuddy-Product-Cache:2.183.0`，不是仅检查静态基础配置。当前所选已有环境的两份缓存均含 **34个模型**，下列 **7个目标模型齐全**：

`gpt-6-astra`、`gpt-6-sol`、`gpt-6-luna`、`gpt-6.1-sol`、`gpt-5.6-sol`、`gpt-5.6-terra`、`gpt-5.6-luna`。

两份缓存中的七款均未见模型级 `api`、`url`、`labels` 覆盖。缓存mtime为2026-11-16 **09:11（Asia/Shanghai，UTC+08:00）**。字段与时效见[目标接口字段记录](<.cache/websearch-upstream-20261116/current-target-interface-keys.json>)。

另一已有环境的 **10:04（同日、UTC+08:00）** 远端配置缓存同样未见七款专属路由字段，见[另一环境的缓存读取摘要](<.cache/websearch-upstream-20261116/current-model-cache-routes.json>)。两个环境的缓存不能合并解释为同一个实时快照。

根据本轮源码核查与上述缓存条件，官方工厂仍选择默认Chat接口。这个结论的限定条件是**已读取的配置与缓存**，不是服务器所有接口的穷尽证明。

## 可以成立与不能成立的结论

| 观察 | 可以成立的结论 | 不可外推的结论 |
| --- | --- | --- |
| 动态目录GET返回12403 | 找到了真实目录读取入口，但本次读取失败 | 已取得实时模型路由；12403解释正常Codex的11128 |
| 合并缓存中七款无 `api/url/labels` 覆盖 | 补足只看静态基础配置的缺口；这些缓存条件下仍选默认Chat | 当日下午实时目录没有其他路由；服务器绝无其他接口 |
| 七款模型出现在目录缓存 | 缓存包含目标模型声明 | 当前身份能承接完整Codex请求 |

本轮记录为**零模型推理请求、零生产配置修改**，没有重发此前七款相同的11128请求。检查点未给出新的完整Codex成功证据，因此状态仍是：**未获取已验成功的新Codex模型接口，完整Codex未恢复**。这不是对供应商全部能力的否定。

## 后续证据化交接

下一步需要的是**现有官方客户端读取的最新模型目录及实际路由依据**，不是再做同路径的七款失败推理。对后续新证据应同时保存：

1. 目录或配置的获取时间、来源环境与缓存mtime，明确实时读取还是旧快照。
2. 合并后的模型 `api/url/labels` 及官方路由选择依据；只有基础配置或模型名称不足以确定实际接口。
3. 当前身份允许该路由的依据；有官方调用依据与授权路由后，才考虑最小核验。

保持本次调查边界：不换上游代替接口定位、不包装Agent最终文字代替协议接入、不伪装渠道、不越过403、不访问未经授权internal路径。新的成功回执出现前，继续区分目录读取、模型请求与搜索执行三条证据链。
