让你的 AI Agent 直接管理 API Key、直接调各种外部服务，看起来很“端到端智能”，但现实往往是安全事故温床：

- `.env.local` 里塞满了密钥
- 每多一个集成，就多一处潜在泄露点
- Agent 写的一堆 JS Skill / Shell 脚本，很难审计到底在对外做什么

这篇用例提出了一种更稳健的模式：**让 OpenClaw 把所有外部 API 调用都“委托给 n8n”**。Agent 只知道一个 webhook URL，不直接接触任何凭证；所有真实的 API 调用逻辑，都在 n8n 的可视化工作流里完成、审计、加阀门。

---

## 一、痛点：让 Agent 直接调 API 有哪些坑？

如果把所有事情都丢给 OpenClaw 自己处理，会遇到至少三类叠加的问题：

1. **可观察性差（No visibility）**  
   真正跑起来的东西，被埋在：
   - JavaScript Skill 文件
   - Shell 脚本
   - 零散的配置中

   想搞清楚“这个 Agent 在对外具体做了什么”非常费劲。

2. **凭证泛滥（Credential sprawl）**  
   每接一个新服务：
   - `.env` 多一个 API Key
   - Agent 运行环境里多一份敏感信息

   只要有一次不小心提交到 Git、或日志里打出了请求头，就可能直接泄露。

3. **浪费 Token（Wasted tokens）**  
   很多子任务其实是**确定性流程**：
   - 发一封固定模版的邮件
   - 在表格里追加一行记录
   - 拉取某个 API 的最新数据

   这些本可以直接由工作流执行，却每次都让 LLM 去“思考一次”，白白消耗推理 Token。

---

## 二、这个方案具体做了什么？

这套模式的核心，是把 OpenClaw 和 n8n 之间关系，设计成一种“**代理 / 代理人（Proxy pattern）**”：

- **代理模式（Proxy pattern）**  
  - 由 OpenClaw 来**生成 n8n 工作流（workflow）**，并为其创建一个带入站 webhook 的触发器
  - 之后所有外部 API 调用，都通过这个 webhook 进入 n8n

- **凭证隔离（Credential isolation）**  
  - 所有 API Key 都只保存在 n8n 自己的 Credential Store 里
  - OpenClaw 只知道：`http://n8n:5678/webhook/xxx` 这样的 URL

- **可视化调试（Visual debugging）**  
  - 所有节点、数据流都在 n8n 的拖拽式 UI 里一目了然
  - 出错时可以直接看每个节点的输入 / 输出

- **可锁定工作流（Lockable workflows）**  
  - 工作流在由 Agent 创建、测试完毕后，可以“锁定”
  - 锁定后 Agent 无法再修改它与外部 API 的交互方式

- **保护步骤（Safeguard steps）**  
  - 你可以在 n8n 工作流中加入：
    - 参数校验
    - 频率限制（Rate limiting）
    - 审批节点（例如需要人工点一下“确认发送”）

整体链路可以画成这样：

```text
┌──────────────┐     webhook 调用       ┌─────────────────┐    实际 API 调用    ┌──────────────┐
│   OpenClaw   │ ───────────────────→  │   n8n Workflow   │ ────────────────→ │  外部服务    │
│   (agent)    │   （无任何凭证）      │  （被锁定，      │   （凭证只在此    │  (Slack 等)  │
│              │                       │   持有 API Key） │    处保存）       │              │
└──────────────┘                       └─────────────────┘                   └──────────────┘
```

---

## 三、需要哪些技能 / 基础能力？

要跑通这条链路，至少需要：

- **n8n API 访问能力**  
  - 用来创建 / 更新 n8n 工作流
  - 用来触发测试运行

- **`fetch` / `curl` 等 HTTP 调用能力**  
  - 让 OpenClaw 可以向 n8n 的 webhook 发送 JSON 请求

- **Docker（如果用预配置方案）**  
  - 通过 Docker Compose 一次性拉起 OpenClaw + n8n

- **n8n 凭证管理能力**  
  - 在 n8n UI 里添加 / 管理 API Key
  - 每个集成只需要手动设置一次

---

## 四、如何搭建？

### 方案一：预配置 Docker 集合（推荐）

