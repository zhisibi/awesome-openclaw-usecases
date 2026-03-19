运行一台家庭服务器，意味着你要 7×24 小时“随叫随到”给自己的基础设施打工：

- 服务半夜 3 点挂掉
- 证书悄悄过期
- 磁盘突然写满
- 容器 / Pod 陷入 crashloop

……这些都可能发生在你睡觉或外出的时候。

这个用例展示的是：如何把 OpenClaw 打造成一个**常驻的基础设施 Agent**，让它通过 SSH 访问、自动化 cron 任务和一整套健康检查机制，在你意识到之前**就检测、诊断并修复问题**。

---

## 一、痛点：家庭实验室运维的真实负担

对自建家庭实验室、在家跑一堆服务的人来说，常见痛点包括：

- 健康检查、日志监控、告警系统都要自己一点点搭，一旦疏忽就失效
- 服务出问题时，你得掏出手机 SSH 上去排查、修修补补
- 基础设施即代码（Terraform、Ansible、Kubernetes manifests）需要持续维护
- 关于你整套环境的知识，全都只在你脑子里，不在可搜索的文档里
- 各种例行任务（邮件分拣、部署检查、安全审计）每周吃掉大量时间

这个 OpenClaw 基础设施 Agent 的目标，是把这些日常维护工作尽可能自动化，让“自愈”成为默认状态。

---

## 二、它具体能做什么？

这套自愈家庭服务器方案，大致包含以下能力：

- **自动化健康监控**  
  - 基于 cron 的定时检查：
    - 服务存活状态
    - 部署健康度
    - 系统资源（CPU、内存、磁盘）

- **自愈（Self-healing）**  
  - 通过健康检查发现问题后，可以自主执行修复动作：
    - 重启异常 Pod / 容器 / 服务
    - 动态扩缩资源
    - 修复配置错误

- **基础设施管理**  
  - 编写并应用：
    - Terraform 配置
    - Ansible Playbook
    - Kubernetes manifests

- **晨间简报（Morning Briefing）**  
  - 每天早上生成并发送一份摘要，包含：
    - 系统健康状态
    - 日程安排
    - 天气
    - 任务看板状态

- **邮件分拣（Email Triage）**  
  - 扫描收件箱：
    - 给“需要行动”的邮件打标签
    - 把噪音邮件归档

- **知识抽取（Knowledge Extraction）**  
  - 把笔记和对话导出，整理为结构化、可搜索的知识库

- **博客发布流水线**  
  - 从草稿开始到最终上线，自动跑完：
    - 生成封面图
    - 发布到 CMS
    - 触发托管平台部署

- **安全审计**  
  - 定期扫描：
    - 硬编码密钥
    - 运行特权容器
    - 过度宽松的访问权限

---

## 三、需要具备哪些技能？

要让这个基础设施 Agent 正常工作，你至少需要给它准备：

- 家庭网络机器的 **SSH 访问** 能力
- 管理 Kubernetes 集群的 `kubectl`
- 编排基础设施的 `terraform` 和 `ansible`
- 管理密钥的 `1password` CLI
- 访问邮箱的 `gog` CLI
- 日历 API 访问权限
- 一个 Obsidian 仓库或笔记目录（用于知识库）
- `openclaw doctor` 用于自检

这些工具构成了 Agent 的“手脚”和“感官”。OpenClaw 通过它们读写你的基础设施，并把结果记录在可审计的日志里。

---

## 四、如何搭建这套系统？

### 1. 核心 Agent 配置

首先，在 AGENTS.md 中定义你的基础设施 Agent 名称、权限与约束。例如：

```text
## Infrastructure Agent

You are Reef, an infrastructure management agent.

Access:
- SSH to all machines on the home network (192.168.1.0/24)
- kubectl for the K3s cluster
- 1Password vault (read-only for credentials, dedicated AI vault)
- Gmail via gog CLI
- Calendar (yours + partner's)
- Obsidian vault at ~/Documents/Obsidian/

Rules:
- NEVER hardcode secrets — always use 1Password CLI or environment variables
- NEVER push directly to main — always create a PR
- Run `openclaw doctor` as part of self-health checks
- Log all infrastructure changes to ~/logs/infra-changes.md
```

这里的关键点：

- 明确 Agent 名字（例如“Reef”）和职责范围
- 限定访问边界和敏感资源（SSH、Kubernetes、1Password 等）
- 用规则约束行为：
  - 不硬编码密钥
  - 不直接推 main 分支
  - 必须跑自检
  - 所有基础设施变更必须落盘记录

### 2. 自动化 Cron 任务系统

这套自愈方案的“灵魂”，就是 HEARTBEAT.md 里的定时任务配置。示例：

```text
## Cron Schedule

Every 15 minutes:
- Check kanban board for in-progress tasks → continue work

Every hour:
- Monitor health checks (Gatus, ArgoCD, service endpoints)
- Triage Gmail (label actionable items, archive noise)
- Check for unanswered alerts or notifications

Every 6 hours:
- Knowledge base data entry (process new Obsidian notes)
- Self health check (openclaw doctor, disk usage, memory, logs)

Every 12 hours:
- Code quality and documentation audit
- Log analysis via Loki/monitoring stack

Daily:
- 4:00 AM: Nightly brainstorm (explore connections between notes)
- 8:00 AM: Morning briefing (weather, calendars, system stats, task board)
- 1:00 AM: Velocity assessment (process improvements)

Weekly:
- Knowledge base QA review
- Infrastructure security audit
```

这个调度表定义了 Agent 的“心跳节奏”：

- 每 15 分钟：检查任务看板，继续未完成工作
- 每小时：跑健康检查与邮件分拣
- 每 6 小时：更新知识库、自检系统
- 每 12 小时：代码质量与文档审计、日志分析
- 每天：夜间头脑风暴、晨间简报、效率评估
- 每周：知识库 QA 与基础设施安全审计

