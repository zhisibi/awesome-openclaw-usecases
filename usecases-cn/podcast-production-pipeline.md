你有一个不错的播客点子，甚至已经列了一长串未来要录的选题。但真正动手的时候，发现时间都耗在这些事情上：

- 为嘉宾做背景调研
- 查资料、整理要点
- 写开场白、串场和收尾
- 录完还要产出 Show Notes、节目描述
- 各平台的宣传文案、社交媒体内容也要一条条写

结果是：**真正的对话和创作只占了 30%，剩下 70% 都是重复的“生产流水线”工作。**

这篇用例介绍了一条「播客制作流水线」，通过串联多个 Agent，让你只需要给出一个主题（和嘉宾信息），就能自动拿回一整套**可直接发布的节目素材**。

## 一、痛点：创作者被“生产工作”拖垮

对于个人播客和小团队来说，典型现状是：

- 真正录音的时间很少
- 前期调研和策划特别耗时
- 节目录完，Show Notes 往往被“随便写写”或干脆放弃
- 社交媒体宣传更容易成为“最后一项，直接跳过”

创作者最有价值的部分——高质量的对话、视角和表达——反而被淹没在大量重复、机械但又不能省略的“制作工作”中。

这个 Agent 流水线的目标就是：**接管那 70% 的制作工作**，让你只专注于：

- 想哪些话题值得聊
- 找谁来聊
- 在录制时把对话聊好

## 二、这个流水线具体能做什么？

整个播客制作 Agent 流水线，从一个选题，一路产出到可发布的全套素材，大致包括：

1. **节目调研（Episode Research）**
   - 根据一个话题或嘉宾名字：
     - 汇总嘉宾背景、近期工作、观点、争议点
     - 汇总该话题的关键趋势、最新进展、常见误解
     - 提炼出适合节目讨论的要点和角度

2. **大纲与脚本（Outline & Script）**
   - 生成结构化的节目大纲，通常包括：
     - 开头 Hook（1–2 句抓耳的引子）
     - 30 秒左右的自然口语化开场介绍
     - 5–7 个从浅到深、从破冰到深入的访谈问题
     - 若对话冷场时可以用的 2–3 个“备用问题”
     - 收尾环节和 Call-to-Action（比如订阅、评论、访问链接）

3. **Show Notes（节目笔记）**
   - 录制完成后，根据节目的逐字稿（Transcript）：
     - 生成带时间戳的 Show Notes
     - 每次话题切换都有一个时间点和一行简短说明
     - 加上节目中提到的所有链接（工具、书、文章、人名等等）

4. **社交媒体套件（Social Media Kit）**
   - 为不同平台自动生成宣传文案：
     - X / Twitter：3 条推文
       - 一条是高光金句（pull quote）
       - 一条是核心洞见
       - 一条是抛问题、引讨论
     - LinkedIn：1 条专业语气的长一点的帖子
     - Instagram：1 条更轻松、带 emoji 和话题标签的说明文案

5. **节目描述（Episode Description）**
   - 为 Spotify、Apple Podcasts、YouTube 等平台写一段：
     - 约 200 字的节目简介
     - 自然地包含 3–5 个相关关键词，利于搜索和发现

## 三、你需要具备的技能 / 能力

要跑通这条播客流水线，底层需要几个基础能力（可以是 OpenClaw 中的 Skills，也可以是你自己写的集成）：

- **Web 搜索 / 研究能力**
  - 用于嘉宾背景查询、话题深挖

- **文件系统读写能力**
  - 能从本地读取录音逐字稿
  - 能把调研结果、Show Notes、社交文案写入固定目录

- **IM / 协作工具集成**
  - 比如 Slack、Discord、Telegram 等
  - 用来把生成的内容推送给你本人或团队

- **可选：`sessions_spawn` 并行执行能力**
  - 可以让“调研 Agent”和“写作 Agent”并行工作，加速整体生产

- **可选：RSS 订阅与监控能力**
  - 用来监控竞品播客的 RSS Feed
  - 发现对标话题后，自动提醒你“是否要跟进一期回应或衍生内容”

## 四、如何配置这条播客流水线？

### 1. 录制前：生成调研与节目大纲

在录制前，你可以给 Agent 发送类似下面的系统指令（Prompt），让它根据选题和嘉宾做完调研，并产出完整的大纲：

