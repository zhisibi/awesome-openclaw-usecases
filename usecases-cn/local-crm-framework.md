很多人想用 OpenClaw 做 CRM / 销售自动化，结果一般会卡在这几步：

- 要自己选数据库、搭表结构
- 要搞一个 Web 界面，不然数据只能用命令行看
- 还得接浏览器自动化、邮件、消息通道

最后搞出来往往是一个「半残的 Notion 集成」，既不稳定、也不好用。

这个用例讲的是 **DenchClaw** —— 一个专门为 OpenClaw 打造的本地 CRM + 工作流平台：

> 一条命令 `npx denchclaw` 起一个完整栈：DuckDB 数据库、Web UI、OpenClaw profile、浏览器自动化、技能系统，全都跑在你自己的机器上。

---

## 一、痛点：OpenClaw 很强，但搭 CRM 基建太痛苦

原文先讲了一个事实：

> 把 OpenClaw 用在真实业务（线索管理、外呼、销售漏斗）上，需要你自己拼一堆东西：

- 数据库（Postgres / MySQL / SQLite / DuckDB）
- Web 界面（React / Vue / 表格组件）
- 浏览器自动化（Playwright / Puppeteer）
- 消息通道（Telegram / Slack / Email）
- 文件管理和配置

对大部分人来说，这个工程量太大。

DenchClaw 的目标就是：

> 把这些全部封装成一个「本地运行的 CRM 操作系统」，让 OpenClaw 直接在这套环境里当「自然语言前端」。

---

## 二、DenchClaw 到底做了什么？

原文列了几个关键特性：

1. **一条命令安装**：

```bash
npx denchclaw
```

- 自动安装：DuckDB、Web UI、OpenClaw profile、浏览器自动化等
- 默认打开 `localhost:3100`

2. **自然语言 CRM**：

你可以直接跟它说：

> “Show me companies with more than 5 employees”

它会：

- 在 DuckDB 里跑对应查询
- 更新 Web UI 上的视图（表格 / 看板 / 日历）

3. **浏览器自动化 + 账号复用**：

- 复制你的 Chrome profile
- 让 Agent 拥有和你一样的登录态
- 可以执行：
  - 登录 HubSpot / Salesforce
  - 导出联系人
  - 导入到本地 CRM

4. **多种视图**：

- 表格（Table）
- 看板（Kanban）
- 日历（Calendar）
- 时间线 / 甘特图（Timeline）
- 图库（Gallery）
- 列表（List）

所有视图的配置（筛选条件、排序、列显示）都存成 **文件**，Agent 直接改 YAML 就能改界面。

5. **App Builder**：

- 可以让 OpenClaw 在 DenchClaw 里生成独立 Web App：
  - Dashboard
  - 工具面板
  - 简单小游戏

---

## 三、如何开始？

1. 安装并启动 DenchClaw：

```bash
npx denchclaw
```

2. 跟随引导完成初始化：
   - 创建一个名为 `dench` 的 OpenClaw profile
   - 启动对应网关（默认 19001 端口）

3. 在浏览器打开 `localhost:3100`
   - Safari 用户可以加到 Dock，当成 PWA 用

---

## 四、典型使用方式

文中给了几段对话式用法示例：

1. **创建 Leads 对象并导入 CSV**：

```text
Hey, create a "Leads" object with fields: Name, Email, Company, Status (New/Contacted/Qualified/Won/Lost), and Notes. Import this CSV of leads I downloaded from Apollo.
```

2. **筛选 + 看板视图**：

```text
Show me all leads where Status is "Contacted" and sort by last updated. Switch to Kanban view grouped by Status.
```

3. **浏览器自动化：从 LinkedIn 导入数据**：

```text
Go to my LinkedIn, find the last 20 people who viewed my profile, and add them as leads with their company info enriched.
```

4. **邮件草稿生成**：

```text
Draft a personalized outreach email to each lead in "New" status based on their company's recent news. Save the drafts in a new "Outreach Drafts" document.
```

这一整套操作都在本机完成，不依赖云端 SaaS 平台。

---

## 五、内置技能栈

DenchClaw 自带的几个核心 Skill：

1. **CRM Skill**：
   - 基于 DuckDB 的结构化数据管理
   - 支持对象、字段、记录、视图多种操作

2. **App Builder Skill**：
   - 让 OpenClaw 生成在 DenchClaw 内部运行的小 Web App

3. **Browser Automation Skill**：
   - 基于 Chromium
   - 使用你的 Chrome 登录态做自动化操作（抓数据、发消息等）

所有这些 Skill 都是“文件系统优先”的设计：

- 视图配置、过滤条件、布局、字段信息……全部是 YAML / markdown 文件
- Agent 可以像改代码一样改 UI

---

## 六、关键设计理念

原文里有几个很有意思的 insight：

1. **文件系统就是最佳的 Agent UI**
   - 不必为 Agent 再包装一层 Web API
   - 所有 UI 状态、配置都可以是可读写的文件

2. **DuckDB 是单机 CRM 的甜点位**
   - 既有完整 SQL 能力
   - 又不需要单独的服务进程和网络配置

3. **复制 Chrome Profile 是超级能力**
   - 避免跟一堆 SaaS 的 OAuth / API 限额搏斗
   - 让 Agent 真正「在你的浏览器里工作」

4. **一条 `npx` 命令 > 一个周末的环境折腾**
   - 这点对于希望“以业务为中心而不是以运维为中心”的人非常重要

---

## 七、适合谁用？

- 想把 OpenClaw 用在真实业务场景（销售、运营、客户管理）上的个人 / 小团队
- 希望尽量减少对第三方云服务依赖，偏好本地 / 私有部署
- 习惯 Git + 文件系统思维，把“数据 + UI 配置”都当成可以版本控制的资产

如果你已经有了 OpenClaw 网关，DenchClaw 会让它变成一套：

> 既能处理文档 / 代码，又能做 CRM / 销售自动化 / 浏览器操作的「本地业务操作系统」。

---

## 原文链接

- [Local CRM Framework with DenchClaw](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/local-crm-framework.md)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/local-crm-framework.md
---
