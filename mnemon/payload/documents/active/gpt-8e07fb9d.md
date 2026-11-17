---
id: "8e07fb9d-2bac-4bbc-9b5c-f0f176550a22"
title: "后台GPT报错诊断：普通函数准入与具名工具转译的可修边界"
description: "保存2026-11-16两处本地兼容遗漏、出站离线对照及统计纠正；限定未上线、复杂工具与连接reset仍未解决，供后续窄修复使用。"
status: "active"
created_at: "2026-11-16T14:44:08.558Z"
updated_at: "2026-11-16T14:44:08.558Z"
content_hash: "bd64f6758afa243d015df12f1cbe99ff6bfe33fb2f3adc63992babc017578cd2"
source_paths:
  - "research/gpt-errors-20261116/RESULT.md"
  - "research/gpt-errors-20261116/parameter-review.md"
  - "research/gpt-errors-20261116/transport-review.md"
  - ".cache/gpt-errors-20261116/named-tool-wire-result.json"
  - ".cache/gpt-errors-20261116/parameter-review/toolchoice-output.txt"
  - ".cache/gpt-errors-20261116/parameter-review/reasoning-output.txt"
  - ".cache/gpt-errors-20261116/transport-review/go-output.json"
  - ".cache/gpt-errors-20261116/reset-correlated-redacted.json"
session_ids:
  - "mnemon-run-v1-b46fe485-8fca-4fb5-8912-6477ac290ddc"
memory_body_ids:
  []
---

# 后台GPT报错：普通工具兼容遗漏与修复边界

## 范围与状态

本记录保存2026-11-16已完成的后台错误调查，作为后续窄修复的证据入口。**诊断完成不等于修复完成：本轮未修改业务源码、测试或生产配置，未发布，未认证客户完整Codex恢复。** 数量属于调查采样窗口，不能作为后续实时状态。

