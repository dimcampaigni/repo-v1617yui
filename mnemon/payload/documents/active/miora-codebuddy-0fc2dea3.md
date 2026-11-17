---
id: "0fc2dea3-af67-4834-80c3-d4d8193753af"
title: "Miora/CodeBuddy原生搜索入口：前缀纠正、执行契约与接入边界"
description: "2026-11-16正确原生搜索入口的源码、运行endpoint和真实来源证据；补充托管搜索阴性基线，保留域名过滤与Codex接入未验收边界。"
status: "active"
created_at: "2026-11-16T07:47:30.922Z"
updated_at: "2026-11-16T07:47:30.922Z"
content_hash: "af273db1a324db45d0f7207fdecf826ec0e6b948d56b8c749ea568bd7732a10d"
source_paths:
  - "research/native-search-20261116/runtime-entry-findings.md"
  - "research/native-search-20261116/provider-primary-sources.md"
  - ".cache/websearch-upstream-20261116/native-actual-endpoint-v2.json"
  - ".cache/websearch-upstream-20261116/native-correct-search.json"
  - ".cache/websearch-upstream-20261116/native-cli-owner.json"
  - ".cache/websearch-upstream-20261116/native-search-contract.json"
  - ".cache/websearch-upstream-20261116/native-safeconfig.json"
  - ".cache/websearch-upstream-20261116/native-prefix-urllib-result.json"
  - ".cache/websearch-upstream-20261116/native-search-probe.json"
session_ids:
  - "mnemon-run-v1-35bfaf53-2738-4d38-91c7-ecfe88ad9f57"
memory_body_ids:
  []
---

# Miora/CodeBuddy原生搜索入口：前缀纠正、执行契约与接入边界

## 范围及相关记录

本记录保存2026-11-16完成的原生入口调查，不是生产部署或Codex搜索恢复验收。相关既有Document为 **84028075-74d2-4c15-89b8-7a04f2a12637**，见[托管搜索阳性对照与SSE证据](<documents/active/codebuddy-sse-84028075.md>)。该记录覆盖既有relay上普通function成功、托管搜索未出现执行事件或来源的基线；本文补充后续找到独立执行侧搜索服务的证据，不改写既有Document。

**本次调查确认：**运行中的官方CLI使用endpoint `http://auth.proxy/codebuddy`，按官方路径拼接得到POST `http://auth.proxy/codebuddy/agenttool/v1/search`，实测返回HTTP200及5条结果，并读取其中一个Python官方来源核对HTTP200与标题。先前漏掉 `/codebuddy` 前缀的根路径404，不能用来否定正确入口或整个供应商的搜索能力。

**尚未确认：**Codex端到端托管搜索适配、引用与生命周期、严格域名过滤、取消和搜索计费。入口实测成功不代表这些要求已满足，也不保证未来持续可用。

## 第一方实现与请求契约

现场官方制品为 `@tencent-ai/codebuddy-code` 2.183.0。已完成调查核对公开同版源码与现场安装主文件：`dist/codebuddy.js` SHA256均为 `24f46a3fb614310fd3f62dd1d40b2b29e3b71a8aa32ceccc51e83277c25edca0`。此版本锚点不能自动延伸到其他版本。

- `WebSearch.performWebSearch` 调用 `${product.endpoint}/agenttool/v1/search`；需要当前authentication accessToken或明确authorization header。本记录不保存任何认证值。
- `buildRequestBody` 的已见结构为 `{query,type:"text2text",max_results:5,allowed_domains?,freshness?}`；`blocked_domains` 转成query中的排除site语法。
- 返回字段包括 `provider`、`results`（title/url/snippet等）与 `total_results`；官方客户端组织为 `web_search_tool_result`。这不是已经实现OpenAI Responses的 `web_search_call`、引用与事件生命周期。
- 运行配置的非敏感endpoint摘录确认本线路覆盖值为 `http://auth.proxy/codebuddy`。公版产品endpoint为 `https://www.codebuddy.ai`，internal产品为 `https://copilot.tencent.com`；这些公开地址的存在不表示本线路账户获准跨产品调用。调查未将Miora凭据带到其他产品测试。
- 容器 `CODEBUDDY_SANDBOX_PROXY_TOOLS` 包含WebSearch、WebFetch，仅说明工具声明，不能替代执行回执。

