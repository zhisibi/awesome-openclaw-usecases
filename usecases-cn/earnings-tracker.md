每到财报季，想跟的公司一大堆：NVDA、MSFT、GOOGL、META、AMZN、TSLA、AMD……

- 你要记住每家公司的财报日期和时间
- 要盯日历，还要看新闻、看公告
- 财报发布后，还得翻新闻、研报、社交媒体，拼出“到底是利好还是利空”

这个用例展示的是：用 **OpenClaw + web_search + cron**，搭一个“**AI 财报跟踪助手（Earnings Tracker）**”，帮你：

- 每周自动扫一遍下周财报日历
- 发一条“下周这些 AI/科技公司要发财报”的预告到 Telegram
- 你选哪些公司要跟踪之后，自动为每家公司创建“一次性财报提醒任务”
- 财报一出，自动搜索结果、整理关键指标和 AI 相关亮点，总结发回给你

---

## 一、痛点：手动盯财报既枯燥又容易漏

传统做法一般是：

- 订阅财经网站或券商的“财报日历”
- 手动把关心的公司抄到自己的日历
- 财报当天自己去刷网页 / 刷推特 / 刷社群，找“核心信息”

问题在于：

1. **信息源分散**：日历一个、新闻一个、研报一个，来回切换很累
2. **容易错过**：如果那天很忙，甚至可能错过一个关键公司的财报
3. **重复体力活**：
   - 每周做一次“筛公司 + 记日期”
   - 每次财报后再做一次“浏览 n 篇文章，自己总结”的整理

这些步骤其实都非常适合交给 Agent 处理，尤其是：

- 规整结构化信息（时间、公司、指标）
- 汇总多篇新闻和研报，提取结论

---

## 二、AI Earnings Tracker 做了哪些事？

原用例里，这个工作流主要覆盖三块：

### 1. 每周日出“下周财报预告”

- 每周日晚上（比如 18:00），OpenClaw 自动：
  - 搜索下周的财报日历
  - 过滤出科技 / AI 相关公司
  - 在 Telegram 的 `earnings` 话题里，发一条“**下周值得关注的财报列表**”

你只要在这条消息下面简单回复：

> “帮我跟踪 NVDA、MSFT、META。”

### 2. 为每家公司创建“一次性提醒任务”

收到你的选择后，OpenClaw 会：

- 针对每家公司，在对应的财报发布时间点创建一个一次性 cron 任务
- 到点后自动触发一个工作流：
  - 搜索最新财报结果
  - 汇总关键信息

### 3. 财报发布后的智能总结

财报发布后，这个 Agent 会做几件事：

- 搜索并阅读多篇相关文章 / 公告
- 抽取：
  - 是否“超预期 / 低于预期”（beat / miss）
  - 核心财务指标：营收、EPS、同比增速
  - 管理层在电话会上的重点观点
  - 尤其是 **AI 相关业务的进展和指引**
- 最后把整理好的总结发到你的 Telegram `earnings` 话题：

大致类似这样：

```text
$NVDA 2026 Q1 财报总结

- 结果：收入 / EPS 均超预期
- 营收：xx 亿美元，同比 +xx%
- 数据中心业务：xx 亿美元，同比 +xx%，主要由 AI 加速卡需求驱动
- 管理层重点：
  - 强调 AI 计算需求“多年级别的强劲周期”
  - 提到与云厂商、车企的合作进展
- AI 相关亮点：
  - 新一代 GPU 出货情况
  - 云端推理 / 训练业务的增长
- 指引：下季度预期营收 xx 亿美元，高于市场预期 xx%
```

---

## 三、需要哪些能力？

原文把这个工作流抽象成了三个核心能力：

1. `web_search`：
   - 用于查找“下周财报日历”
   - 用于在财报发布后搜索新闻、公告、分析文章

2. cron / 调度能力：
   - 每周日固定跑一次“扫描 +预告”的任务
   - 每个公司有自己的“一次性财报任务”

3. Telegram 话题 / 频道：
   - 作为所有财报通知的统一入口
   - 方便你在一个地方就看到预告 + 结果

在 OpenClaw 里，这三块基本都是现成的：

- `web_search` / `web_fetch` 是内建工具
- 心跳任务 / 定时任务可以在 `HEARTBEAT.md` 里声明，或用 cron skill
- Telegram 集成只需要配置好 Bot Token 和 chat_id

---

## 四、如何在 OpenClaw 里搭这套 Earnings Tracker？

可以用一段类似下面的 Prompt 作为起点：

```text
Every Sunday at 6 PM, run a cron job to:
1. Search for the upcoming week's earnings calendar for tech and AI companies
2. Filter for companies I care about (NVDA, MSFT, GOOGL, META, AMZN, TSLA, AMD, etc.)
3. Post the list to my Telegram "earnings" topic
4. Wait for me to confirm which ones I want to track

When I reply with which companies to track:
1. Schedule one-shot cron jobs for each earnings date/time
2. After each report drops, search for earnings results
3. Format a summary including: beat/miss, revenue, EPS, key metrics, AI-related highlights, guidance
4. Post to Telegram "earnings" topic

Keep a memory of which companies I typically track so you can auto-suggest them each week.
```

如果用中文，可以类似这样表达：

```text
以后每周日晚上 6 点：
1. 自动搜索下周的财报日历，关注科技 / AI 相关公司；
2. 按我常看的公司白名单筛一遍（NVDA、MSFT、GOOGL、META、AMZN、TSLA、AMD 等）；
3. 在 Telegram 的「earnings」话题里发一条消息，把下周值得关注的公司列出来；
4. 等我回复要跟踪哪些公司。

当我回复要跟踪的公司名单后：
1. 为每家公司在财报发布时间创建一次性 cron 任务；
2. 财报发布后自动搜索最新结果；
3. 用「是否超预期、营收、EPS、关键 AI 业务亮点、指引」这些维度帮我写一个总结；
4. 把总结发到 Telegram 的「earnings」话题。

长期记忆我常跟踪的公司，下次自动优先推荐这些公司。
```

---

## 五、实战建议：从“少公司、少指标”开始

为了不要一开始就把工作流搞得太复杂，建议：

1. **先选 3–5 家最核心的公司**：比如 NVDA、MSFT、GOOGL
2. 初期先输出“**文本总结**”，不急着做可视化表格 / Dashboard
3. 一次财报季跑下来，观察：
   - 哪些信息是真正有用的
   - 哪些信息其实每次都不会看

之后再迭代：

- 对真正重要的指标加专门的“告警”（例如 EPS 大幅低于预期时提醒你减仓）
- 加一个“季度回顾”模式，让 Agent 汇总整季财报

---

## 六、这个工作流的价值在哪里？

对个人投资者来说，这个 AI Earnings Tracker 的价值主要体现在：

- 把「查日历 + 记日期 + 建提醒」这些纯体力活全部自动化
- 把「翻十几篇新闻 / 研报、自己总结」交给 Agent 做初筛
- 你只需要在 Telegram 里看“预告 + 总结”，再决定要不要深入阅读原文

它不会替你做投资决策，但能极大减少你在信息收集上的时间消耗，让你把精力放在“理解商业和行业”上。

---

## 原文链接

- [AI-Powered Earnings Tracker](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/earnings-tracker.md)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/earnings-tracker.md
---
