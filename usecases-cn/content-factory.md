你是一名内容创作者，要在多个平台之间来回切换：找选题、做研究、写脚本、做封面，每一步都要亲自操刀。就算有一点 AI 帮忙，大多数时候也还是你在一遍遍手动喂提示、来回复制粘贴。每天的时间就耗在这些重复流程里。

如果有一支「专职小团队」，分别帮你盯热点、写文案、做封面，而且可以全自动在你睡觉时悄悄干完活——你只需要第二天起来审核、微调、发布，会怎样？

这篇用例讲的是：如何在 Discord 里搭建一条「多代理内容工厂流水线」，让不同的智能体分别负责研究、写作和视觉素材，在各自的频道里自动协作。

## 这个系统能做什么？

- **研究代理（Research Agent）**：每天早上扫描热点新闻、竞品内容和社交媒体表现，找出最值得做的内容机会
- **写作代理（Writing Agent）**：接收研究代理给出的最佳选题，产出完整的脚本、长文、线程或 Newsletter 草稿
- **缩略图代理（Thumbnail Agent）**：为每个内容生成 AI 缩略图或封面图
- 每个代理都有自己的 Discord 频道，所有输出按频道分类存档，方便回看与审核
- 整条流水线可以按时间表自动运行（例如每天早上 8 点），你醒来的时候，一整套内容已经准备好

## 要解决的痛点

内容创作大致分三段：**研究 → 写作 → 设计**。

大多数创作者都在自己做完这三件事：
- 选题：自己刷信息流、看竞品、做判断
- 写作：自己写脚本、写贴文、写邮件
- 设计：自己做封面、改缩略图、裁配图

即使用了一些 AI 写作工具，也往往是：
- 一次只处理一个步骤
- 每一步都要重新给提示、复制结果、粘贴到下一步

这套多代理系统的核心，是把这些步骤**串成一条自动化流水线**：前一个代理的输出自动成为下一个代理的输入，整个过程基本「免手动」。

## 你需要具备 / 配置的能力

要实现这一套「内容工厂」，你至少需要：

- 一个配置好机器人的 **Discord 服务器**，支持多个频道
- 类似 `sessions_spawn` / `sessions_send` 的能力，用来在不同会话中调度多个代理（子会话、多代理编排）
- 一个用于社交媒体/资讯研究的技能，比如 [x-research-v2](https://clawhub.ai) 或同类工具
- 本地图片生成能力（例如在本机跑 Nano Banana），或接入一个图像生成 API
- （可选）[knowledge-base](https://clawhub.ai) 技能，用于基于你自己的知识库做 RAG 检索增强

## 如何搭建这套内容工厂

### 1. 准备 Discord 环境

先创建（或让 OpenClaw 帮你创建）一个 Discord 服务器。你可以直接对 OpenClaw 说：

> “帮我设置一个 Discord 服务器，用来做内容工厂。”

### 2. 为不同代理创建频道

在服务器里创建至少三个频道，对应三类代理：

- `#research` —— 用于存放每天的热点研究与内容机会列表
- `#scripts` —— 用于存放写好的草稿、脚本和大纲
- `#thumbnails` —— 用于存放生成好的封面和缩略图

每个频道对应一个「职责明确」的代理，所有产出按频道归档，方便你回顾与反馈。

### 3. 给 OpenClaw 下「系统级」建厂指令

在 OpenClaw 里给出类似这样的长提示，让它帮你在 Discord 中搭建整条流水线（原文示例）：

```text
I want you to build me a content factory inside of Discord.
Set up channels for different agents:

1. Research Agent (#research): Every morning at 8 AM, research top trending
   stories, competitor content, and what's performing well on social media
   in my niche. Post the top 5 content opportunities with sources.

2. Writing Agent (#scripts): Take the best idea from the research agent
   and write a full script/thread/newsletter draft. Post it in #scripts.

3. Thumbnail Agent (#thumbnails): Generate AI thumbnails or cover images
   for the content. Post them in #thumbnails.

Have all their work organized in different channels.
Run this pipeline automatically every morning.
```

这段话的含义是：

1. **研究代理（#research）**：每天早上 8 点，自动研究你所在细分领域里：
   - 最新热点故事
   - 竞争者做过的内容
   - 在社交媒体上表现最好的内容形式
   最后给出「前 5 个最值得做的内容机会」，并附上来源链接。

2. **写作代理（#scripts）**：从研究代理产出的内容机会中，挑出最好的那个，产出完整的：
   - 视频脚本
   - 帖子线程
   - Newsletter 草稿
   然后把结果发到 `#scripts` 频道。

3. **缩略图代理（#thumbnails）**：基于选中的内容，为它生成 AI 缩略图或封面，发布到 `#thumbnails` 频道。

整条流水线每天早上自动跑一遍，所有产出依据不同步骤分频道存放。

### 4. 按你的平台做定制

如果你不做 YouTube，而是讲 Twitter / X、Newsletter 或播客，可以直接在原始提示上做定制（示例原文）：

```text
I focus on X/Twitter threads, not YouTube. Change the writing agent
to produce tweet threads instead of video scripts.
```

意思是：

> 「我的重心是 X/Twitter 线程，不是 YouTube。请把写作代理改成生成推文线程，而不是视频脚本。」

同理，你也可以改成：
- 生成 Newsletter
- 生成播客大纲
- 生成 LinkedIn 长文
- 生成博客文章草稿

只要改清楚「写什么形式」「面向哪个平台」，写作代理就会围绕你指定的形态输出内容。

## 关键思路与使用心得

- 真正的威力在于 **「代理串联」**，而不是某一个单独的代理：
  - 研究代理 → 给出有数据支撑的选题
  - 写作代理 → 把选题变成成品文案
  - 缩略图代理 → 为内容生成视觉资产
  整条链就像一条小型内容工厂流水线。

- 利用 Discord 频道做「工位划分」，可以非常清楚地看到：
  - 研究是否对路（热点是否相关、来源是否可靠）
  - 文案哪里需要调整（太长、太短、语气不对）
  - 封面视觉风格是否统一（颜色、构图、风格）
  你可以在对应频道里给出非常具体的反馈，例如「脚本太长」「多关注 AI 相关新闻」。

- 这套架构几乎可以适配任何内容形式：
  - 推文 / X 线程
  - Newsletter
  - LinkedIn 帖子
  - 播客大纲
  - 博客文章

- 对于图片生成环节，如果你在本地跑模型（比如在 Mac Studio 上跑 Nano Banana），有两个额外好处：
  - **成本更低**：不必为每张图付云端 API 费用
  - **可控性更好**：可以调自己的 LoRA / 模型版本，风格更统一

## 灵感来源

这套多代理内容工厂的思路，灵感来自 [Alex Finn 关于改变工作方式的 OpenClaw 用例视频](https://www.youtube.com/watch?v=41_TNGDDnfQ)。

## 延伸阅读

- [OpenClaw 子代理（Subagent）文档](https://github.com/openclaw/openclaw)
- [Discord Bot 开发者文档](https://discord.com/developers/docs)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/content-factory.md
---