### 3. 安全设置（重中之重）

在给 Agent 开 SSH、kubectl 等高权限入口之前，需要先做好防护。示例“安全检查清单”：

```text
## Security Checklist

1. Pre-push hooks:
   - Install TruffleHog or similar secret scanner on ALL repositories
   - Block any commit containing hardcoded API keys, tokens, or passwords

2. Local-first Git workflow:
   - Use Gitea (self-hosted) for private code before pushing to public GitHub
   - CI scanning pipeline (Woodpecker or similar) runs before any public push
   - Human review required before main branch merges

3. Defense in depth:
   - Dedicated 1Password vault for AI agent (limited scope)
   - Network segmentation for sensitive services
   - Daily automated security audits checking for:
     * Privileged containers
     * Hardcoded secrets in code or configs
     * Overly permissive file/network access
     * Known vulnerabilities in deployed images

4. Agent constraints:
   - Branch protection: PR required for main, agent cannot override
   - Read-only access where write isn't needed
   - All changes logged and auditable via git
```

核心思路：

- **TruffleHog 等工具作为 pre-push 钩子**，禁止任何带硬编码密钥的提交
- **本地优先的 Git 工作流**：先推到自建 Gitea，再推公开 GitHub
- **多层防护**：
  - 给 Agent 单独的 1Password 仓库
  - 对敏感服务做网络隔离
  - 每天自动跑安全审计脚本
- **强约束 Agent 改动范围**：
  - main 分支必须走 PR
  - 能只读就不写
  - 所有变更都有 git 记录可追踪

### 4. 晨间简报模版

这个 Agent 每天早上会生成并发送“早报”，模版类似：

```text
## Daily Briefing Format

Generate and deliver at 8:00 AM:

### Weather
- Current conditions and forecast for [your location]

### Calendars
- Your events today
- Partner's events today
- Conflicts or overlaps flagged

### System Health
- CPU / RAM / Storage across all machines
- Services: UP/DOWN status
- Recent deployments (ArgoCD)
- Any alerts in last 24h

### Task Board
- Cards completed yesterday
- Cards in progress
- Blocked items needing attention

### Highlights
- Notable items from nightly brainstorm
- Emails requiring action
- Upcoming deadlines this week
```

这让你每天一看早报，就能快速掌握：

- 今天的天气和日程
- 家中所有机器与服务的健康状态
- 昨天完成了什么，还有哪些任务被卡住
- 夜间头脑风暴产出的灵感与本周重要截止日

---

## 五、关键洞见

- **“我简直不敢相信我有了一台自愈服务器”**  
  通过 SSH、Terraform、Ansible、kubectl 等工具，Agent 可以在你还没意识到出问题之前，就自动修复基础设施故障。

- **AI 一定会硬编码密钥——除非你强制阻止**  
  这是最大的安全风险：
  - 如果不设防，Agent 会很自然地把 API Key 写进代码
  - 人类的“直觉安全感”在它身上并不存在
  - 所以**pre-push 钩子和密钥扫描是强制项**

- **本地优先 Git 流程是刚需**  
  不要让 Agent 直接把东西推到公开仓库：
  - 先推到自建 Gitea 当作“缓冲区”
  - 让 CI 做扫描
  - 通过人工审核后再推 GitHub

- **真正的产品，是 Cron 任务**  
  日常带来价值的，并不是你临时让 Agent 跑的命令，而是那一整套：
  - 健康检查
  - 邮件分拣
  - 晨间简报

- **知识抽取的价值是“复利”**  
  把笔记、聊天记录、邮件等持续抽取成结构化知识库，会随时间越来越有价值。
  
  有一个用户仅仅从自己的 ChatGPT 历史导出，就抽取出了 **49,079 条原子事实**。

---

## 六、灵感来源与相关链接

这个用例主要来自 Nathan 的长文记述：

> [Everything I've Done with OpenClaw (So Far)](https://madebynathan.com/2026/02/03/everything-ive-done-with-openclaw-so-far/)

文中他介绍了自己的 OpenClaw Agent“Reef”如何运行在家庭服务器上：

- 拥有对所有机器的 SSH 访问
- 管理一个 Kubernetes 集群
- 集成 1Password
- 管理一个超过 5000 条笔记的 Obsidian 仓库

Reef 运行着：

- 15 个活跃的 cron 任务
- 24 个自定义脚本
- 还曾自主构建并部署了一个任务管理 UI 等应用

在第一天就经历了一次 API Key 泄露事故之后，他的结论是：

> “AI 助手会心安理得地硬编码密钥，它们没有人类那种安全直觉。”

因此，他设计了一整套“纵深防御”安全设置（TruffleHog pre-push 钩子、本地 Gitea、CI 扫描、每日审计），非常值得任何尝试类似模式的人细读。

在 [OpenClaw Showcase](https://openclaw.ai/showcase) 中，`@georgedagg_` 也描述了类似模式：

- 部署监控
- 日志审查
- 自动修复配置
- 提交 PR

——这些都可以在他牵着狗出去散步的时候自动进行。

**相关链接：**

- [Nathan 的完整文章](https://madebynathan.com/2026/02/03/everything-ive-done-with-openclaw-so-far/)
- [OpenClaw 文档](https://github.com/openclaw/openclaw)
- [TruffleHog（密钥扫描）](https://github.com/trufflesecurity/trufflehog)
- [K3s（轻量级 Kubernetes）](https://k3s.io/)
- [Gitea（自建 Git 平台）](https://gitea.io/)
- [n8n（工作流自动化）](https://n8n.io/)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/self-healing-home-server.md
---
