---
id: "09d3e37f-275f-433a-92e3-172437dba44d"
title: "一枝春手机私有安全看板：首版架构、证据边界与运维交接"
description: "2026-11-14首版独立Worker/D1轻量监控的五路采集、真实鉴权验收、数据未知规则、覆盖盲区、凭据续配与安全停用流程。"
status: "active"
created_at: "2026-11-14T03:27:21.362Z"
updated_at: "2026-11-14T03:27:21.362Z"
content_hash: "3de635f147b41f101af94ffdfc9d8e577073fa0f9cac8a8f98ab39553f772ad5"
source_paths:
  - "research/security-monitor-implementation-20261114/上线记录.md"
  - "deploy/security-monitor/collector.py"
  - "deploy/security-monitor/worker.mjs"
  - "research/security-monitor-implementation-20261114/server-readback.json"
  - "research/security-monitor-implementation-20261114/access-verification.json"
  - "research/security-monitor-implementation-20261114/web-verification.json"
  - "deploy/security-monitor/business_check_remote.py"
session_ids:
  - "mnemon-run-v1-a00a7c52-40fa-4ca4-bbe2-4e6b7d622612"
memory_body_ids:
  []
---

# 一枝春手机私有安全看板：首版架构、证据边界与运维交接

## 记录范围与入口

本记录整理2026-11-14首版实施交付的可复用知识，依据提供的完整[上线记录](<research/security-monitor-implementation-20261114/上线记录.md>)及成功验证输出，不代表后续仍在运行或已排除入侵。目标是在手机私有网页查看本人一枝春中转站的攻击尝试、入侵迹象和采集健康。

入口：https://security.yizhichun.online 。使用独立只读监控密码，不是中转站账号密码。密码只在本机私有配置中保留并向用户交付，本记录不保存密码、采集令牌或其他秘密。

## 隔离架构与数据流

- 独立Cloudflare Worker `yzc-security-monitor`、独立D1及专用子域名，不新增源站公网端口，不复用主站路由承载监控。
- 源站采集器/SQLite位于 `/opt/yizhichun-security` 与 `/var/lib/yizhichun-security`，配置目录 `/etc/yizhichun-security`。`yizhichun-security-collect.service` 与 `.timer` 约每分钟增量采集，只外送白名单字段；扫描日志只读，不删除原日志或应用 `audit_logs`。
- 配置资源上限为CPUQuota 10%、MemoryMax 192M；这不是实测日常占用。
- 既有SSH/fail2ban、Caddy、Cloudflare防护和客户业务保持运行；本轮未购买服务，未启用自动封禁、反击或模型训练。
- 实现源码：[collector.py](<deploy/security-monitor/collector.py>)、[worker.mjs](<deploy/security-monitor/worker.mjs>)。

### 五路观察范围

1. Web：Caddy敏感路径与路径穿越规则记录。
2. SSH：失败认证与成功登录日志；成功登录是待核对线索，不能直接视作入侵。
3. 主机：精选14个文件的哈希/权限变化，范围有限。首次基线只能证明之后的变化，不能证明此前安全。
4. 边缘：Cloudflare安全事件详情样本。
5. 应用：管理员接口成功变更审计，按admin路由区分，不能把普通用户成功POST当成管理员越权。

## 状态语义与避免误报

- 首版是轻量规则及哈希/权限观察，不是完整EDR。调研中的CrowdSec和AIDE尚未安装，不能声称等价于其完整场景库或可信基线FIM。
- 超过3分钟的旧快照：今日数字置 `null`、来源标 `stale`、总状态 `unknown`；断网、缺字段、源失败或配额受限也不得变成绿色安全或虚假的0。
- 各来源数字不能相加成精确攻击总数。SSH可能重复，Cloudflare事件可能抽样，挑战也可能来自合法客户端。
- 界面最近最多150条安全事件，优先保留high/warning，避免高风险证据被边缘样本淹没。
- 本轮新增监控service/timer导致的两项文件变化只按这次安装标为预期，不豁免未来变更。应用历史普通活动误分类已纠正，保留原数据与备份。
- 没有未解释的high事件不等于没有入侵；成功SSH及管理员动作仍需本人逐项核对，未逐一认证全部历史行为合法。

