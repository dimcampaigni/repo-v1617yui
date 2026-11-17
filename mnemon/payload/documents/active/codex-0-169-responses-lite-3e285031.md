---
id: "3e285031-c4ae-454c-9e9e-bf6bcdcfdcf6"
title: "Codex 0.169文件工具诊断：Responses Lite声明与协议准入边界"
description: "基于固定官方源码与客户定向调查，区分本地reasoning.context准入拦截、工具未声明和未正确调用；保存最小脱敏取证流程，客户恢复仍未验证。"
status: "active"
created_at: "2026-11-16T16:36:00.129Z"
updated_at: "2026-11-16T16:36:00.129Z"
content_hash: "a7d9affc3b354dd41f088724dec777113529aeaef140bd720ff7b63df326510c"
source_paths:
  - "research/codex-user-tools-20261117/DIAGNOSIS.md"
  - "sub2api-cny/backend/internal/pkg/apicompat/restricted_responses_content.go"
  - "sub2api-cny/backend/internal/pkg/apicompat/restricted_responses_tools.go"
  - ".cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/client.rs"
  - ".cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs"
  - ".cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/registry.rs"
session_ids:
  - "mnemon-run-v1-be55e4b7-536e-4ceb-a99c-654dd9aab567"
memory_body_ids:
  []
---

# Codex 0.169文件工具诊断：Responses Lite声明与协议准入边界

## 范围与来源

本记录整理2026-11-17（Asia/Shanghai）完成检查点中的定向调查，不是后续生产变更授权，也不宣称客户文件工具已恢复。主调查见[客户定向诊断](<research/codex-user-tools-20261117/DIAGNOSIS.md>)。官方实现限定Codex 0.169、commit `d32063f96c2dff1dc96af839b3952abb6acfeda4`；检查点报告固定tag/commit获取、原文读取及17/17 Git blob hash匹配通过，未运行该客户CLI或新模型请求。本维护记录未重新执行上述检查。

相关既有Document **69b9717f-69c5-454c-8a03-05f234c12a0a**（[GPT普通工具兼容窄修上线](<documents/active/gpt-69b9717f.md>)）覆盖三模型普通function往返与账单验收；其成功不能外推为本客户完整Codex、namespace或本地文件执行恢复。

## 已定位与仍未知的事实

- 用户207当时账号及密钥处于active，未发现本站停用；这不是文件执行能力证明。
- 11月16日18:59:21–19:00:05（+08）有4个时间点的Responses错误，运维日志每个时间点重复两行，共8行。均在本站本地request阶段以400阻断，报 `unsupported reasoning constraint: unsupported tool field "context"`；没有该批provider HTTP失败证据。不能按8行计算8个独立请求或失败实验。
- 11月17日00:14–00:15（+08）的自然请求实际从Chat Completions入口进入并有输出用量；独立access元数据支持入站path/protocol判型。UA声明 `codex-tui/0.169.0 Windows19045` 不能认证官方二进制，也不能排除站前包装/转译。
- 客户后来提供的是模型文字回答：当前会话没有可用文件读写或终端工具。它不是Windows/Codex权限异常，也不是实际工具声明的权威清单。
- 成功请求日志没有工具声明或正文，仅path/protocol/body_bytes等元数据；客户端有效配置和工具shape仍缺。**不能确定是工具未传入、注册被裁剪，还是工具已传入而模型/协议未正确调用；文件恢复保持未验证。**

