刷 Reddit 很容易变成“时间黑洞”：

- 打开首页，先被推荐一堆和你没那么相关的内容
- 点进某个热门帖，评论区一刷就是半小时
- 想系统地跟几个 Sub 的高质量讨论，却总是漏掉好帖

这篇用例介绍的是：用 OpenClaw + 只读 Reddit Skill 做一个「每日 Reddit 精选 Digest」，帮你：

- 每天定时扫描你关心的几个 Subreddit
- 把当天表现最好的帖子挑出来
- 按你的喜好过滤掉垃圾内容（比如不看 Meme）
- 整理成一份简短 Digest 发给你

重点是：**完全只读、不负责发帖 / 点赞 / 评论。**

---

## 一、痛点：时间都浪费在“刷”的过程里

你真正想要的通常只有：

- 这几个 Sub 里，今天有什么值得一看的东西？
- 有哪些讨论和我的工作 / 兴趣强相关？

但实际花的时间却主要耗在：

- 在各个 Sub 之间切换
- 被无关的热门内容吸走注意力
- 手动标记“这个不错，回头再看”，结果根本回不来

如果用 OpenClaw 帮你“扫 + 选 +记偏好”，你每天只需要花几分钟看 Digest 就足够。

---

## 二、这个每日 Reddit Digest 能做什么？

用这个用例里的方案，你可以让 OpenClaw：

1. **定时扫一批 Subreddit**  
   - 比如：`r/MachineLearning`、`r/dataisbeautiful`、`r/startups` 等
   - 每天在固定时间（例如下午 5 点）
   - 分别获取：
     - 热门（hot）
     - 最新（new）
     - Top（如当天 / 本周）帖子

2. **按主题搜索 + 按表现排序**  
   - 除了按 Sub 扫，还可以按关键词搜索帖子（搜索特定话题的讨论）
   - 然后按：
     - 点赞数
     - 评论数
     - 创建时间
   - 排出一个优先列表

3. **拉取评论线程做上下文**  
   - 对于特别感兴趣的帖子，可以顺便拉主要评论线程：
     - 看不同观点
     - 看总结 / TL;DR
     - 看有价值的链接和引用

4. **维护一个“偏好记忆”**  
   - Digest 里会问你：

     > 今天这份列表你满意吗？哪些你喜欢？哪些不喜欢？

   - 你的反馈会被记录成规则，比如：
     - 不要包含纯 Meme / 图梗
     - 多给我技术实现 / 教程型内容
     - 避免重复看到同一类水帖

5. **每天发给你一份简短 Digest**  
   - 格式类似：

   ```text
   [r/MachineLearning]
   - [Post Title 1] (Score 1.2k, 250 comments)
     Why it might matter to you: ...
   - [Post Title 2] (...)

   [r/startups]
   - [...]
   ```

   - 你可以选择：
     - 直接点链接过去看
     - 或让 OpenClaw再帮你总结某个帖子的重点和评论分歧

---

## 三、需要的技能：reddit-readonly

这个用例使用了一个开箱即用的 Skill：[reddit-readonly](https://clawhub.ai/buksan1950/reddit-readonly)。特点：

- **只读**：
  - 不能发帖
  - 不能投票
  - 也不能评论
- 不需要认证：
  - 无需登录 Reddit 账号
  - 直接使用公开接口

安装方式（按原文）：

- 使用 ClawHub 安装 `reddit-readonly` skill 即可

---

## 四、如何在 OpenClaw 里配置？

原用例给了一个非常简洁的提示模版，你可以直接用：

```text
I want you to give me the top performing posts from the following subreddits.
<paste the list here>
Create a separate memory for the reddit processes, about the type of posts I like to see and every day ask me if I liked the list you provided. Save my preference as rules in the memory to use for a better digest curation. (e.g. do not include memes.)
Every day at 5pm, run this process and give me the digest.
```

翻成中文大意：

> 我想让你从下面这些 Subreddit 里抓出表现最好的帖子：
> 
> - （把列表贴在这里）
> 
> 为 Reddit Digest 单独建一块记忆，用来记录我喜欢看的帖子类型。每天给完今日清单后，问我“这份列表是否满意”，并把我的反馈保存成规则（例如：不要包含 Meme）。
> 
> 每天下午 5 点，自动跑一遍这个流程，把 Digest 发给我。

配置后，OpenClaw 会：

- 使用 `reddit-readonly` 按你给的 Sub 列表抓数据
- 每天 5 点执行一次
- 在对话里发给你当日精选，并等待你的反馈

---

## 五、实践建议

1. **一开始 Sub 列表不要太多**  
   - 先从 3–5 个核心 Sub 开始
   - 过一段时间再慢慢加

2. **明确写出“不想看什么”**  
   - 比如：
     - 不要 Meme
     - 不要纯情绪发泄 / rant
     - 不要太水的“无图无内容”的贴
   - 这些会极大提升 Digest 的含金量

3. **把“看完再处理”变成“处理完再看”**  
   - 可以让 OpenClaw 帮你：
     - 直接从评论里抽取实用链接
     - 总结支持 / 反对的主要观点
   - 你只需要花时间看有价值的结论，而不是从 0 开始扒评论

4. **长期积累你的 Reddit 偏好记忆**  
   - 每天用一句话回应“今天这份如何”
   - 一两周后，Digest 会越来越像“你自己的 Reddit 首页”，而不是算法推荐的泛泛信息流

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/daily-reddit-digest.md
---
