很多人的早晨是这样开始的：

- 刷新闻，看一堆和自己无关的东西
- 打开待办工具，看哪一堆任务最刺眼
- 翻日历、翻邮件、翻聊天记录，试图拼出“今天要干啥”

这篇用例介绍的是一条「自定义晨报」工作流：

- 每天固定时间（比如早上 8 点）
- OpenClaw 自动帮你：
  - 浏览与你兴趣相关的新闻
  - 查看你的任务清单
  - 生成当天的重点任务和建议
  - 顺便帮你在夜里写好一些文案 / 提案 / 草稿
- 然后把这一切整理成一份清晰的晨间报告，用 Telegram / Discord / iMessage 发给你

你起床后，打开手机，直接看到：**“今天的世界、你的任务和 AI 可帮你做的事”**。

---

## 一、痛点：早晨黄金时间被“对齐信息”消耗掉

绝大多数人的早晨，都在做一个低效的动作：

> 把自己和这个世界“对齐”。

比如：

- 看新闻：今天发生了什么？哪些和我相关？
- 看待办：今天有什么硬截止？什么可以推？
- 看聊天 / 邮件：昨晚别人给我留下了哪些“球”？

这些工作：

- 重复、机械，却不得不做
- 很烧时间和意志力
- 但真正有价值的是**对重点的选择和决策**，而不是信息搬运本身

如果把这些工作交给 OpenClaw 去做，你早晨能直接跳到：

- “今天最重要的三件事是什么？”
- “有什么 AI 可以替你先干掉？”

---

## 二、这个自定义晨报包含什么？

用这个用例里的配置，晨报会包含几个核心部分：

1. **个性化新闻简报**  
   - 基于你的兴趣关键词（比如 AI、初创公司、某些行业）
   - 自动浏览一轮夜间新闻和社交媒体趋势
   - 给出 3–5 条“真的与你有关”的故事

2. **任务清单精简版**  
   - 检查你的任务工具（如 Todoist、Apple Reminders、Asana 等）
   - 筛出：
     - 今天的硬截止任务
     - 近期到期但你还没动手的任务
   - 按优先级和紧急度排一个小清单

3. **内容 / 创意输出**  
   - 利用夜里闲置的算力：
     - 帮你写好一两份邮件草稿
     - 生成一个短视频 / 博客 / Newsletter 的初稿
     - 提一两条今天可以实验的创意 / 方案

4. **AI 自主可做任务推荐**  
   - 晨报中会有一段：

   > “我可以今天帮你做这些事情：……要我现在开始吗？”

   - 例如：
     - 整理昨天的会议记录
     - 给客户 /同事写 follow-up 邮件
     - 把某个主题的调研做完

5. **可选：天气 / 日程 / 心情小提醒**  
   - 你可以要求加上：
     - 所在城市天气简报
     - 今日 3 个最重要的日程
     - 一句你喜欢风格的“打气话”

---

## 三、需要哪些集成？

为了让这个晨报真正好用，你至少需要：

- 一个消息通道：
  - Telegram / Discord / iMessage / 飞书 等
- 一个任务管理工具：
  - Todoist / Apple Reminders / Asana / Things / Notion 等
- （可选）一个新闻 / 社交趋势研究工具：
  - 比如 [x-research-v2](https://clawhub.ai) skill，用于 X / Twitter 趋势抓取

这些集成可以通过：

- OpenClaw 的技能（Skills）
- 或你已有的 HTTP / API 工具

来打通。

---

## 四、如何在 OpenClaw 里配置这条晨报？

原用例给了一个很直接的 Prompt 模版，你可以稍作修改后直接用：

```text
I want to set up a regular morning brief. Every morning at 8:00 AM,
send me a report through Telegram.

I want this report to include:
1. News stories relevant to my interests (AI, startups, tech)
2. Ideas for content I can create today
3. Tasks I need to complete today (pull from my to-do list)
4. Recommendations for tasks you can complete for me today

For the content ideas, write full draft scripts/outlines — not just titles.
```

翻成中文大意就是：

> 每天早上 8 点，通过 Telegram 发给我一份日报，包括：
> 
> 1. 和我兴趣相关的新闻
> 2. 我今天可以创作的内容想法
> 3. 我今天必须完成的任务（从待办工具获取）
> 4. 你今天可以替我完成的任务建议
> 
> 对于内容想法，不要只有标题，要直接给出完整的大纲或脚本草稿。

配置完成后，OpenClaw 会：

- 自动在后台创建一个定时任务（cron / scheduler）
- 每天固定时间触发一次
- 按你定义的结构组装日报，然后发你选定的消息渠道

---

## 五、如何迭代你的“晨报模版”？

用久了之后，可以直接跟 Agent 说：

- “把早报里加上天气预报”
- “以后不要再发泛泛的新闻，只要跟 AI / 初创相关的”
- “每天加一句激励自己的话，语气像 XX 这种风格”

这些自然语言调整，都会反映到它内部维护的“晨报模版”里。

你甚至可以交给它一个完全空白的指令：

```text
I want this report to include things relevant to me.
Think of what would be most helpful to put in this report.
```

让它自己根据你过往的使用习惯，想“什么对你最有用”。

---

## 六、关键洞见

这个用例最有价值的几点在于：

1. **报表不是目的，行动才是**  
   晨报不是多一个“信息流”，而是帮你在醒来时就看到：
   - 今天最该做什么
   - 有哪些事可以交给 AI 去做

2. **夜间算力不要浪费**  
   - 把“写初稿、列大纲、生成模板”这类工作丢给夜间 Agent
   - 你早上只需要修改 / 挑选，而不是从零开始

3. **AI 主动提建议很关键**  
   - “你今天可以做 X / Y / Z”这一段，本质是在训练 Agent 主动思考“能帮你做什么”，而不是被动等指令

4. **迭代模版=迭代你的晨间习惯**  
   - 每次你觉得晨报哪里不对劲，就直接用自然语言改需求
   - 一段时间之后，你会得到一个高度贴合自己节奏的晨报系统

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/custom-morning-brief.md
---