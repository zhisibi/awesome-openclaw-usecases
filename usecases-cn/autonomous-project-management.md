管理一个包含多条并行工作流的复杂项目，是一件极其耗神的事：

- 你要在不同工具之间来回切换
- 频繁上下文切换、同步进度、安排交接
- 很多时候，你更像是在“指挥交通”，而不是做高价值的思考

这个用例实现了一种**去中心化自治项目管理模式**：让多个子 Agent 通过共享的状态文件协同工作，而不是靠一个“中央调度 Agent”来 micromanage 一切。

---

## 一、痛点：传统 Orchestrator 模式的瓶颈

传统的“总控 Orchestrator”模式，常见问题包括：

- 主 Agent 变成了**交通警察**：
  - 所有任务派发、结果汇总都得走它
  - 每件事都要它来“批示”下一步

- 项目一旦复杂（多仓库重构、研究冲刺、内容工厂流水线），主 Agent 就：
  - 频繁阻塞
  - 上下文混乱
  - 难以真正做到并行

现实中，更好的模式是：

> **让 Agents 自己围绕共享状态自组织，而不是靠一个主脑来调度所有细节。**

---

## 二、这个方案做了什么？

这套自治项目管理模式的核心思路：

- **去中心化协调**  
  - 所有 Agent 都读写一个共享的 `STATE.yaml` 文件
  - 这个文件是项目状态的**单一事实来源（single source of truth）**

- **并行执行**  
  - 多个子 Agent 可以同时工作、各自推进自己的任务
  - 通过读写 `STATE.yaml` 协调依赖与进度

- **主会话极瘦（CEO 模式）**  
  - 主 Agent 只负责：
    - 接任务
    - 指派合适的 PM 子 Agent
    - 定期查看 `STATE.yaml` 总结进度
  - 不参与具体执行

- **自文档化（Self-documenting）**  
  - 所有任务信息、进度、阻塞原因，都写在版本控制的状态文件中
  - 形成天然的项目日志 / 历史记录

---

## 三、核心模式：STATE.yaml

每个项目维护一个 `STATE.yaml` 作为项目协调文件，结构示例如下：

```yaml
# STATE.yaml - Project coordination file
project: website-redesign
updated: 2026-02-10T14:30:00Z

tasks:
  - id: homepage-hero
    status: in_progress
    owner: pm-frontend
    started: 2026-02-10T12:00:00Z
    notes: "Working on responsive layout"
    
  - id: api-auth
    status: done
    owner: pm-backend
    completed: 2026-02-10T14:00:00Z
    output: "src/api/auth.ts"
    
  - id: content-migration
    status: blocked
    owner: pm-content
    blocked_by: api-auth
    notes: "Waiting for new endpoint schema"

next_actions:
  - "pm-content: Resume migration now that api-auth is done"
  - "pm-frontend: Review hero with design team"
```

关键点：

- `tasks` 数组里，每个任务都是一个**最小可交付工作单元**
- `status` 清晰标明：`todo / in_progress / blocked / done` 等
- `owner` 表明当前负责的子 Agent（例如 `pm-frontend`, `pm-backend`）
- `blocked_by` 说明依赖关系
- `next_actions` 提醒下一步动作，类似简版的站会记录

---

## 四、工作流程是怎样的？

一个完整的自治项目管理工作流通常是这样：

1. **主 Agent 收到一个新任务**  
   - 比如：“重构认证模块并更新文档”

2. **主 Agent 按项目生成 / 选择 PM 子 Agent**  
   - 若已有项目 PM，则转发给它
   - 若是新项目，则 spawn 一个新的 PM 子 Agent

3. **PM 子 Agent 读取对应项目的 `STATE.yaml`**  
   - 若文件不存在，则创建并拆解任务
   - 若已存在，则更新任务列表与状态

4. **PM 子 Agent 自主推进任务**  
   - 可以再继续拆分细粒度子任务
   - 需要并行时，spawn 子子 Agent（例如专职写文档、改代码）
   - 每推进一步，就更新 `STATE.yaml`

5. **其他 Agent 轮询 `STATE.yaml`**  
   - 发现自己负责且状态为 `todo` / `unblocked` 的任务
   - 自动 Pick up 并推进

6. **主 Agent 定期查看 `STATE.yaml`**  
   - 总结当前项目进度
   - 把高层状态回报给你

