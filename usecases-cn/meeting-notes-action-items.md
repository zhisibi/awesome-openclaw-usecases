你可能在做 会议纪要 & 行动项助手 相关的事情时，经常会遇到这样的问题：开会时大家聊得很爽，散会后却没人记得谁要做什么、什么时候完成，会议纪要也常常拖延甚至缺席。

这个 OpenClaw 用例 **“会议纪要 & 行动项助手”** 的目标是：让 OpenClaw 接管会议记录，从原始文字/录音中自动生成结构化会议纪要、行动项列表、负责人和截止日期，并同步到任务系统。

## 用例能做什么

- 把零散的信息和对话整理成结构化结果，减少人工重复
- 自动生成会议纪要、行动项和负责人清单
- 通过集成任务系统（如 Todoist、Notion、Jira 等）落地执行
- 支持持续追加更新，形成可追踪的会议决策历史

## 典型使用场景

- 产品评审会后，需要把决策、反对意见和后续行动整理下来
- 跨部门周会，希望形成统一可追踪的行动项清单
- 一对一反馈会后，把双方承诺的改进点记录并定期跟进

## 工作流设计要点

1. 会前配置好会议主题、参与人和记录方式（文字或录音转写）。
2. 会后把原始记录交给 OpenClaw，让它自动提取议题、结论和未决问题。
3. 识别出所有行动项，补全负责人、截止日期和关联项目。
4. 通过集成把行动项同步到你的任务系统，并定期提醒跟进。

## 在 OpenClaw 中如何落地

- 参考 awesome-openclaw-usecases 仓库中的原始用例配置和工作流脚本。
- 本文对应的英文用例文件：`usecases/meeting-notes-action-items.md`。
- 你可以在此基础上替换成自己的 API Key、数据源和下游系统（如 Notion、Todoist、自建 CRM 等）。

## 可拓展的改造思路

- 接入公司常用的视频会议工具（如 Zoom、Google Meet）自动抓取会议记录。
- 和 OKR/项目管理系统对接，让关键会议结论直接落到目标和里程碑上。
- 给不同会议类型（周会、项目会、一对一）定制不同的纪要模版。

## 原始用例链接

- GitHub 仓库：[https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/meeting-notes-action-items.md](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/meeting-notes-action-items.md)

---
本文由小龙虾博客助手整理翻译。

原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/meeting-notes-action-items.md
---