社区维护了一个现成的 Docker Compose 项目：[openclaw-n8n-stack](https://github.com/caprihan/openclaw-n8n-stack)，已经帮你把 OpenClaw 和 n8n 拉到同一个 Docker 网络里：

```bash
git clone https://github.com/caprihan/openclaw-n8n-stack.git
cd openclaw-n8n-stack
cp .env.template .env
# 在 .env 里填入你的 Anthropic API Key
docker-compose up -d
```

启动后你会得到：

- 端口 3456 上的 OpenClaw
- 端口 5678 上的 n8n
- 共享 Docker 网络：OpenClaw 可以直接请求 `http://n8n:5678/webhook/...`
- 一些预构建的工作流模版，例如：
  - 多模型事实核查
  - 邮件分拣 / triage
  - 社交媒体监控

### 方案二：手动集成

如果你已经有自己的 n8n 部署，也可以手动对接：

1. 安装并运行 n8n  
   - `npm install n8n -g`，或直接用 Docker 运行

2. 让 OpenClaw 知道 n8n 的 Base URL  
   - 比如在配置里写明：`N8N_BASE_URL=http://localhost:5678` 或 `https://n8n.yourdomain.com`

3. 在 AGENTS.md 中添加一段“n8n 集成规范”说明（示例）：

   ```text
   ## n8n Integration Pattern

   When I need to interact with external APIs:

   1. NEVER store API keys in my environment or skill files
   2. Check if an n8n workflow already exists for this integration
   3. If not, create one via n8n API with a webhook trigger
   4. Notify the user to add credentials and lock the workflow
   5. For all future calls, use the webhook URL with a JSON payload

   Workflow naming: openclaw-{service}-{action}
   Example: openclaw-slack-send-message

   Webhook call format:
   curl -X POST http://n8n:5678/webhook/{workflow-name} \
     -H "Content-Type: application/json" \
     -d '{"channel": "#general", "message": "Hello from OpenClaw"}'
   ```

4. 之后在具体用例中，Agent 按上述规范：
   - 先检查是否已有 `openclaw-slack-send-message` 之类的工作流
   - 若没有，就通过 n8n API 创建一个，并让你去 UI 里：
     - 填上 Slack Token / GitHub Token 等
     - 测试、锁定
   - 后续所有发送消息的操作，都只需要 POST 到对应 webhook。

---

## 五、关键洞见

1. **一次配置，三重收益**  
   这套模式同时在三个维度带来好处：
   - **可观测性（Observability）**：所有集成都在 n8n 可视化界面，出事容易排查
   - **安全性（Security）**：API Key 被隔离在 n8n，Agent 环境里不再存放敏感凭证
   - **性能 / 成本（Performance）**：确定性子任务交给工作流执行，减少 LLM Token 浪费

2. **“构建 → 测试 → 锁定”是关键节奏**  
   如果只让 Agent 不断改工作流，而你不锁定：
   - 有可能某次更新悄悄改变了调用方式
   - 安全边界和节流逻辑也可能被覆写

   正确姿势是：
   - 先让 Agent 帮你把工作流搭出来
   - 你在 n8n 里验证逻辑无误后，**锁定**
   - 后续 Agent 只能调用，不能再随意改。

3. **善用 n8n 自带的 400+ 集成节点**  
   n8n 已经内置了 400+ 服务的节点：
   - 大部分你想连的 SaaS / 工具，官方或社区节点都已经有
   - Agent 不需要自己写低层级的 HTTP 调用

4. **天然审计日志（Audit trail for free）**  
   n8n 会为每次工作流执行保留：
   - 输入 / 输出数据
   - 具体走过的节点

   这就天然成了一份“审计日志”，方便你事后检查“某次外部调用到底发生了什么”。

---

## 六、灵感来源与相关链接

这个模式最早由 [Simon Høiberg](https://x.com/SimonHoiberg/status/2020843874382487959) 提出，他总结了这种做法优于“让 OpenClaw 直接调所有 API”的三个核心理由：

1. 通过 n8n 可视化 UI 获得更好的可观测性
2. 通过凭证隔离实现更高安全性
3. 让确定性子任务由工作流执行，而不是每次都走 LLM

开源仓库 [openclaw-n8n-stack](https://github.com/caprihan/openclaw-n8n-stack) 提供了一个可直接运行的 Docker Compose 集合，把这个模式完整落地。

**相关链接：**

- [n8n Documentation](https://docs.n8n.io/)
- [openclaw-n8n-stack (Docker setup)](https://github.com/caprihan/openclaw-n8n-stack)
- [n8n Webhook Trigger Docs](https://docs.n8n.io/integrations/builtin/core-nodes/n8n-nodes-base.webhook/)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/n8n-workflow-orchestration.md
---