## Cloudflare权限、留存与盲区

- 实际授权验证：Free可读取本次详情数据集，但不允许 `firewallEventsAdaptiveGroups` 聚合。因此看板展示本地已记录样本，单轮最多1000条，不是完整全天恶意攻击总量。
- 本域只读Analytics凭据的交付时到期日为2026-12-11，需续配并重新核验采集成功；失效必须显示来源失败/未知，不得显示0。此记录仅保存生命周期信息，不含令牌。
- 源站本地库留存31天；独立D1保存当前摘要和7天趋势，不是不可篡改长期取证仓。
- 源站root失陷仍能干扰采集；内存后门、未知漏洞和未记录的权限滥用属于盲区；独立只读密码不等于MFA。
- 不保存客户聊天/请求正文或上游密钥，不外送源站/应用敏感字段；共享Worker/D1额度仍需观察。

## 已有验收证据及其限制

以下为2026-11-14约11:20–11:22（Asia/Shanghai）的验收，不自动延伸为未来实时健康证明。

- 鉴权：匿名GET数据401、错误密码401、登录缺同源Origin 403、正确密码200；会话Cookie具备 `__Host-`、HttpOnly、Secure、SameSite=Strict；退出200，退出后数据401；无采集token上传401。见[公网鉴权验证](<research/security-monitor-implementation-20261114/access-verification.json>)。
- 手机界面：390×844真实Chromium完成登录、真实分源数据显示和退出，无横向溢出或页面JS错误。测试通过本机已有代理访问公网，未放宽站点CSP。见[手机浏览器验证](<research/security-monitor-implementation-20261114/web-verification.json>)及[截图](<research/security-monitor-implementation-20261114/mobile-live.png>)。
- 断采边界：当前部署Worker代码的超过3分钟旧快照处理已检查，未为验证修改生产数据。上线记录另载UI断网、缺字段和恶意文本边界检查。
- 采集：远端timer active，service `Result=success` / `ExecMainStatus=0`，五路来源最近成功；Web增量重复检查新增0。见[服务器最终读回](<research/security-monitor-implementation-20261114/server-readback.json>)。
- 原业务：主站 `/` 与 `/health` 返回200，`/api/v1/auth/me` 与 `/v1/models` 匿名401。检查脚本为[business_check_remote.py](<deploy/security-monitor/business_check_remote.py>)。这只证明这些检查的状态与鉴权预期，没有执行付费推理或全场景性能压测，不能据此声称所有业务无回归。

## 停用与回退原则

仅停监控采集：

```sh
systemctl disable --now yizhichun-security-collect.timer
```

不要停止业务、SSH/fail2ban或删除原始日志。停采集超过3分钟后看板应显示旧证据/未知。旧采集器bootstrap与 `events.before-audit-classification.sqlite` 保存在监控私有目录，保留作回退和分类核对。

Cloudflare侧仅禁用独立 `yzc-security-monitor` 或移除其专用domain，不动已有 `yzc-upload-gzip` 或主站路由。操作前核对目标资源身份，不能把监控停用扩大为业务网络调整。

## 实施中已验证的取舍

- 旧Tunnel证书权限不足不是当前本人会话不可用的证明；实施最终用已登录管理API完成发布。
- 大单行资产读取被截断曾造成两次无效发布，改为分行字节模块后成功；发布时需要核对模块读取完整性，不把工具截断误认为源码损坏。
- 既有WAF拒绝默认Python User-Agent，`workers.dev` 存在网络解析异常；实施保持WAF，用明确标识的自有域名上传，不为自动化测试放松生产防护。
- 审计分类必须按管理员路由，而不是把所有POST视为管理变更；修分类时保留原始证据，不能靠删除历史记录压低风险显示。
