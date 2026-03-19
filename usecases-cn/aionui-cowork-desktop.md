很多人已经在命令行或者 Telegram 里用 OpenClaw，但有几个常见痛点：

- 看不到 Agent 实际在文件系统、终端里做了什么，只能靠日志脑补
- OpenClaw 网关连不上、配置崩了，人又不在机器前，完全没法“远程自救”
- 手上不止一个 Agent（OpenClaw、Claude Code、Qwen Code 等），环境、MCP 配置一团乱

这篇用例讲的是：把 **AionUi** 当作「桌面 Cowork + 多代理中控台」，

- 让 OpenClaw 拥有一个可视化桌面工作区
- 帮你在 Telegram / WebUI 上远程“抢救”挂掉的 OpenClaw
- 把多个 Agent 收拢到同一个 UI 里统一管理和切换

---

## 一、痛点：只有 CLI / 聊天窗口，运维体验很糟

如果你现在的使用方式是：

- 在终端里跑 `openclaw` 或通过 Telegram Bot 聊天
- 把 OpenClaw 部署在一台常开的机器上（NAS、N1、小主机、服务器）

很容易遇到这些问题：

1. **“看不见”的工作空间**  
   你只能看到模型的回复，却看不到：
   - 它在编辑哪些文件
   - 它实际执行了哪些命令
   - 哪些网页被打开、爬取、浏览过

2. **OpenClaw 挂了就没法救**  
   当网关挂了、端口不通、配置写坏了：
   - 你正好不在那台机器旁边
   - SSH 也不一定随时能上去排查
   - 遇到“我连不上 OpenClaw”这样的典型远程事故，只能干着急

3. **多个 Agent 分散在不同 App**  
   例如：
   - 有时用 OpenClaw
   - 有时用 Claude Code / Codex
   - 偶尔还要切到别的模型

   每个 Agent 一套配置、一个终端或 App，MCP 配置重复配置不说，日常切换也很烦。

AionUi 的定位就是：**把这些都收在一个“桌面 Cowork”里**，同时内置一个“OpenClaw 部署与救援专家”。

---

## 二、OpenClaw + AionUi 具体能做什么？

用这套组合，你可以得到：

### 1. 桌面 Cowork：看得见的 Agent 工作区

- AionUi 会把 OpenClaw 作为一等公民 Agent 加载进来
- 在同一个桌面 UI 中，你能看到：
  - 文件读写（哪些文件被打开、修改）
  - 终端命令执行情况
  - 浏览器访问的页面
- 不再只是“聊天窗口”，而是一个完整的、可观察的工作环境。

### 2. 远程救援：OpenClaw 挂了也能修

AionUi 内置了一个 **“OpenClaw 部署专家”**，配合 Telegram / WebUI，可以：

- 当你在外面、电脑不在身边时：
  - 用 Telegram 连接到 AionUi
  - 让“部署专家”帮你跑 `openclaw doctor`
  - 检查网关、配置、端口占用等问题
  - 直接在远程把 OpenClaw 网关重启、修好

许多用户会把这套组合部署在一台常开的主机上，用来“看家护院”：

- OpenClaw 自己负责跑日常自动化任务
- AionUi 则负责提供一个“可以随时远程抢救 OpenClaw 的入口”

### 3. 多代理中控：一个 UI，管理全部 Agent

AionUi 支持：

- OpenClaw
- 内置通用 Agent（可用 Gemini / OpenAI / Anthropic / Ollama 等后端）
- Claude Code、Codex 等 12+ 不同角色的 Agent

你可以在同一个 Cowork 界面里：

- 切换不同 Agent
- 甚至并行运行多个 Agent 协作
- 共享同一套 MCP 服务器配置（下一点会说）

### 4. MCP 一次配置，多处复用

在 AionUi 里配置 MCP 服务器之后：

- 这套 MCP 设置会被同步给：
  - OpenClaw
  - 其他所有在 AionUi 中运行的 Agent

好处是：

- 不用在每个 Agent / 每台机器上重复复制粘贴 MCP 配置
- 换一个模型或 Agent，只是“换一个大脑”，而不是重配一遍工具链

### 5. 远程访问 & 定时自动化

