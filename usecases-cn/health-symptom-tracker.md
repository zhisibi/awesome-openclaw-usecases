很多人都有类似的困扰：

- 吃完某些食物偶尔会不舒服，但又说不清到底是哪一种
- 怀疑自己对某些成分（乳糖、麸质、坚果等）不耐受，却没有长期数据
- 医生让你「记饮食日记 + 症状」，坚持两天就放弃了

这个用例是用 **OpenClaw + Telegram 话题 + 简单文件日志**，搭一个「**健康与症状追踪器**」，帮助你：

- 每天用自然语言随手发消息记录吃了什么、感觉如何
- Agent 自动解析并写入一份时间序列日志
- 每周自动分析哪些食物和不适症状存在明显相关性

---

## 一、痛点：长期记录太难，导致永远“靠感觉”

如果平时只是在脑子里记：

> “我好像吃了某某之后肚子有点不舒服……”

几乎得不到什么可靠结论：

- 记忆不准确：你能记住上周二午饭的细节吗？
- 没有系统化记录：时间、食物、症状都混在一起
- 很难做交叉分析：
  - 比如「在晚上吃乳制品 + 第二天早上肚子痛」这种跨时间关联

医生通常会建议「坚持记饮食和症状日记」，但纯手工记录太枯燥：

- 每次要打开 App 或记事本
- 还要自己规范格式
- 一旦生活变忙，几天不记就彻底断档

而这个用例的核心思路就是：

> **你只要像平时发消息一样说话，其它的让 Agent 来做。**

---

## 二、整体方案：Telegram 话题 + markdown 日志

原用例的设计非常简单实用：

1. 创建一个专门的 Telegram 话题，比如叫 `health-tracker`
2. 在本地创建一份日志文件：`~/clawd/memory/health-log.md`
3. 对 OpenClaw 下指令：

```text
When I message in the "health-tracker" topic:
1. Parse the message for food items and symptoms
2. Log to ~/clawd/memory/health-log.md with timestamp
3. Confirm what was logged

Set up 3 daily reminders:
- 8 AM: "🍳 Log your breakfast"
- 1 PM: "🥗 Log your lunch"
- 7 PM: "🍽️ Log your dinner and any symptoms"

Every Sunday, analyze the past week's log and identify patterns:
- Which foods correlate with symptoms?
- Are there time-of-day patterns?
- Any clear triggers?

Post the analysis to the health-tracker topic.
```

翻译成中文大致是：

- 在 `health-tracker` 话题中：
  - 每条消息解析出「吃了什么」「出现了什么症状」
  - 带时间戳写入 `health-log.md`
  - 回一条确认信息
- 每天 3 个固定时间，用 cron 或 heartbeat 提醒你记录三餐和症状
- 每周日自动分析过去一周的数据：
  - 哪些食物和症状一起出现得最多
  - 是否存在特定时间段更容易出问题

---

## 三、日志文件大致长什么样？

你可以用 markdown 做一个简单但结构化的格式，例如：

```markdown
## 2026-03-19 08:10
Food: 燕麦粥、牛奶、香蕉
Symptoms: 轻微胀气

## 2026-03-19 13:05
Food: 米饭、清蒸鱼、炒青菜
Symptoms: 无

## 2026-03-19 19:40
Food: 披萨（奶酪）、啤酒
Symptoms: 夜间轻微腹泻，睡前有反酸
```

Agent 负责：

- 把自由文本归纳成类似上面的结构
- 自动带上时间戳
- 可选：加上一些标签，比如 `#dairy`、`#spicy`、`#alcohol`

---

## 四、如何做“找触发因素”的分析？

每周分析时，Agent 可以：

1. 按症状聚合：
   - 统计「腹痛」「腹泻」「皮疹」「头痛」分别出现了多少次
   - 对每类症状，列出过去 1–2 小时内最常一起出现的食物

2. 按食物聚合：
   - 对特定食物或类别（乳制品、小麦、酒精、辛辣）统计：
     - 出现时的症状概率
     - 不出现时的症状概率
   - 粗略给出「相对风险提升」的提示

3. 按时间段分析：
   - 例如：晚餐吃高油脂 + 夜间反酸
   - 午餐吃太少 + 下午低血糖不适

最后输出一份「**并非医学诊断，但有助于你与医生讨论的观察报告**」，例如：

- 过去 4 周里，在吃含乳制品的餐后 12 小时内出现腹泻的比例为 X%
- 有 5 次在晚上 10 点后进食高脂肪 + 夜间反酸的记录

---

## 五、在 OpenClaw 中怎么落地？

你需要的能力其实很基础：

- Telegram 话题集成
- 文件读写（写入 `health-log.md`）
- 定时任务（可以用 cron skill 或 HEARTBEAT）

典型 HEARTBEAT 配置思路：

- 每天 08:00、13:00、19:00：
  - 调用 Telegram Bot API 给 `health-tracker` 话题发一条提醒
- 每周日 20:00：
  - 读取 `health-log.md`
  - 做一轮简易统计 + 相关性分析
  - 把结论发到话题里

---

## 六、给自己的使用建议

- **尽量用自然语言，说人话就好**：
  - “刚吃完火锅，有点撑，胃不舒服。”
  - 让 Agent 去识别关键词和食物
- 不要追求一开始就完美结构化，可以先靠 LLM 清洗文本
- 持续几周之后，再考虑：
  - 是否要增加更细致的标签
  - 是否要导出到表格 / 可视化工具

真正的价值在于：

> 把“长期、枯燥、容易放弃”的记录工作交给 Agent，你只需要**按时说几句真话**。

---

## 原文链接

- [Health & Symptom Tracker](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/health-symptom-tracker.md)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/health-symptom-tracker.md
---
