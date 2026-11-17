---
id: "84028075-74d2-4c15-89b8-7a04f2a12637"
title: "CodeBuddy 当前接入线路托管搜索核查：阳性对照、SSE证据与结论边界"
description: "2026-11-16 gpt-6.1-sol 既有relay的普通function与托管web_search对照调查；保留完整流验证方法、管理连接超时的归因边界及后续入口核查方向。"
status: "active"
created_at: "2026-11-16T07:11:44.674Z"
updated_at: "2026-11-16T07:11:44.674Z"
content_hash: "a1e99f1ef89eddddde0bf8d3be5766e62a6159fd693cf1fe4012979ea74aca69"
source_paths:
  - ".cache/websearch-upstream-20261116/probe_full_result_remote.py"
  - ".cache/websearch-upstream-20261116/provider-full-result.json"
  - ".cache/websearch-upstream-20261116/provider-probe-diagnostic.json"
session_ids:
  - "mnemon-run-v1-16237afb-0498-4e3f-a0d8-f4b4a8e0b1bd"
memory_body_ids:
  []
---

# CodeBuddy 当前接入线路托管搜索核查

## 范围与核心结论

本记录基于2026-11-16完成的上游能力调查。用户当轮要求先查看上游是否支持；调查不是搜索恢复验收，也未实施生产修复。

测试对象为 `gpt-6.1-sol`，报告路径为既有 relay → `auth.proxy /codebuddy/v2/chat/completions`。在该模型、线路及请求形态下，普通 function 调用成功；托管 `web_search` 请求收到完整响应，但未观察到可验证的搜索执行或来源记录，最终正文为 `SEARCH_UNAVAILABLE`。**HTTP200不能作为搜索完成或恢复的证据，也不能据此将整个CodeBuddy供应商判为不支持搜索。**

## 可复核实测

完整报告的采样时间为 `2026-11-16T07:05:04.591027+00:00`，即北京时间15:05:04。两个用例均为单次请求，共用 `stream=true`、`max_tokens=256`、`reasoning_effort=low`、`tool_choice=required`。

| 用例 | 请求要点 | 实际响应 | 流与搜索证据 |
| --- | --- | --- | --- |
| 普通function阳性对照 | 要求调用无外部副作用的 `compat_probe`，参数value为17 | HTTP200；正确调用 `compat_probe`，参数为 `{"value":17}`；无正文回答 | 7个已解析SSE事件，5825字节，未截断；`[DONE]` 存在，`finish_reason=tool_calls` |
| 托管搜索 | `tools=[{"type":"web_search"}]`；要求查python.org当时的稳定版与官方URL，不可编造结果，无浏览能力时明确回答不可用 | HTTP200；正文为 `SEARCH_UNAVAILABLE`；无tool calls | 5个已解析SSE事件，3951字节，未截断；`[DONE]` 存在，`finish_reason=stop`；搜索事件0、来源记录0 |

此阳性对照说明相同线路能够完成普通工具调用，不能将搜索阴性结果简单归结为整条推理线路不可用或所有工具均失败；它不证明托管搜索字段被上游正确解释，也不证明其他接口拥有同样能力。

## 管理通道阻塞与结论边界

此前直接provider探针未到达可用于判定搜索能力的结果。诊断回执显示Node运行阶段报 `UND_ERR_CONNECT_TIMEOUT`，`networkPhase=sandbox.envd.run`，尝试连接沙箱管理端口443时超时。该失败属于管理执行通道连接阻塞，不是搜索请求被接口拒绝。随后改用已有推理relay取得上述对照证据。

- 结论只覆盖本次模型、既有relay、Chat Completions调用方式和所发送的托管搜索描述。
- 零事件计数表示未观察到探针识别的 `web_search*` 事件以及 `search_results`、`sources`、`annotations` 列表记录；不能据此证明供应商所有后台或原生入口均无搜索能力。
- 供应商其他原生搜索入口仍未核实。检查点中的 `/v1/responses` 404说法没有相应工具回执，本文不将其列为已验证结果。
- 不应仅解除本站 `disabled` 或删除托管搜索拒绝守卫，就宣称搜索已恢复。能力必须由真正搜索执行与可靠来源证据确认。

## 可复用探测方法

1. 使用既有、非保护的可用付费relay，先跑无外部副作用的普通function阳性对照，再跑托管搜索用例；保持模型、流式设置与工具选择条件一致。
2. 在有界上限内先采集响应，再解析完整SSE流，检查 `[DONE]`、finish reason、tool calls、可见搜索事件与来源记录，最后才输出摘要。此次采集上限为512KiB，两例均未触限。若触限，必须保留截断标记，不能声称流已完整结束。
3. 将HTTP状态、流终止、工具调用和真实搜索结果分别验收，避免把普通文本中自称联网或HTTP200当作执行证据。
4. 若要继续调查，应先核对供应商原生搜索协议与入口，再做对应的小范围实测；保持管理通道连接失败和搜索能力判定分离，不重复同一超时路径来推断模型能力。

本次报告记录配置变更为0、未新建运行环境、所测账号不在保护集合内。上述方法是此次调查的技术记录，不是后续生产操作授权。

## 证据入口

- [完整对照探针](<.cache/websearch-upstream-20261116/probe_full_result_remote.py>)：请求形态、512KiB采集上限及SSE解析方式。
- [完整实测回执](<.cache/websearch-upstream-20261116/provider-full-result.json>)：两个用例的状态、终止标记、调用参数、事件/来源计数及未截断标记。
- [管理通道诊断回执](<.cache/websearch-upstream-20261116/provider-probe-diagnostic.json>)：连接超时类型与发生阶段。

以上为检查点中可见的本地缓存来源路径；缓存不是长期保留承诺。本文保存关键调查结论与依据，不依赖重新读取缓存才能理解结果。
