---
id: "c9d98798-d6bc-4609-99b4-e9d035fc0993"
title: "CodeBuddy IDE扩展发行来源核验：latest旧包与Marketplace 4.12鉴别"
description: "保存2026-11-16两套官方VSIX回执、包内版本与SHA256；说明latest直链不能代表当时Marketplace版本，发行核验不等于Codex接口验收。"
status: "active"
created_at: "2026-11-16T08:55:01.718Z"
updated_at: "2026-11-16T08:55:01.718Z"
content_hash: "585672831f4109d6430550ff4b73b97017934a313510aa44ef781d427e059308"
source_paths:
  - "research/native-search-20261116/current-interface-scope.md"
  - ".cache/codebuddy-ide-interface-20261116/download-receipt.json"
  - ".cache/codebuddy-ide-interface-20261116/vsix/extension/package.json"
  - ".cache/codebuddy-ide-interface-20261116/current-download-receipt.json"
  - ".cache/codebuddy-ide-interface-20261116/current-vsix/extension/package.json"
session_ids:
  - "mnemon-run-v1-1ec2e934-d8d2-4838-9433-33baa2c5178e"
memory_body_ids:
  []
---

# CodeBuddy IDE扩展发行来源核验

## 范围与关联记录

本记录保存2026-11-16（Asia/Shanghai，UTC+08:00）接口调查中的VSIX发行来源核验。证据是检查点完整可见的下载回执和包内清单，不依赖 `latest` 文件名或产品宣传。它不包含模型请求构造、扩展安装、实际推理或完整Codex验收。

相关既有Document **c1c69850-1eae-4dee-863a-d45f042353bf**（[CodeBuddy官方SDK无推理目录读取：12403后续核验与路由边界](<documents/active/codebuddy-sdk-12403-c1c69850.md>)）覆盖SDK目录与CLI版本调查。本记录仅补充IDE扩展的发行证据，不重复该文档的模型目录结论。项目调查归口为[接口核查记录](<research/native-search-20261116/current-interface-scope.md>)。

## 两套包的已见证据

两份包内清单均声明 `name=coding-copilot`、`publisher=Tencent-Cloud`；版本不同。

| 核验项 | 官方latest直链取得的包 | 本轮Marketplace来源包 |
| --- | --- | --- |
| 包内version | `4.10.33259736` | `4.12.39997873` |
| HTTP状态 | 200 | 200 |
| 字节数 | 18,448,500 | 21,738,300 |
| LastModified（服务器元数据，UTC） | 2026-07-16 07:08:51 | 2026-10-08 02:22:03 |
| FetchedAt（UTC+08:00） | `2026-11-16T16:48:20.2518762+08:00` | `2026-11-16T16:50:19.3956319+08:00` |

LastModified是发行对象元数据，不是本轮下载时间；这些观察不构成未来版本承诺。

### 官方latest直链

请求：`https://copilot.tencent.com/v2/plugin/download?platform=vscode&version=latest`

最终URL：`https://acc-1258344699.cos.accelerate.myqcloud.com/plugins/saas/vscode/coding-copilot-latest.vsix`

SHA256：`ED311C609DA2E2543BCD2950B54488F54F97C3BC9CEE166D6688CFE0B41EAAF5`

来源：[下载回执:2–11](<.cache/codebuddy-ide-interface-20261116/download-receipt.json#L2-L11>)、[旧包清单:3–7](<.cache/codebuddy-ide-interface-20261116/vsix/extension/package.json#L3-L7>)。

### Marketplace来源包

请求与最终URL一致：`https://tencent-cloud.gallerycdn.vsassets.io/extensions/tencent-cloud/coding-copilot/4.12.39997873/1791426121448/Microsoft.VisualStudio.Services.VSIXPackage`

SHA256：`8d5dd93a77dfa40ead5afd5aa7a0c7e180d7c1e3424e5060b60065a0693371c7`

回执中的实际SHA256与 `ExpectedMarketplaceSHA256` 一致，`HashMatched=true`。这证明本轮文件符合回执所记录的Marketplace预期摘要，不应外推为独立签名审计或未来仍为最新版。

来源：[当前包下载回执:2–10](<.cache/codebuddy-ide-interface-20261116/current-download-receipt.json#L2-L10>)、[当前包清单:3–7](<.cache/codebuddy-ide-interface-20261116/current-vsix/extension/package.json#L3-L7>)。

## 结论与可复用核验方法

1. **本轮官方latest直链实际提供4.10，而Marketplace来源包为4.12。** 判断分析对象必须使用包内版本和下载回执，不能把 `version=latest` 或 `coding-copilot-latest.vsix` 当作版本证明。
2. 本轮若研究4.12的静态实现，应以[当前包清单](<.cache/codebuddy-ide-interface-20261116/current-vsix/extension/package.json>)所对应的解包产物为对象；旧包证据不能无条件外推到4.12。
3. 对发行对象保留来源URL、最终URL、获取时间、字节数、摘要和包内版本；有预期摘要时单独记录比对结果。不同发行通道的时效需要分别核验。
4. **发行来源核验与协议能力核验是独立证据链。** 包下载成功、版本较新或产品宣称多模型支持，均不能证明原生Responses、客户端工具回调或完整Codex兼容。检查点没有包含4.12实际模型请求构造结果，本记录不补全这些结论。
