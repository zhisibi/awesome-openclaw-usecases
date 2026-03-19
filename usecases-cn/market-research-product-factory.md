你可能在做 市场调研 & 产品工厂 相关的事情时，经常会遇到这样的问题：想做一个新产品，却不知道用户真正需要什么；或者已经有业务，但对用户最近在吐槽什么、期待什么缺乏系统认知。

这个 OpenClaw 用例 **“市场调研 & 产品工厂”** 的目标是：用 OpenClaw 持续爬取 Reddit 和 X 等平台最近 30 天的讨论，自动挖掘高频痛点，并把这些痛点转化为结构化的产品机会、功能列表和 MVP 实验计划。

## 用例能做什么

- 基于真实用户讨论挖掘需求，而不是拍脑袋想功能
- 基于工作流自动化处理重复性高的操作
- 减少手工搬运信息的时间，把人力留给决策和创造

## 典型使用场景

- 独立开发者想验证一个新产品方向是否值得做
- 小团队准备立项，下决心前希望先从真实用户讨论里找佐证
- 已有 SaaS/内容产品，想系统收集“最近一个月用户都在抱怨什么”

## 工作流设计要点

1. 通过 Last 30 Days 等技能从 Reddit/X 抓取最近 30 天与目标主题相关的讨论。
2. 用多轮分析把原始帖子聚类、提炼成高频痛点和机会点。
3. 针对每个痛点，让 OpenClaw 生成可能的产品形态、功能列表和 MVP 验证方案。
4. 把结果整理成可执行的产品路线图或 backlog，方便后续进入实现阶段。

## 在 OpenClaw 中如何落地

- 参考 awesome-openclaw-usecases 仓库中的原始用例配置和工作流脚本。
- 本文对应的英文用例文件：`usecases/market-research-product-factory.md`。
- 你可以在此基础上替换成自己的 API Key、数据源和下游系统（如 Notion、Todoist、自建 CRM 等）。

## 可拓展的改造思路

- 把 Reddit/X 换成你关心的垂直社区或中文社区数据源。
- 把输出从“文档报告”升级为“自动生成问卷/落地页文案”，直接进入用户验证阶段。
- 结合 A/B 测试工具，基于这些痛点快速验证多个产品假设。

## 原始用例链接

- GitHub 仓库：[https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/market-research-product-factory.md](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/market-research-product-factory.md)

---
本文由小龙虾博客助手整理翻译。

原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/market-research-product-factory.md
---
