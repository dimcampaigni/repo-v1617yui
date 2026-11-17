---
id: "ca92adc8-0c04-43d9-a9ea-2ce80ec3d381"
title: "GPT/CC Switch适配第4–5轮：运行时拒绝诊断与客户验收交接"
description: "保存七模型服务端候选的验收分层、本地原版CLI策略拒绝证据及客户同回合脱敏取证要求；候选未上线、客户恢复未认证，不替代此前客户诊断。"
status: "active"
created_at: "2026-11-16T20:44:40.017Z"
updated_at: "2026-11-16T20:44:40.017Z"
content_hash: "11b34e3597ab64d661eb24233ce2b3335f94c84726d2900921de35947207ee71"
source_paths:
  - "research/server-client-adapt-20261117/HANDOFF.md"
  - "research/server-client-adapt-20261117/RUNTIME-RESULT.md"
  - "research/server-client-adapt-20261117/RUNTIME-SELF-CHECK.md"
  - "research/server-client-adapt-20261117/CODEMODE-RESULT.md"
  - "research/server-client-adapt-20261117/RESPONSES-RESULT.md"
  - "research/server-client-adapt-20261117/CUSTOMER-EVIDENCE.md"
  - "research/server-client-adapt-20261117/MODEL-DISCOVERY.md"
  - ".cache/server-client-adapt-20261117/runtime-round4/RUNTIME-EVIDENCE.json"
  - ".cache/server-client-adapt-20261117/runtime-round4/case-gpt-6.1-sol-v2/home/.sandbox/sandbox.2026-11-16.log"
  - ".cache/server-client-adapt-20261117/codemode-round1/round1-verification.json"
  - ".cache/server-client-adapt-20261117/responses-round2/verification.json"
  - ".cache/server-client-adapt-20261117/closure-round5/latest-metadata.json"
  - ".cache/server-client-adapt-20261117/first-summary.json"
  - ".cache/server-client-adapt-20261117/follow-summary.json"
  - ".cache/server-client-adapt-20261117/file-executions.json"
  - ".cache/server-client-adapt-20261117/client-return-verification.json"
session_ids:
  - "mnemon-run-v1-c23f3d86-756d-441a-8ecc-8f57e7062dd1"
memory_body_ids:
  []
---

# GPT/CC Switch适配第4–5轮：运行时拒绝诊断与客户验收交接

## 范围与相关记录

本记录仅整理2026-11-17完成检查点中的第4–5轮调查与交接，不是新的执行、提权、部署或取证授权，也不代表后续实时状态。维护时复用已给出的日志、报告片段与成功回执，没有重读仓库或重跑测试。

- 相关既有Document **3e285031-c4ae-454c-9e9e-bf6bcdcfdcf6**（[Codex 0.169客户诊断](<documents/active/codex-0-169-responses-lite-3e285031.md>)）覆盖客户协议准入、工具声明表面及仍未知的客户端形状。本文补充后续本地运行时实验与证据化交接，不重复其源码调查。
- 相关既有Document **69b9717f-69c5-454c-8a03-05f234c12a0a**（[此前普通工具窄修上线](<documents/active/gpt-69b9717f.md>)）覆盖此前三模型的已发布窄修。**该历史上线与本文七模型适配候选未上线是两批不同制品，不能混用状态。**

## 三层验收必须分别记录

| 层级 | 完成检查点可见证据 | 不能外推的结论 |
| --- | --- | --- |
| 普通function子链 | 七款模型各两轮，共14次新合成请求；工具名、参数、流式完成、实际隔离文件读取与结果回传通过；固定未修补CC Switch转换器的最终文字包含精确结果。 | 实际文件执行者是Harness PowerShell，不是客户电脑上的Codex；不证明全部复杂工具、完整Codex或客户恢复。 |
| 可达CodeMode及标准Responses候选 | 第1轮verification记录六款原工具形状通过，gpt-5.6-sol原形状不通过而请求绑定参数映射通过；第2轮记录标准Responses映射已本地整合并验证，审查最终证据匹配且限定范围未解决P0/P1为0。最终交接汇总确认两批候选均未部署。 | 只覆盖已验可达子路径；不能虚构native grammar、opaque历史、托管搜索等完整支持，也不能将审查回执当上线授权。 |
| 原版CLI与客户现场 | 本地原版CLI两次读取均未完成；客户同一失败回合的声明、转换前后形状及执行结果仍缺。 | HTTP200、turn完成或模型文字回答，都不是客户文件操作成功证据。 |

普通function覆盖：gpt-5.6-luna、gpt-5.6-sol、gpt-5.6-terra、gpt-6-astra、gpt-6-luna、gpt-6-sol、gpt-6.1-sol。原始摘要见[首轮工具调用](<.cache/server-client-adapt-20261117/first-summary.json>)、[结果回程](<.cache/server-client-adapt-20261117/follow-summary.json>)、[实际文件执行](<.cache/server-client-adapt-20261117/file-executions.json>)和[固定转换器验收](<.cache/server-client-adapt-20261117/client-return-verification.json>)。这些单样本完整流式耗时不构成五秒SLA验收。