- 支持通过 WebUI、Telegram、飞书、钉钉等远程访问 AionUi
- AionUi 自带简单的定时任务（cron）能力：
  - 可以定时唤起某个 Agent（包括 OpenClaw）
  - 跑一段维护任务或固定流程

---

## 三、你需要具备哪些前置条件？

要让 OpenClaw 和 AionUi 一起跑起来，大致需要：

- 一台可以长期运行的机器：
  - macOS / Windows / Linux 个人电脑
  - 或者一台 HomeLab 服务器 / NAS / 小主机
- 已经安装好的 OpenClaw（可选，AionUi 也可以帮助安装）
- 一两组模型 API Key（例如 Anthropic / OpenAI / 本地 Ollama 等）

AionUi 本身是一个桌面应用，负责：

- 提供 Cowork UI
- 管理多个 Agent 的会话
- 统一 MCP 配置
- 和 Telegram / WebUI 等远程入口打通

---

## 四、如何搭建：从 0 到 AionUi + OpenClaw

### 步骤 1：安装 AionUi

到 GitHub Releases 下载对应系统的安装包：

- [AionUi Releases](https://github.com/iOfficeAI/AionUi/releases)
  - macOS / Windows / Linux 均支持

安装完成后，启动 AionUi。

### 步骤 2：安装或接入 OpenClaw

如果你还没有装 OpenClaw，可以先在这台机器上安装：

```bash
npm install -g openclaw@latest
openclaw onboard --install-daemon   # 可选：装守护进程，24/7 常驻
```

之后，在 AionUi 里：

- 通常会自动检测到本机已安装的 OpenClaw
- 如果没识别到，可以打开 AionUi 里的 **“OpenClaw Setup / 部署助手”**：
  - 帮你配置网关
  - 写好基础配置文件
  - 跑 `openclaw doctor` 做自检

### 步骤 3：在 Cowork 里选择 OpenClaw

在 AionUi 中：

1. 创建一个新的 Cowork 会话
2. 在 Agent 选择里选中 “OpenClaw”
3. 进入工作区：
   - 左边是文件树、终端、浏览器视图
   - 右边是聊天 / 指令窗口

此时你就“看得见” OpenClaw 的所有操作了，而不仅仅是文本回复。

### 步骤 4：配置远程访问和“救援通道”

为了能在外网或手机上远程修 OpenClaw，你可以在 AionUi 里：

- 绑定 Telegram / 飞书 / 钉钉等账号
- 打开 WebUI 访问入口（如果部署在可公网访问的机器上）

之后，当出现：

- OpenClaw 连接失败
- 网关报错 / 不响应
- 配置改坏了

你都可以：

1. 在手机上打开 Telegram
2. 通过 AionUi 机器人连接到这台机器
3. 让内置的“OpenClaw 部署专家”帮你：
   - 跑 `openclaw doctor`
   - 修复配置和端口
   - 重启相关服务

---

## 五、关键实践建议

结合原用例，比较重要的几个实践经验是：

1. **让 AionUi 成为“多代理中控台”，而不是只给 OpenClaw 用**  
   - 把你常用的几个 Agent 都接进来
   - 把 MCP 工具统一配置在 AionUi
   - 用同一个 UI 管理所有 Agent 的会话

2. **把“部署 / 运维 OpenClaw”也交给 Agent**  
   - 利用内置的 OpenClaw 部署专家
   - 常规操作（升级、重启、检查）都通过它来做
   - 你只需要在关键步骤上点头确认即可

3. **紧急情况时，有一个“可视化 + 远程”的入口很关键**  
   - CLI 挂了、网关挂了，你只剩下日志很难处理
   - AionUi 提供了一个“另一扇门”，即使 OpenClaw 本体崩了，也能通过它来修

4. **把这个组合当成“OpenClaw 的图形外壳”和“生命维持系统”**  
   - OpenClaw 专注做智能决策和自动化
   - AionUi 负责可视化、远程通道、多 Agent 管理

---

## 六、相关链接

- [AionUi GitHub](https://github.com/iOfficeAI/AionUi)
- [AionUi 官网](https://www.aionui.com)
- [OpenClaw GitHub](https://github.com/openclaw/openclaw)
- [OpenClaw 文档](https://docs.openclaw.ai)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/aionui-cowork-desktop.md
---