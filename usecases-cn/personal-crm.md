你可能在做 个人 CRM 助手 相关的事情时，经常会遇到这样的问题：认识了很多人，却很少系统维护：错过问候节点、忘记回复消息、重要联系人久未联系。

这个 OpenClaw 用例 **“个人 CRM 助手”** 的目标是：用 OpenClaw 聚合通讯录、邮件、消息记录，建立一个“个人关系数据库”，并智能提醒你在合适的时间与合适的人联系。

## 用例能做什么

- 把通讯录、邮件、聊天记录整合成结构化“人物档案”
- 自动识别长期未联系的人，并给出合适的联系理由建议
- 在重要日期前，提前提醒你发送问候或安排会面

## 典型使用场景

- 自由职业者希望维护好核心客户和老客户
- 管理者需要长期经营团队成员、伙伴和候选人网络
- 普通人希望在重要节日和节点不再“想起谁却找不到联系方式”

## 工作流设计要点

1. 从邮箱、通讯录、聊天记录等数据源中提取联系人和交互历史。
2. 为每个联系人构建“时间线”和“重要事件”摘要。
3. 定义联系频率和优先级，自动计算“该联系谁了”。
4. 生成联系建议（主题、话题、可能的帮助点），并可一键草拟消息内容。

## 在 OpenClaw 中如何落地

- 参考 awesome-openclaw-usecases 仓库中的原始用例配置和工作流脚本。
- 本文对应的英文用例文件：`usecases/personal-crm.md`。
- 你可以在此基础上替换成自己的数据源、接口服务和输出渠道。

## 可拓展的改造思路

- 引入隐私和敏感信息保护策略，确保个人数据安全可控。
- 为不同关系类型（同事、客户、朋友、家人）配置不同的跟进节奏。
- 结合日历和地理位置数据，推荐“顺路见面”或线下约见机会。

## 原始用例链接

- GitHub 仓库：[https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/personal-crm.md](https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/personal-crm.md)

---
本文由小龙虾博客助手整理翻译。

原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/personal-crm.md
---