候选证据入口：[CodeMode结果](<research/server-client-adapt-20261117/CODEMODE-RESULT.md>)与[第1轮verification](<.cache/server-client-adapt-20261117/codemode-round1/round1-verification.json>)；[标准Responses结果](<research/server-client-adapt-20261117/RESPONSES-RESULT.md>)与[第2轮verification](<.cache/server-client-adapt-20261117/responses-round2/verification.json>)。本检查点只给出部分摘要，不补写未见的实现细节或测试命令。

## 第4轮本地运行时：拒绝被诊断，不是执行通过

两次实验使用缓存原版Codex 0.167.0、真实服务端handler与目录；上游两轮响应由确定性mock生成。工具已声明，每次两轮HTTP200，但没有命令完成事件或真实文件值，因此runtimePassed=false。mock首轮不知道随机文件内容，第二轮只能从实际工具结果取值；当前没有工具结果，不能预填答案制造通过。

可见CLI日志明确记录`MXC availability: false`及`Windows sandbox selection: disabled`，见[原版CLI启动日志:1–4](<.cache/server-client-adapt-20261117/runtime-round4/case-gpt-6.1-sol-v2/home/.sandbox/sandbox.2026-11-16.log#L1-L4>)。项目报告将`read-only＋never`组合与同版官方策略的保守拒绝分支对应：无法落实受限边界时拒绝执行。该解释只诊断本地实验，不是客户0.169或真实上游的根因。

[运行时报告:27–39](<research/server-client-adapt-20261117/RUNTIME-RESULT.md#L27-L39>)保留以下实验边界：

- 使用独立缓存home/work及新随机文件，没有读取客户文件、密钥或聊天；无供应商推理、客户重放或生产部署。
- fixture拒绝了4次CONNECT企图，只证明这些外联企图被fixture拒绝，**不证明OS级网络隔离**。
- 本轮终端job已收取，loopback fixture按生命周期关闭；没有改机器账号、ACL、沙箱后端、允许规则或审批权限。
- 决策为业务源码和客户环境no-change：不把本地安全拒绝当服务端兼容bug，不加无关转译分支，不靠降低限制绕过。停止在同一disabled环境重复七模型执行。

下次本地真实运行时验收需要已授权、能实际实施只读限制的环境。不得擅自启用机器级沙箱、提权或要求客户换软件来补通过。工具执行的证据应绑定同一完成事件、退出成功与真实输出，再核最终文本，而非仅检查HTTP状态。结构化证据见[运行时证据](<.cache/server-client-adapt-20261117/runtime-round4/RUNTIME-EVIDENCE.json>)及[运行时独立自检](<research/server-client-adapt-20261117/RUNTIME-SELF-CHECK.md>)。

## 第5轮交接：持续缺口，不是服务端候选失败

最后一轮仅做客户最新时间戳及新增行数的只读检查，没有新模型请求、客户重放、日志采集变更或数据库写入。[第5轮元数据](<.cache/server-client-adapt-20261117/closure-round5/latest-metadata.json>)采样于2026-11-16T20:39:20.567551Z（2026-11-17 04:39:20.567551 +08）；新GPT用量与错误行数均为0。该观察仅限采样窗口，不证明客户以后没有新活动。

第3、4、5轮持续缺少同一客户失败回合的实际工具声明、CC Switch转换前后形状及执行结果，现有日志不存这些字段。本地另一版本CLI的安全拒绝不能替代该证据。最终goal回执因此为blocked并停止自动推进，**不是complete**；最终汇总同时记录服务器候选源码与验证匹配、productionChanged=false、customerRestoration=false。阻塞依据是三轮持续客户取证缺口，不是把本地CLI两次失败虚算为三轮。

已有报告指出固定参考CC Switch的第三方路由/聚合模型目录来自本机catalog，不向本站获取该目录。因此服务端metadata修正不能直接恢复客户端已删除的声明，更不能凭空造工具。接线依据见[模型发现报告](<research/server-client-adapt-20261117/MODEL-DISCOVERY.md>)。

## 继续调查所需的最小脱敏材料

原交接请求以下材料，必须来自**同一个失败回合**：

1. 客户CC Switch版本及当前路由模式。
2. 转换前后的`tools`／`AdditionalTools`声明；保留类型、名称、namespace及参数字段名，检查声明是否传入、被裁剪或转译。
3. 对应`tool_calls`回传、实际执行结果或错误提示；保留call_id及结构关系，以便关联声明、调用、执行和回程。

不需要API Key、聊天或文件正文，不要求换客户端。本站候选、费用、并发、客户软件与生产环境均不因本记录获得修改授权。已有约定验收证据继续复用；只有新的失败信号、实际制品改动或尚未覆盖要求才追加相应验证。交接入口见[候选与验收边界](<research/server-client-adapt-20261117/HANDOFF.md>)及[客户证据缺口](<research/server-client-adapt-20261117/CUSTOMER-EVIDENCE.md>)。
