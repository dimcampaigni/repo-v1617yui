---
id: "3f259eba-c29c-42fb-bcd0-73616b96ba0e"
title: "GPT全系适配新机发布：运行身份、工具回程与安全回退边界"
description: "保存2026-11-17新机1d1a815f发布、四次有限工具回程、CREDIT新账关联与发布接缝教训；补充历史候选文档，完整客户Codex恢复仍未认证。"
status: "active"
created_at: "2026-11-17T00:56:58.860Z"
updated_at: "2026-11-17T00:56:58.860Z"
content_hash: "a367ba576ea3cfb0424f9074d5be5f81f3c5cf4faebb776bca5d2b2f4edc1d93"
source_paths:
  - "research/gpt-client-release-20261117/DELIVERY.md"
  - "research/gpt-client-release-20261117/NEW-HOST-REVIEW.md"
  - ".cache/gpt-client-release-20261117/manifest.json"
  - ".cache/gpt-client-release-20261117/new-host-round7/retry4/FINAL-EVIDENCE.json"
  - ".cache/gpt-client-release-20261117/new-host-round7/retry4/post-canary-summary.json"
  - ".cache/gpt-client-release-20261117/new-host-round7/retry4/post-billing.json"
  - ".cache/gpt-client-release-20261117/new-host-round7/retry4/post-live.json"
session_ids:
  - "mnemon-run-v1-f68f55d8-73fd-49e6-813f-bc948f7d12fc"
memory_body_ids:
  []
---

# GPT全系适配新机发布：运行身份、工具回程与安全回退边界

## 状态与历史文档

本文整理2026-11-17完成检查点的新机上线及最终证据匹配，是时间点上的项目交接记录，不是生产操作授权或后续实时状态保证。本维护仅复用检查点和一次Document查重，没有重读源文件、重跑模型或测试。

相关既有Document保留不变：
- **ca92adc8-0c04-43d9-a9ea-2ce80ec3d381**（[GPT/CC Switch第4–5轮交接](<documents/active/gpt-cc-switch-4-5-ca92adc8.md>)）保存七模型候选未上线、本地CLI策略拒绝及客户取证边界。本文补充后续发布，不把旧候选阶段改写成当时已上线。
- **69b9717f-69c5-454c-8a03-05f234c12a0a**（[此前普通工具窄修上线](<documents/active/gpt-69b9717f.md>)）保存2026-11-16三模型六请求的另一批上线。本次制品、四请求及账单不能与其混算。

## 发布范围与运行身份

稳定版迁移完成后，才在香港新机发布已审五文件GPT服务端适配包；未重启旧站、接管迁移或修改客户端。发布回执为deployed，时间 **2026-11-17 08:44:53.767315（Asia/Shanghai，UTC+08:00）**，本次成功切换维护 **88.65秒**。

- 正式程序 **128016544B**，SHA256为 `1d1a815f1bde59047b4b0e14d16436ede4743799b8e31e27dd68cce1fc19c3f3`。
- 实际运行程序与正式文件SHA256一致，readonly bind确认通过；不是仅确认磁盘副本或容器健康。
- runtime fingerprint、Compose配置、网关源、index源与管理源未变；PG、Redis、Caddy等非目标容器未变，入口门控范围已恢复。
- 费用、倍率、账号并发、分组及保护100规则保持；旧站未启动，旧快照未覆盖新账。保持不变是相对重新取得的当前发布基线，不是回写历史配置。

证据入口：[上线交付](<research/gpt-client-release-20261117/DELIVERY.md>)、[最终制品证据](<.cache/gpt-client-release-20261117/new-host-round7/retry4/FINAL-EVIDENCE.json>)、[发布清单](<.cache/gpt-client-release-20261117/manifest.json>)及[运行核验](<.cache/gpt-client-release-20261117/new-host-round7/retry4/post-live.json>)。检查点没有五文件的精确差异，本文不从共享dirty列表反推发布文件。

## 有限的公网工具与目录验收

最终独立审阅匹配了4次新的合成公网请求，不是客户请求重放或真实客户Codex执行：

| 子链 | 实际模型 | 已证内容 |
| --- | --- | --- |
| 普通function首轮／结果轮 | gpt-6.1-sol | release_probe函数、value=17参数及随机nonce结果精确回程 |
| Sol alias首轮／结果轮 | gpt-5.6-sol | exec输入／代码保真、同request-bound alias函数wrapper及随机结果精确回程 |

4次均HTTP200、done=true、failed=false，ordinaryFunctionPassed、solAliasPassed与toolRoundtripAcceptancePassed均为true。验收看工具名、参数与随机结果回传，不只看HTTP200；alias wrapper通过不等于模型原生grammar enforced，也不是客户宿主文件执行认证。

