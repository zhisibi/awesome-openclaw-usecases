订阅了一堆 YouTube 频道，却总觉得：

- 真想看的视频经常没在首页出现
- 通知中心要么刷屏，要么关键视频完全没有提醒
- 想跟进某个话题的新视频，结果不是忘记就是被算法喂别的东西

这篇用例介绍的是：用 OpenClaw 做一条「每日 YouTube 精选」流水线，帮你：

- 每天固定时间，检查你关注的频道有没有新视频
- 拉取视频字幕 / 文本，生成要点摘要
- 把这些摘要整理成一份 Digest 发给你
- 也可以按关键词跟踪某个话题的新视频

你每天只需要花几分钟扫一眼，就能知道有哪些值得看的视频，以及每个视频讲了什么。

---

## 一、痛点：推荐算法和通知都“不可靠”

YouTube 的推荐和通知，有几个典型问题：

- 首页推荐更多是“平台想让你看什么”，而不是“你真正关心什么”
- 通知中心容易淹没在各种直播 / 评论 / 社区贴里
- 想跟进某个细分主题（比如“OpenClaw 用例”“AI Agent 案例”）时，没有特别好的原生订阅方式

你真正需要的是：

- 一份只包含“你主动指定的频道 / 话题”的每日摘要
- 每个视频都有简洁要点，而不是你点进去看完才发现“浪费时间”

---

## 二、这个每日 YouTube Digest 能做什么？

用这个用例里的方案，你可以让 OpenClaw：

1. **按频道拉新视频**  
   - 比如你关注：
     - `@TED`
     - `@Fireship`
     - `@ThePrimeTimeagen`
     - `@lexfridman`
   - 每天早上 8 点：
     - 检查这些频道过去 24–48 小时的新视频
     - 跳过太旧或早就处理过的内容

2. **拉取字幕并生成摘要**  
   - 对每个新视频：
     - 调用字幕 / transcript 接口拿到文本
     - 用 2–3 个要点总结主要内容
     - 列出视频标题、频道名和链接

3. **根据关键词跟踪话题**  
   - 比如：
     - “OpenClaw”
     - “Claude Code”
     - “AI agents”
   - 每天搜索 YouTube 上关于这些关键词的最新视频
   - 只对“没处理过的视频 ID”抓字幕、做摘要

4. **避免重复处理**  
   - 用一个简单文件（例如 `seen-videos.txt`）记录已处理的视频 ID：

   ```text
   seen-videos.txt:
   - dQw4w9WgXcQ
   - AbCdEf12345
   ```

   - 每次 Digest 生成时：
     - 先过滤掉已经在这个名单里的 ID
     - 对真正新的视频才花费 API 调用和摘要算力

5. **以 Digest 的形式发给你**  
   - 格式类似：

   ```text
   [频道 Digest]
   - [Channel] [Title]
     - 要点 1
     - 要点 2
     - 要点 3
     Link: ...

   [关键词 Digest]
   - ["OpenClaw" 新视频]
     - ...
   ```

   - 你可以直接根据摘要决定：
     - 哪些值得完整看一遍
     - 哪些只看摘要就够

---

## 三、需要的技能：youtube-full

用例里推荐使用一个专用 skill：[youtube-full](https://clawhub.ai/therohitdas/youtube-full)。这个 skill 的特点：

- 通过 TranscriptAPI.com 获取字幕 / 文本
- 对 Agent 友好：返回结构化 JSON
- 不需要手动安装二进制工具（不像 `yt-dlp` 那样依赖本地环境）

安装方式：

- 让 OpenClaw 自己装：

  ```text
  "Install the youtube-full skill and set it up for me"
  ```

- 或者用命令：

  ```bash
  npx clawhub@latest install youtube-full
  ```

安装后：

- skill 会引导你完成账号创建和 API Key 配置
- 提供 100 个免费调用额度进行体验
- API Key 会根据系统环境自动存放在合适的位置

对比传统 `yt-dlp` 一类工具的优势：

| 传统 CLI（yt-dlp 等）         | youtube-full / TranscriptAPI        |
|-------------------------------|-------------------------------------|
| 日志冗长，容易淹没 Agent 上下文 | 返回干净的 JSON 结果                |
| 云端环境 / GCP 上不好用        | 只走 HTTP，在哪都能用               |
| 容易被 YouTube 封 /限流        | 为大规模服务设计，有缓存             |
| 需要安装外部二进制             | 无需本地二进制，完全走 API          |

---

## 四、如何在 OpenClaw 里配置？

原用例给了两个典型配置方式。

### 1. 按频道做每日 Digest

可以给 OpenClaw 这样的提示：

```text
Every morning at 8am, fetch the latest videos from these YouTube channels and give me a digest with key insights from each:

- @TED
- @Fireship
- @ThePrimeTimeagen
- @lexfridman

For each new video (uploaded in the last 24-48 hours):
1. Get the transcript
2. Summarize the main points in 2-3 bullets
3. Include the video title, channel name, and link

If a channel handle doesn't resolve, search for it and find the correct one.
Save my channel list to memory so I can add/remove channels later.
```

大意就是：

> 每天早上 8 点，从上面这些 YouTube 频道抓最新视频，为每个视频生成 2–3 条要点摘要，并给出标题、频道名和链接。如果频道 handle 解析失败，就先搜一下找到正确频道。把频道列表存到记忆里，方便我随时增删。

### 2. 按关键词做话题追踪

如果你关心的是特定话题，可以用类似的指令：

```text
Every day, search YouTube for new videos about "OpenClaw" (or "Claude Code", "AI agents", etc).

Maintain a file called seen-videos.txt with video IDs you've already processed.
Only fetch transcripts for videos NOT in that file.
After processing, add the video ID to seen-videos.txt.

For each new video:
1. Get the transcript
2. Give me a 3-bullet summary
3. Note anything relevant to my work

Run this every morning at 9am.
```

这样就变成了一条**关键词监控流水线**：

- 持续搜某个话题的新视频
- 只处理没见过的 ID
- 聚焦跟你的工作 / 项目有关系的部分

---

## 五、注意事项和小建议

1. **频道 Digest + 关键词 Digest 可以同时存在**  
   - 一条针对“我关注的创作者”
   - 一条针对“我关注的话题”

2. **善用“0 成本查询 + 按需付费”的模式**  
   - `channel/latest` 和 `channel/resolve` 这类 API 是免费的
   - 只有真正拉字幕时才消耗额度
   - 所以多做一次“是否需要拉字幕”的筛选，是很划算的

3. **可以让 Agent 生成人性化的摘要风格**  
   - 比如：
     - 给出“是否值得我完整看一遍”的建议
     - 标记“非常适合剪成短视频”的时间点
     - 给出“对我现有项目是否有启发”的一句话点评

4. **把 Digest 和日常计划结合起来**  
   - 你可以把 YouTube Digest 和前面提到的“晨报”串在一起：
     - 早报的一部分就是“今天值得看的视频 + 要点”

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/daily-youtube-digest.md
---
