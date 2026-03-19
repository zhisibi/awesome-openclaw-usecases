每天刷推特、读文章、看视频，最大的痛点是：

> 「上次看到一篇讲 Agent 记忆的文章很好，但我完全找不回来了。」

浏览器书签、稍后阅读工具、收藏夹……用久了往往都会变成：

- 一堆「看起来很有价值，但再也没打开过」的链接
- 靠标题搜索很难找到特定一句话或某个观点

这个用例的目标是：

> 用 OpenClaw 搭一个**个人知识库（RAG）**，把你平时丢给它的 URL / 文本都吃进去，之后可以直接问：
>
> “我之前存过哪些关于 agent memory 的文章？”

---

## 一、核心能力：语义搜索 + 多源内容抓取

原文里总结了这个 workflow 的三大功能：

1. **统一入口**：
   - 在 Telegram 话题或 Slack 频道中，随手丢各种内容：
     - 文章链接
     - 推特 / 线程
     - YouTube 视频
     - PDF 报告

2. **自动抓取 + 入库**：
   - 用 `web_fetch` 把网页正文、推特内容、视频字幕抓下来
   - 加上元数据（标题、URL、日期、类型）存进知识库

3. **语义搜索（RAG）**：
   - 用自然语言提问：
     - “我存过哪些关于 LLM 记忆架构的资料？”
     - “给我找几篇讲 RAG 评估方法的内容。”
   - 返回相关片段 + 来源链接

此外，这个知识库还能被其它工作流复用：

- 做视频创意时，自动去 KB 里找相关素材
- 写文章 / 做研究时，优先从自己看过的内容里检索

---

## 二、依赖的技能

原文建议：

- 使用现成的 **`knowledge-base` skill**（ClawHub 上有）
- 或者自己搭：
  - embeddings 向量存储
  - 一个简单的检索接口

基础能力包括：

- `web_fetch`（OpenClaw 内置）
- Telegram / Slack 集成
- 本地文件或数据库存储

---

## 三、如何在 OpenClaw 中配置？

原文给了一个简单直接的 Prompt：

```text
When I drop a URL in the "knowledge-base" topic:
1. Fetch the content (article, tweet, YouTube transcript, PDF)
2. Ingest it into the knowledge base with metadata (title, URL, date, type)
3. Reply with confirmation: what was ingested and chunk count

When I ask a question in this topic:
1. Search the knowledge base semantically
2. Return top results with sources and relevant excerpts
3. If no good matches, tell me

Also: when other workflows need research (e.g., video ideas, meeting prep), automatically query the knowledge base for relevant saved content.
```

翻成中文就是：

- 在 `knowledge-base` 话题中：
  - 收到 URL 时自动抓取内容 + 入库
  - 回一条“小结 + 分片数量”的确认信息
- 在同一话题中提问时：
  - 先查知识库的语义相似内容
  - 给你返回若干条结果 + 摘要 + 原文链接
- 其它工作流需要研究资料时，也优先来这里查一遍

---

## 四、知识库的结构建议

你可以按下面的结构组织：

- `kb/` 目录下：
  - `metadata.jsonl`：每条记录一行，包含：
    - `id`
    - `title`
    - `url`
    - `source_type`（article / tweet / video / pdf）
    - `saved_at`
  - `chunks/` 目录：按 `id` 存储分段后的文本
  - `embeddings/` 目录：存储向量（可以是 `.npy` 或数据库）

Agent 的工作：

1. 对新内容做分段（根据句子 / 段落 / token 数）
2. 求 embeddings 并存储
3. 建立倒排索引 / 向量索引

查询时：

1. 对你的问题做 embedding
2. 在向量空间里求最近邻
3. 把最相关的若干 chunk 按相似度排序返回

---

## 五、实践中的小技巧

1. **只往里放你「认真看过」的内容**：
   - 不要把它当成新的“稍后再看”垃圾场
   - 只在你觉得“这篇值得收藏以后用”的时候丢给 Agent

2. **适当加标签 / 主题词**：
   - 在发 URL 时补一句：`标签：RAG, memory, evaluation`
   - Agent 可以把这些标签写进 metadata，检索时更好用

3. **定期做「知识库回顾」**：
   - 例如每周让 Agent 输出：
     - 本周新增了哪些主题
     - 哪些内容被多次引用或很值得二刷

4. **和写作 / 创作者工作流打通**：
   - 在写博客 / 做视频脚本时，让 Agent 先从 KB 里找相关材料
   - 这样输出会自带“你自己的知识体系”味道

---

## 六、这个个人知识库有何价值？

跟「直接上网搜索」相比，一个只包含**你自己筛选过内容**的知识库，最大的区别是：

- 噪音少：没有 SEO 垃圾和重复信息
- 更贴近你的理解路径：
  - 同一概念你可能已经看过多次不同表述
  - Agent 可以基于这些内容给出更符合你口味的解释

长期来看，它会变成一个：

> 既存储「你看过什么」，又体现「你是怎样思考这些东西」的第二大脑。

---

## 原文链接

- [Personal Knowledge Base (RAG)](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/knowledge-base-rag.md)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/knowledge-base-rag.md
---