双域目录HTTP200且allSevenPresent=true，列出gpt-5.6-luna／sol／terra、gpt-6-astra／luna／sol及gpt-6.1-sol，均tool_mode=direct、use_responses_lite=false。**七模型目录完整与两款模型的四次公网往返是不同覆盖面**，不能声称七款在本次都重复付费探测，或客户本地实际读取的目录与schema已认证。

交付汇总引用既有匹配回归的371个test/subtest通过事件、0失败；最终审阅复用既有证据，没有重新运行旧回归。这里是事件计数，不是371个独立测试案例；检查点未提供完整命令和覆盖清单，本文不补造。

来源：[工具回程摘要](<.cache/gpt-client-release-20261117/new-host-round7/retry4/post-canary-summary.json>)及[最终独立范围审阅:269–306](<research/gpt-client-release-20261117/NEW-HOST-REVIEW.md#L269-L306>)。最终审阅一次匹配摘要与已读运行／制品证据，未再次读取全部原wire或运行真实Codex／JS文件。

## 新账关联：唯一组合不是header直接外键

[账单读回](<.cache/gpt-client-release-20261117/new-host-round7/retry4/post-billing.json>)限定运营者key、起始账单marker及窗口，再按模型、token、effort核对：4次模型请求对应4条唯一新增记录，associationComplete=true、ambiguous为空。

| 请求标签 | 输入／输出token | 新增usageLogID | actual_cost（CREDIT） |
| --- | --- | --- | --- |
| function-first | 83／18 | 290353 | 0.00002249 |
| function-result | 130／22 | 290356 | 0.00003120 |
| alias-first | 95／34 | 290361 | 0.00009718 |
| alias-result | 169／22 | 290365 | 0.00009783 |

合计 **0.00024870 CREDIT**，currencyConversionApplied=false，不换算成USD或CNY。4条requestIDDirectHeaderMatch均false：证明是窄范围内模型／token／effort等组合唯一关联，**不是响应header UUID与数据库记录直接相等或外键映射**。existingLedgerChanged=false，未改旧账或删除历史错误；不外推全站计费审计，也不把上游usage的credit字段代替本站actual_cost。

## 发布接缝与可复用核查要点

检查点保留三次失败，成功后不抹去失败记录。发布前曾被商业指纹变化拦住；可见定位落在混元组描述和配置表示，价格、倍率、账号并发及GPT分组未变。处理是核清来源、保留现行配置、重新取得基线并保留商业守卫，不删除校验求绿或恢复旧价卡。

检查点收尾保存的发布教训如下，作为本项目核查要点，而非新用户全局规则：
1. 排空等待匹配真实请求预算；十分钟请求不能仅因等待三分钟就判卡死，仍须双在途归零。
2. Docker Mounts枚举顺序可能变化；完整挂载记录排序后再取指纹，保留全部字段和值，不靠忽略字段掩盖变化。
3. 原子发布前核对预声明目标SHA，并重核旧目的文件；仅比较当下源与副本不足以防漂移。
4. 断言保存异常类型及文件行号，不只保留空消息，不输出私密变量。r2缺原堆栈的证据限度仍保留，不补推其失败细节。
5. 商业指纹漂移逐字段核对并绑定当前配置，不放宽守卫或回写历史商业状态。

这些要点来自检查点中的可见教训读回，不足以重建每次失败的完整时序、脚本或精确堆栈；本文不编写可直接执行的发布命令。

## 回退入口与未认证边界

本批备份入口为 `/opt/backups/gpt-client-adapt-20261117-newhost-r4`，具体回退核对以[上线交付](<research/gpt-client-release-20261117/DELIVERY.md>)与[独立审查报告](<research/gpt-client-release-20261117/NEW-HOST-REVIEW.md>)为准。不得因为回退程序而直接启动旧站写入者，或用旧快照覆盖新账；本文没有完整回退步骤，不新增恢复命令或操作权限。

有限的新机发布／服务端工具回程／对应新账目标在父会话中关闭；以下事项仍未认证：
- 原客户完整Codex、CC Switch及0.169安装现场的文件工具恢复；原CLI被policy拒绝的环境恢复。
- Lite／namespace／custom已删schema的恢复、opaque历史、全部App/runtime与所有工具流程。
- 模型端原生grammar约束、七模型全部新公网往返或全场景可靠性／延迟SLA。

原CodeMode失败样本与counterfactual差别保留，不能洗成通过。工作区根不是Git仓库，子repo共享dirty；本次没有stage用户改动、提交或推送，部署成功不等于Git交付完成。已有证据覆盖当前有限制品时停止重复验收；只有真实新失败、制品变化或未覆盖需求才追加检查。扩大客户恢复声明需要对应现场证据。