实现及配置证据：[同版实现定位与摘要](<.cache/websearch-upstream-20261116/native-cli-owner.json>)、[请求契约](<.cache/websearch-upstream-20261116/native-search-contract.json>)、[工具配置](<.cache/websearch-upstream-20261116/native-safeconfig.json>)、[实际endpoint摘录](<.cache/websearch-upstream-20261116/native-actual-endpoint-v2.json>)；第一方资料整理见[公开源码报告](<research/native-search-20261116/provider-primary-sources.md>)。

## 实测与正确解释

正确入口证据时间为2026-11-16 15:39–15:40（Asia/Shanghai），完整回执时间为 `2026-11-16T07:39:45.203543+00:00`。

| 路径 | 现场回执 | 结论边界 |
| --- | --- | --- |
| `/codebuddy/v2/chat/completions` | 空JSON返回400/code11101，不支持该非流式请求 | 仅作代理与应用错误来源对照，不是正常推理验收 |
| `/codebuddy/v2/responses` | 空JSON返回403 `path not allowed` | 当前代理权限边界；不能证明更下游不存在Responses，未绕过 |
| `/agenttool/v1/search` | 官方body探测返回404 `not support moudle` | 路径缺少实际endpoint前缀；不是正确搜索入口的阴性证据 |
| `/codebuddy/agenttool/v1/search` | HTTP200、provider为0、5条结果、完整1931字节；response_time_ms=2265 | 独立搜索取得来源回执；单次耗时不是性能承诺，Codex适配未实施 |

5条结果中两条来自Python官网，另三条来自 `libcloud.apache.org`、`blog.csdn.net`、`ityard.blog.csdn.net`，超出所请求的域名范围。因此**本次 `allowed_domains` 未严格过滤**，不能声称满足Codex域名限制。

返回的[Python 3.13.0官方发布页](https://www.python.org/downloads/release/python-3130/)随后读取HTTP200且title匹配，仅核对实际来源存在，不证明排名、内容完整性或它是查询时的最新稳定版答案。

回执：[正确搜索完整结果](<.cache/websearch-upstream-20261116/native-correct-search.json>)、[早期路径对照](<.cache/websearch-upstream-20261116/native-prefix-urllib-result.json>)、[缺前缀搜索探测](<.cache/websearch-upstream-20261116/native-search-probe.json>)。早期失败回执保留，但解释已在[现场核查报告](<research/native-search-20261116/runtime-entry-findings.md>)纠正。

## 复用方法与后续接入边界

1. **先核对运行配置，再拼接源码路径。**官方默认endpoint不一定等于现场覆盖值；错误前缀的404只证明该错误路径失败，不证明能力不存在。
2. **区分管理连接、模型调用和执行侧搜索。**此前Node undici访问容器envd超时；换Python urllib后取得既有runtime RUNNING、connect200、health204及命令200。管理通道成功只是允许继续实测，不是搜索成功。正确搜索请求另有完整结果与真实来源证据。
3. **跟踪实际调用数据流，不凭静态符号判断接口。**官方SDK的 `can_use_tool` 是权限询问，`mcp_message` 是另一个已注册服务器通道；SDK依赖中的Responses类型或WebSearchCall字符串，不能证明原生Responses服务可调用。可复用的sandbox-proxy WebSearch执行消息协议仍未核实。
4. **接入目标独立验收。**后续若获得实施授权，应分别核对执行结果映射、来源引用与生命周期、域名限制、取消及计费；不能只解除 `disabled` 或删除搜索拒绝守卫来宣称恢复。以上是待验证范围，不是已经批准的生产变更方案。

原生入口调查回执记录modelCalls=0、生产配置修改=0、新运行环境=0；与相关Document中的先前模型对照测试分开计数。未扩大代理许可、未跨产品测试凭据、未变更分组、并发、价卡或账务。本文仅保留可复用调查结论和来源定位，不保存临时执行代码、认证材料或运行进度。