主报告为[完整诊断:7](<research/gpt-errors-20261116/RESULT.md#L7>)；独立审查及汇总见[诊断汇总:48](<research/gpt-errors-20261116/RESULT.md#L48>)、[参数审查](<research/gpt-errors-20261116/parameter-review.md>)和[传输审查](<research/gpt-errors-20261116/transport-review.md>)。本记录只使用完成检查点中已经可见的结论和回执，不补写未见的实现细节。

## 两处优先修复候选

| 候选 | 已见依据 | 可修范围与限制 |
| --- | --- | --- |
| GPT-6-Sol / Luna普通函数准入遗漏 | 诊断汇总归类最近2小时48条；现行身份的普通function支持配置已读回并确认身份匹配，报告结合生产基线判定旧“工具＋推理”准入判断仍误挡。源码提取还显示两款属于旧Sol/Luna推理守卫家族。 | 在当前已验证身份与普通function能力范围内修准入判断；不能由此解除搜索、图片工具结果、加密/密封历史等未验证能力限制。检查点未包含完整守卫实现，后续必须按实际生产代码定位，不能从模型名推断全部能力。 |
| GPT-5.6-Sol具名function选择格式遗漏 | 诊断汇总归类最近2小时2条。现网出站函数保留具名选择对象及两个工具；候选把唯一选中的函数保留为单一工具并发送字符串required。上游错误要求字符串，现网对象格式不兼容。 | 仅对合法、唯一匹配的具名函数做等价转译；保持该函数参数和单次派发，不改为auto，不靠重发制造成功。已有gpt-6.1-sol路径作为受影响范围对照，不能无条件扩大到全部模型或工具类型。 |

48＋2只是被归类的错误行数，**不能承诺50条原请求全部恢复**：同一请求还可能包含其他不支持的语义，也可能有后续供应商拒绝。

### 出站转译的直接离线证据

[出站对照回执](<.cache/gpt-errors-20261116/named-tool-wire-result.json>)记录passed=true，并明确使用真实发送函数的提取版本、合成数据和fake networkFetch，**只证明wire transformation，不涉及供应商调用或Codex恢复**：

| 模型与版本 | 出站tool_choice | 出站工具数量 | 派发次数 |
| --- | --- | ---: | ---: |
| gpt-5.6-sol，现网函数 | 具名function对象，选择f | 2 | 1 |
| gpt-5.6-sol，候选函数 | required | 1 | 1 |
| gpt-6.1-sol，现网函数 | required | 1 | 1 |
| gpt-6.1-sol，候选函数 | required | 1 | 1 |

[参数提取验证:2–20](<.cache/gpt-errors-20261116/parameter-review/toolchoice-output.txt#L2-L20>)注明提取来源为[发送函数:301–328](<.cache/gpt-errors-20261116/live-gateway.mjs#L301-L328>)，fetch完全stub、零socket，回执报告7例通过；可见唯一目标与遗漏对照均未修改输入。传输侧[提取核查回执:2–3](<.cache/gpt-errors-20261116/transport-review/go-output.json#L2-L3>)列出17个检查并注明isolated source-function extraction、no production repair。不要将这些离线结果包装成生产修复或真实客户验收。

## 排除误诊与未解决部分

### 不同模型的none规则不能拼成同模型循环

[推理守卫提取结果:1](<.cache/gpt-errors-20261116/parameter-review/reasoning-output.txt#L1>)显示：gpt-6.1-sol拒绝reasoning effort none并提示low/medium/high/xhigh/max；gpt-6-sol和gpt-6-luna在该检查中没有none错误；gpt-5.6-sol同样未在该检查中拒绝none。不同模型的建议和拒绝不是同一模型反复矛盾的证据。该结果只覆盖所提取的本地判断，不证明上游支持所有推理取值；本轮没有据此修改推理参数或错误提示。

### 连接中断与上游拒绝仍待定位

- 本机connection reset：报告归类24小时9条、最近2小时3条，定位到Sub2API → 本机网关这一段，但未核到根因。[脱敏关联记录](<.cache/gpt-errors-20261116/reset-correlated-redacted.json>)记录三组关联，各5条记录；身份数据已移除。不能据此归因为特定超时配置，也不能声称增加超时或重试即可修复。
- 11128渠道拒绝、11133未指明字段的参数拒绝：没有足够证据保证本地兼容改动能解决。
- 大量并发或无可用账号错误主要属于早前窗口；报告称最近2小时未新增，不能仅凭累计错误再调并发或账号规模。

## 数量口径

调查统计回执：24小时运维错误行1781，取样1781；用量记录41010，其中带输出40858，覆盖7款模型。**运维错误行不是独立失败请求；有输出用量也不是完整Codex或客户完整接收验收。** 不据此直接计算失败率。

HTTP200错误行最终校正为36：

- 35条Recovered：11128为29，11133为5，11101为1；29＋5＝34只是前两类小计，不是Recovered总数。
- 另1条为本地tool终止错误。

35条Recovered不能全部当终态失败，HTTP200也不能保证客户收到了完整输出。统计纠正保留原始记录，不删除日志或标resolved来制造“已修复”。

## 后续窄修复的证据要求

这是技术交接，不构成新增实施或发布授权：

1. 以当前生产制品及其真实源码对应关系为基线，核对身份、模型与协议范围；候选源码不能冒充生产版本。
2. 普通函数准入只处理已验证子集；具名选择仅在合法且唯一目标时等价转译，保留参数与单次派发。
3. 对照受影响模型、已有正常路径及不支持的复杂语义，复用上面离线回执解释原差异，并为实际改动取得匹配的新证据。
4. 分别报告本地验证、生产部署、自然请求和完整客户端验收；未获得供应商或客户端证据前，不把“参数格式已转译”写成“全部GPT/Codex恢复”。

## 相关既有Documents

- Document **84028075-74d2-4c15-89b8-7a04f2a12637**，[CodeBuddy托管搜索对照调查](<documents/active/codebuddy-sse-84028075.md>)：另一次实测区分普通function阳性对照与web_search无可验证执行；这里只引用其能力边界，不重复搜索调查。
- Document **c1c69850-1eae-4dee-863a-d45f042353bf**，[官方SDK目录与路由边界](<documents/active/codebuddy-sdk-12403-c1c69850.md>)：覆盖七款GPT所读配置默认Chat的限定结论；目录或路由读回不能替代完整Codex请求能力验收。