```text
I'm recording a podcast episode about [TOPIC]. My guest is [NAME].

Please:
1. Research the guest — their background, recent work, hot takes, and
   anything controversial or interesting they've said publicly.
2. Research the topic — key trends, recent news, common misconceptions,
   and what the audience likely already knows vs. what would surprise them.
3. Generate an episode outline:
   - Cold open hook (1-2 sentences to grab attention)
   - Intro script (30 seconds, casual tone)
   - 5-7 interview questions, ordered from easy/rapport-building to deep/provocative
   - 2-3 "back pocket" questions in case the conversation stalls
   - Closing segment with call-to-action

Save everything to ~/podcast/episodes/[episode-number]/prep/
```

这段指令的效果是：

- 先对嘉宾和话题做全面调研
- 然后给出一套具备“节奏感”的提纲和提问顺序
- 最后把所有准备材料统一保存到：

```text
~/podcast/episodes/[期数]/prep/
```

这样你每次录制前，只要检查这个文件夹，就能快速进入状态。

### 2. 录制后：从逐字稿生成 Show Notes 和宣传内容

节目录完之后，你只需要把逐字稿（或文件路径）交给 Agent：

```text
Here's the transcript for Episode [NUMBER]: [paste or point to file]

Please:
1. Write timestamped show notes — every major topic shift gets a timestamp
   and one-line summary. Include links to anything mentioned (tools, books,
   articles, people).
2. Write an episode description (max 200 words) optimized for podcast
   search. Include 3-5 relevant keywords naturally.
3. Create social media posts:
   - X/Twitter: 3 tweets — one pull quote, one key insight, one question
     to spark discussion. Each under 280 chars.
   - LinkedIn: 1 post, professional tone, 100-150 words.
   - Instagram caption: 1 post with emoji, casual tone, include relevant hashtags.
4. Extract a "highlights" list — the 3 most interesting/surprising moments
   with timestamps.

Save everything to ~/podcast/episodes/[episode-number]/publish/
```

这一步完成后，你会在：

```text
~/podcast/episodes/[期数]/publish/
```

这个目录下拿到：

- 一份带时间戳、适合传给听众看的详细 Show Notes
- 一段可直接贴到播客平台的节目描述
- 一整套可复制粘贴到 X / LinkedIn / Instagram 的宣传文案
- 三个最精彩高光片段（含时间点），方便你后续做剪辑或二次分发

### 3. 可选：监控竞品播客 RSS Feed

如果你想做更系统的选题运营，可以加一个“竞品监控” Agent：

```text
Monitor these podcast RSS feeds daily:
- [feed URL 1]
- [feed URL 2]

When a new episode drops that covers a topic relevant to my podcast,
send me a Telegram message with:
- Episode title and link
- One-sentence summary
- Whether this is something I should respond to or cover from my angle
```

这个 Agent 每天扫描你关注的播客 RSS Feed，一旦发现：

- 有新节目在讨论你关心的话题
- 或者可能影响你受众认知的内容

就会自动在 Telegram（或其它 IM）里给你发一个简报，包含：

- 节目标题与链接
- 一句话摘要
- 以及一个建议：“你是否值得跟进一期，或者从自己的视角回应？”

## 五、关键洞见

作者在用这条播客流水线时，有几个重要的体会：

1. **前期调研的价值远超后期润色**
   - 带着扎实的嘉宾和话题研究走进录音室，对话质量会大幅提升
   - 这些是后期剪辑、文案再怎么打补丁都难以弥补的

2. **带时间戳的 Show Notes 是提升留存的关键**
   - 听众可以快速跳到自己感兴趣的段落
   - 很多播客跳过这一步，只因为它太耗时间、太机械
   - 让 Agent 来做，就能几乎“零额外成本”提升专业度

3. **社交媒体套件在“长期”节省的时间最多**
   - 每期节目都需要宣传
   - 格式和结构高度重复，是非常适合交给 Agent 执行的工作

4. **播客流水线可以与「多代理内容工厂」联动**
   - 这条流水线产出的 Show Notes、Highlights、逐字稿
   - 可以进一步交给多代理内容工厂，批量转成：
     - 博客文章
     - 邮件 Newsletter
     - 短视频脚本与字幕

## 六、相关链接

- [Podcast RSS Feed Spec](https://podcasters.apple.com/support/823-podcast-requirements)：苹果官方的播客 RSS 规范
- [Spotify for Podcasters](https://podcasters.spotify.com/)：Spotify 提供的创作者后台
- [Whisper (OpenAI)](https://github.com/openai/whisper)：可在本地运行的开源语音转录模型，用于生成逐字稿

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/podcast-production-pipeline.md
---