整个过程里，主 Agent 更像一个 CEO：

- 制定方向
- 选择合适的 PM
- 间歇性查看报表

而真正的执行和协作，都在子 Agent 与共享状态文件之间完成。

---

## 五、需要的技能 / 能力

要实现这种自治项目管理，Agent 需要具备：

- **`sessions_spawn` / `sessions_send` 能力**  
  - 用于创建 / 管理子 Agent 会话

- **文件系统访问能力**  
  - 读写 `STATE.yaml` 文件

- **Git 能力（推荐）**  
  - 把 `STATE.yaml` 纳入版本控制
  - 所有状态变更都有 commit 记录成为审计日志

---

## 六、AGENTS.md 中的配置示例

你可以在 AGENTS.md 中引入一个“PM Delegation Pattern”的规范：

```text
## PM Delegation Pattern

Main session = coordinator ONLY. All execution goes to subagents.

Workflow:
1. New task arrives
2. Check PROJECT_REGISTRY.md for existing PM
3. If PM exists → sessions_send(label="pm-xxx", message="[task]")
4. If new project → sessions_spawn(label="pm-xxx", task="[task]")
5. PM executes, updates STATE.yaml, reports back
6. Main agent summarizes to user

Rules:
- Main session: 0-2 tool calls max (spawn/send only)
- PMs own their STATE.yaml files
- PMs can spawn sub-subagents for parallel subtasks
- All state changes committed to git
```

要点：

- 主会话只负责协调，工具调用严格限制在 0–2 次（spawn / send）
- 每个 PM 子 Agent“拥有”自己负责项目的 `STATE.yaml`
- PM 可以再向下派生子子 Agent 做并行子任务
- 所有状态更新都要 commit 到 Git，以便回溯

---

## 七、示例：如何启动一个 PM

假设你说了一句：

> “重构认证模块并更新文档。”

主 Agent 的行为大致是：

1. 检查 `PROJECT_REGISTRY.md`：
   - 如果没有活跃的 `pm-auth` 项目管理 Agent：

2. 调用 spawn：

   ```text
   sessions_spawn(
     label="pm-auth-refactor",
     task="Refactor auth module, update docs. Track in STATE.yaml"
   )
   ```

3. 然后回复你：

   > “已创建 pm-auth-refactor 子 Agent，它会负责拆解任务并在 STATE.yaml 中跟踪进度。我会在完成后给你汇报整体结果。”

此时，PM 子 Agent 的流程是：

1. 创建 / 更新 `STATE.yaml`，拆分任务：
   - 例如：`auth-api-refactor`、`auth-doc-update`、`tests-hardening` 等

2. 自己推进一部分任务，必要时：
   - spawn 代码执行子 Agent
   - spawn 文档专职 Agent

3. 在每个任务的生命周期里，更新 `status`、`notes`、`output` 等字段

4. 当所有任务完成时：
   - 在 `STATE.yaml` 中标记项目完成
   - 给主 Agent 一个汇总报告

---

## 八、关键洞见

- **`STATE.yaml` > 中央 Orchestrator**  
  - 基于文件的协调，比纯消息传递更稳定、更易回溯
  - 所有 Agent 都围绕同一份“事实源”工作

- **Git 即审计日志**  
  - 每次改动 `STATE.yaml` 都是一次 commit
  - 完整记录了项目推进的决策过程和状态变更轨迹

- **命名规范非常重要**  
  - 建议用 `pm-{project}-{scope}` 的方式命名 PM Agent
  - 例如：`pm-auth-refactor`、`pm-website-redesign`

- **主会话越“瘦”越好**  
  - 主 Agent 不陷入执行细节，响应会更快
  - 你的注意力也可以放在高层策略而不是具体执行

---

## 九、灵感来源与相关链接

这种自治项目管理模式，灵感来自 Nicholas Carlini 等人在自治编码 Agent 领域的实践：

> 与其微管理每个步骤，不如让 Agent 围绕共享状态文件自组织。

相关链接：

- [OpenClaw Subagent 文档](https://github.com/openclaw/openclaw)
- [Anthropic: Building Effective Agents](https://www.anthropic.com/research/building-effective-agents)

---
本文由小龙虾博客助手整理翻译。
原文链接：https://github.com/zhisibi/awesome-openclaw-usecases/blob/main/usecases/autonomous-project-management.md
---