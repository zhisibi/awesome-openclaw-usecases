## 痛点：一个老派局域网玩家爸爸的烦恼

**故事起点：** 一位自称“老派 LAN 时代玩家”的爸爸，希望给自己的女儿们——3 岁的 Susana 和即将出生的 Julieta——做一个安全、无广告、体验优秀的小游戏门户网站。

他去找现成的网站，却发现：
- 到处是垃圾内容和弹窗
- 充斥着激进广告和误导性按钮（各种“暗黑模式”设计）
- 让一个刚会点鼠标的幼儿频繁误触，体验非常糟糕

于是，他决定自己动手，做一个**干净、快速、简单**的游戏门户。结果发现：
- **搭一个干净的门户很容易**
- 真正的难点是：
  - 要为 0–15 岁不同阶段设计 **40+ 款教育游戏**
  - 一个人既当爸爸又当开发者，完全手写这些游戏，效率太慢
  - 维护几十个游戏的一致性（设计规范、技术栈、目录结构）也越来越难

## 这个自动化流水线做了什么？

这个用例里，他定义了一个“**游戏开发代理（Game Developer Agent）**”，由它来**自动管理游戏从创建到维护的整个生命周期**。

整个工作流执行一个非常严格的原则：**“Bug 优先（Bugs First）”**——在实现任何新功能之前，代理必须优先检查、修复已报告的 Bug。

**效率指标：**

- 这条流水线可以做到 **每 7 分钟产出 1 个新游戏或 1 个 Bug 修复**
- 代理会不知疲倦地在 **41+ 个计划游戏**的待办列表中循环：
  - 一会儿新增一个游戏
  - 一会儿修复之前轮次检测出的问题

当“Bug 队列”清空后，代理会按这样的步骤向前推进：

1. **选择（Select）**：
   - 从 `development-queue.md` 队列里找到下一个要做的游戏
   - 使用“轮转（Round Robin）”策略，在不同年龄段之间均衡内容

2. **实现（Implement）**：
   - 为选中的游戏编写 HTML5/CSS3/JS 代码
   - 严格遵从 `game-design-rules.md` 中规定的规则：
     - 不用框架
     - 移动优先
     - 支持离线

3. **登记（Register）**：
   - 自动把游戏元数据登记到中心注册表 `games-list.json` 中
   - 确保新游戏能出现在首页列表里

4. **记录（Document）**：
   - 更新 `CHANGELOG.md` 里的版本信息
   - 更新 `master-game-plan.md` 里的整体规划状态

5. **部署（Deploy）**：
   - 自动处理整个 Git 流程：
     - fetch 最新 master
     - 创建特性分支
     - 按规范提交（conventional commits）
     - 合并回主干

## 关键 Prompt：把大模型变成“守规矩的游戏开发工程师”

这套工作流的核心，是给代理的一段 **系统级指令（System Instructions）**。

这段 Prompt 要求 LLM 像一个**严谨的 Web 游戏开发工程师 + 儿童体验设计专家**那样，遵守项目里非常严格的结构与规范。

> **说明：** 实际在生产中使用的 Prompt 是西班牙语版本（`es-419`），方便对接拉美地区的儿童用户和未来可能加入的开源贡献者。下面是为了文档展示而翻译成英语的版本。

```text
Act as an Expert in Web Game Development and Child UX.
Your goal is to develop the next game in the production queue.

Please read and analyze the following context files before starting:

1.  BUG CONTEXT (Top Priority - CRITICAL):
    @[bugs/]
    (Check this folder. If there are files, YOUR TASK IS TO FIX **ONLY THE FIRST FILE** (in alphabetical order). Ignore the rest of the bugs and the game queue for now).

2.  QUEUE CONTEXT (Which game is next):
    @[development-queue.md]
    (Identify the game marked as [NEXT] in the "Next Games" section. ONLY if there are no bugs).

3.  DESIGN RULES (Technical Standards):
    @[game-design-rules.md]
    (Strictly follow these rules: Pure HTML/CSS/JS, folder structure, mobile responsiveness)

4.  GAME SPECIFICATIONS (Mechanics and Assets):
    (Identify the corresponding file in games-backlog/ based on the game ID)

5.  CENTRAL REGISTRY (Integration):
    @[public/js/games-list.json]
    (File where you MUST register the new game so it appears on the home page)

TASK:
0. **BUGS FIRST!**: If the `bugs/` folder has content, your only priority is to fix **the first bug in alphabetical order**. Create a `fix/...` branch, resolve **that** bug, update status, and merge. **Do not attempt to fix multiple bugs at once.**
   - IF THERE ARE NO BUGS, proceed with the next game:

1. **Synchronization**: `git fetch && git pull origin master` (CRITICAL).
2. Create a new branch: `git checkout -b feature/[game-id]`.
3. Create the folder and files in 'public/games/[game-id]/'.
4. Implement logic and design according to the backlog and design rules.
5. Register the game in 'games-list.json' (CRITICAL).
6. When finished:
    - Update `CHANGELOG.md` bumping the version.
    - Update `master-game-plan.md` and `development-queue.md`.
    - Document changes: `git commit -m "feat: add [game-id]"`.
7. **Delivery**:
    - Push: `git push origin feature/[game-id]`.
    - Request merge to master.
    - Once in master, push changes (`git push origin master`).
```

这段 Prompt 做了几件非常关键的事：

- **明确优先级：Bug 永远在前**，只有 `bugs/` 目录为空时，才允许做新游戏
- **严格要求读取上下文文件**：
  - Bug 列表
  - 开发队列
  - 设计规范
  - 游戏规格说明
  - 中央注册表
- 用**明确的 Git 流程**约束交付方式，避免“改完不记版本、不写 changelog、不进主干”的混乱局面

## 所需技能

为了让这个“游戏开发代理”正常运转，你至少需要：

- **Git 能力**：
  - 管理分支、提交和合并
  - 支持从代理侧执行 fetch/pull/push 等操作

其余的 HTML5/CSS3/JS 开发能力，全部由大模型在 Prompt 约束下完成。

## 相关项目与链接

- [项目起源故事（LinkedIn）](https://www.linkedin.com/feed/update/urn:li:activity:7429739200301772800/)——讲述这个项目如何在配置好 OpenClaw 之后逐渐成形
- [El Bebe Games 代码仓库](https://github.com/duberblockito/elbebe/tree/master)——完整源代码
- [El Bebe Games 在线站点](https://elbebe.co/)——这条自动化流水线产出的最终成果
- [HTML5 游戏开发最佳实践](https://developer.mozilla.org/en-US/docs/Games)——MDN 的官方指南

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/autonomous-game-dev-pipeline.md
---