这些观察及边界见[诊断:5–34](<research/codex-user-tools-20261117/DIAGNOSIS.md#L5-L34>)。旧0.167渠道拒绝、普通function上线、HTTP200或文字回复均不是本客户0.169文件操作根因或验收。

## 本站明确的准入拦截

[reasoning白名单:9–17](<sub2api-cny/backend/internal/pkg/apicompat/restricted_responses_content.go#L9-L17>)只允许 `effort`、`summary`，存在 `context` 即拒绝。通用[白名单错误生成:96–103](<sub2api-cny/backend/internal/pkg/apicompat/restricted_responses_tools.go#L96-L103>)导致错误称为“tool field”，实际被拦的是 `reasoning.context`，并非客户磁盘权限。

固定官方源码[reasoning构造:1145–1157](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/client.rs#L1145-L1157>)在 `use_responses_lite=true` 时携带 `ReasoningContext::AllTurns`，否则省略context并使用Responses默认 `current_turn`。关联条件不是 `supports_experimental_context`。不同上下文选择有语义；不能无声删除该字段换取200后声称修复，也不能吞掉工具调用改回普通文字。客户实际Lite/catalog值仍未知，源码关联不证明客户已经命中Lite转译丢工具。

官方该版仅支持 `wire_api=responses`，设置chat会在配置解析时报不支持；因此本站Chat入站需要核实来源与封装，不能直接要求客户切换Chat。切回Responses亦不保证解决本站context及其它完整Codex边界。

## 工具声明不止顶层tools

| 表面 | 固定源码事实 | 诊断限制 |
| --- | --- | --- |
| Responses Lite | `use_responses_lite=true`且prompt.tools非空时，声明进入input前缀 `ResponseItem::AdditionalTools`（developer），顶层tools为None。 | 顶层tools为空不证明无工具；需查input条目及其tools。只看顶层的转译器可能丢声明，但本站或客户是否发生此事尚未证实。 |
| code-mode / namespace | tool_mode先取ModelInfo，否则由code_mode_only/code_mode选择；可通过嵌套definitions和执行入口暴露能力。 | 直接function名字少、没有独立read_file，不能证明无文件操作。 |
| metadata | tool_info开关与Lite只控制tool_namespaces_info，model_visible_specs独立生成。 | metadata没有tool_info不等于无工具。 |
| tool_choice | 原生请求采用auto。 | 有工具也可能输出普通文字；文字回答不能证明工具往返。 |

来源：[Lite声明分支:1198–1213](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/client.rs#L1198-L1213>)、[模式解析:66–88](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/mod.rs#L66-L88>)、[嵌套声明链:1051–1149](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs#L1051-L1149>)、[metadata:500–506](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs#L500-L506>)、[spec独立生成:564–585](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs#L564-L585>)、[请求字段:1295–1303](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/client.rs#L1295-L1303>)。

检查点还显示本站 `ValidateVerifiedChatFunctionRequest` 对已验证窄路由检查function类型、唯一名称和选择约束；Responses input中的additional_tools会检查其tools，而native/custom等形态仍拒绝。由此不能将“支持普通function”当作“兼容所有Codex声明”。该函数完整来源路径未在摘录中显示，本记录不补造路径，也不将有限摘录当完整转换链审计。

## 真正能裁剪工具的条件与反例

- `features.shell_tool=false` 或选中catalog的 `shell_type=disabled` 可不注册shell；不等于所有MCP/utility/补丁能力关闭。
- `apply_patch_tool_type=null` 可不注册原生apply_patch；不等于不能经shell读取文件。
- resolved `ToolPolicy.require_unified_exec=true` **且** UnifiedExec关闭时shell注册return；单独关闭UnifiedExec不够，源码保留one-shot执行。
- resolved `require_managed_sandbox=true`且线程或所选环境有非Managed profile时，整组core tool sources可不注册；不是普通read-only sandbox的同义词。
- resolved `ToolPolicy.allows(tool_name)=false`会过滤对应工具；内部gate已核，不推定客户外部配置来源。

来源：[Managed gate:1231–1248](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs#L1231-L1248>)、[shell gates:1289–1319](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs#L1289-L1319>)、[patch gate:1472–1475](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/spec_plan.rs#L1472-L1475>)、[ToolPolicy过滤:534–560](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/core/src/tools/registry.rs#L534-L560>)。客户是否命中这些条件均未知。

反例：未知/自定义model slug的fallback明确设置shell_type=UnifiedExec，不自动禁全部工具；`experimental_supported_tools=[]`不是shell总allowlist。provider公开capabilities的external_web_access、remote_compaction、cyber_access_programs不是shell开关。普通sandbox路径/审批错误可能限制执行，但不是工具未声明的证据。参见[fallback:97–145](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/models-manager/src/model_info.rs#L97-L145>)、[capabilities定义:10–32](<.cache/codex-user-tools-20261117/client-review/raw/codex-rs/model-provider-info/src/capabilities.rs#L10-L32>)。

## 后续最小脱敏取证流程

1. 取得实际安装来源、启动方式及报错界面原文；打码账号、私人路径和内容。模型自述先归为待验证信号，不先加账号/Windows权限。
2. 核对有效model/provider/profile/features、wire_api及自定义catalog/model_catalog_url；删掉API Key、Bearer、env值、URL密钥和私人路径。无需运营者代客户执行PowerShell。
3. 仅在有合法既存capture时提取shape：源入口path；顶层tools数量/类型/名字及namespace子项；input里的AdditionalTools；code-mode入口/definitions；tool_choice；实际Lite和选中shell_type/apply_patch_tool_type/tool_mode。无需input/messages正文或客户文件内容。
4. 按证据区分“客户端未注册/未发送”“站前封装/本站转译遗漏”“工具已声明但选择或执行失败”，处理真实断点，不随机删字段、换接口、放宽守卫或提高权限。

停止边界：当前仍缺有效配置与实际工具声明；不把询问超时/沉默当授权，不以普通文字或HTTP200宣布恢复。完成调查轮未改生产、账户、计费、权限或客户端，未客户重放；普通function既有上线与客户文件工具恢复必须分别记录。
