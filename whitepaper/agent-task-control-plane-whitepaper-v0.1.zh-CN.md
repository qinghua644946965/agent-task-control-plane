# Agent Task Control Plane

[English Whitepaper](agent-task-control-plane-whitepaper-v0.1.en.md) | **简体中文白皮书** | [Repository README](../README.md)

## 面向人类治理的多智能体任务控制平面参考架构

**Version:** v0.1  
**Status:** Draft / Open for Discussion  
**Language:** Chinese  
**Author:** Community Proposal  
**License Recommendation:** CC BY 4.0 for this whitepaper; Apache-2.0 or MIT for future reference implementation

---

## 摘要

当 AI Agent 从“单任务、单会话”逐渐走向“多任务、并行、异步执行”后，新的瓶颈开始出现：**Agent 的并行能力增长得很快，但人的工作记忆、上下文切换能力和注意力并不会同步扩展。**

在实际使用多个 Coding Agent 时，一个人很快会遇到以下问题：

- 不记得某个任务当前做到哪一步；
- 不知道代码是否已经部署；
- 不知道部署后是否验证；
- 不清楚某个页面或文件究竟由哪个任务修改；
- 多个任务之间容易串上下文；
- Agent 完成自己的局部工作后容易过早宣布“任务完成”；
- 多个 Agent 并行执行时，人本身开始充当 Router、项目经理、状态机、观察者和消息队列；
- 当任务数量增加时，人的认知负担近似线性上升，最终抵消 Agent 带来的并行收益。

本白皮书提出一个参考架构：

> **Agent Task Control Plane（ATCP）——面向多智能体任务执行的任务控制平面。**

其目标不是制造更多 Agent，而是将以下职责从人脑迁移到系统：

- 任务路由；
- 任务状态管理；
- Agent 协调；
- 执行过程观察；
- 人工审核；
- 结果验证；
- 全链路可追溯；
- 人类注意力调度。

核心观点是：

> **Scaling agents should not scale human cognitive load.**  
> **Agent 数量增加，不应导致人的认知负担同步增加。**

---

# 1. 问题背景

## 1.1 从单 Agent 到并行 Agent

单 Agent 模式下，用户通常只需要维护一个上下文：

```text
用户
 ↓
Agent
 ↓
结果
```

当 Agent 开始支持长任务、异步执行和并行任务后，用户的操作模式变为：

```text
           ┌─ Agent A
用户 ──────┼─ Agent B
           └─ Agent C
```

表面上看，这是并行能力的提升。

但实际运行中，人必须不断维护：

- A 的目标；
- B 的当前状态；
- C 的失败原因；
- 哪个 Agent 还在执行；
- 哪个 Agent 已经完成；
- 哪个 Agent 需要部署；
- 哪个 Agent 已经部署但尚未验证；
- 哪些任务之间存在依赖或冲突；
- 哪条指令属于哪个上下文。

因此，多 Agent 带来的第一个现实瓶颈并不一定是模型，而是：

> **Human Context Management**

---

## 1.2 人脑正在承担不应该承担的系统职责

在缺乏统一任务控制层时，人实际上承担了：

```text
Human
├── Router
├── Coordinator
├── State Store
├── Context Switcher
├── Observer
├── Approval Gate
├── Verifier
└── Trace System
```

这是一种不可扩展的结构。

当并行任务从 1 个增加到 3 个、5 个、10 个时，人需要频繁执行：

```text
卸载任务 A 上下文
↓
重新加载任务 B
↓
确认任务 B 当前状态
↓
执行一次决策
↓
切换任务 C
↓
重新恢复任务 C 的历史约束
```

上下文切换本身开始成为主要成本。

---

# 2. 核心问题定义

ATCP 试图解决的不是“如何启动更多 Agent”，而是：

> **如何让一个人同时管理多个异步 Agent 时，仍然只需要关注少量当前决策。**

核心问题包括：

## 2.1 上下文污染

用户可能将属于任务 A 的消息发送到任务 B。

Agent 往往不会判断“这条消息是否发错”，而是会尝试将其解释进当前上下文，从而造成隐性污染。

---

## 2.2 状态遗忘

典型问题：

```text
代码改了吗？
构建了吗？
部署了吗？
验证了吗？
是谁改的？
哪个任务改的？
```

如果这些问题只能通过聊天记录和人的记忆回答，那么多 Agent 系统无法真正扩展。

---

## 2.3 局部完成 ≠ 任务完成

Coding Agent 可能认为：

```text
代码已经修改完成
→ 任务完成
```

但真实任务可能要求：

```text
代码完成
+ Review
+ Build
+ Deploy
+ Verify
= DONE
```

因此，“DONE”不能只是 Agent 的自然语言声明。

---

## 2.4 目标漂移

任务最初目标可能是：

```text
部署一个已经完成的前端修改
```

执行过程中却逐渐演化为：

```text
排查 SSH
→ 修改配置
→ 更换登录方式
→ 继续排查环境
```

Agent 的当前动作可能逐渐偏离 Original Goal。

---

## 2.5 人类注意力被过度占用

如果 20 个 Agent 同时运行，而用户需要持续关注 20 个任务，那么系统虽然实现了计算并行，却没有实现组织效率提升。

真正理想的系统应该做到：

```text
20 个任务
↓
15 个正常
3 个自动处理中
2 个需要人决策
↓
只把 2 个推给用户
```

---

# 3. 设计目标

ATCP 采用以下设计目标。

## 3.1 Task-first，而不是 Chat-first

系统的一等公民应该是：

```text
Task
State
Event
Artifact
Approval
Trace
```

Chat 只是任务输入的一种方式。

---

## 3.2 状态必须有权威来源

状态不应依赖 Agent 自己描述。

系统需要一个明确的：

> **Source of Truth**

---

## 3.3 确定性逻辑优先使用软件规则

可以使用状态机解决的问题，不应全部交给 LLM 判断。

例如：

```text
DEPLOYED
→ 必须进入 VERIFYING
→ 未完成验证不得 DONE
```

这是确定性规则。

而以下问题适合由 LLM 辅助：

```text
这条用户输入属于任务 A 还是任务 B？
```

```text
当前 Agent 是否明显偏离原始目标？
```

---

## 3.4 默认隔离上下文

不同 Agent 不应默认共享完整聊天历史。

Agent 间通信应优先使用：

- Task ID；
- 结构化状态；
- 事件；
- Artifact；
- 明确约束；
- 最小必要上下文。

---

## 3.5 Human-in-the-loop 应成为状态机的一部分

人工审核不是外围弹窗，而应成为正式状态：

```text
WAITING_HUMAN_APPROVAL
```

---

## 3.6 人类注意力是一种稀缺资源

系统不应把“正常运行”持续推送给用户。

只有以下事件应优先占用人的注意力：

- 高风险操作；
- 方向冲突；
- 无法自动恢复的失败；
- 重要审批；
- 视觉或业务判断；
- Agent 间状态不一致；
- 长时间停滞；
- 目标漂移。

---

# 4. 核心抽象

ATCP 定义六个基础抽象。

## 4.1 Task

Task 是整个系统的核心单位。

一个 Task 至少包含：

```yaml
task_id: T-1024
goal: "修复录像页面播放器说明并上线"
owner: human
priority: normal
created_at: ...
desired_state: VERIFIED
observed_state: EXECUTING
```

---

## 4.2 State

状态描述任务当前生命周期位置。

状态必须由 State Machine Manager 管理。

---

## 4.3 Event

任何重要动作都形成 Event，例如：

```text
TASK_CREATED
AGENT_ASSIGNED
CODE_CHANGED
BUILD_PASSED
DEPLOY_STARTED
DEPLOY_FINISHED
VERIFY_FAILED
HUMAN_APPROVED
TASK_REOPENED
```

Event 构成任务历史。

---

## 4.4 Artifact

Artifact 是 Agent 产生的可验证产物：

- 代码 Diff；
- Commit；
- 构建产物；
- 部署版本；
- 测试报告；
- 截图；
- 日志；
- 数据分析结果；
- 文档。

---

## 4.5 Approval

Approval 表示必须由人或指定角色完成的授权。

---

## 4.6 Trace

Trace 将一次需求从头到尾串起来：

```text
Human Intent
   ↓
Task ID
   ↓
Agent
   ↓
Diff
   ↓
Commit
   ↓
Build
   ↓
Deploy
   ↓
Verify
```

目标是做到：

> 打开任何一个结果，都可以反查“为什么会产生它”。

---

# 5. 核心角色

## 5.1 Router

职责：

> 判断输入属于哪个 Task，或是否应该创建新 Task。

Router 解决的是“发给谁”。

---

## 5.2 Coordinator

职责：

> 管理多个 Task / Agent 之间的依赖、并发、互斥和优先级。

Coordinator 解决的是“谁先做、谁后做、谁等谁”。

例如：

```text
Coding
  ↓
Review
  ↓
Build
  ↓
Deploy
  ↓
Verify
```

或者：

```text
Task A ─┐
        ├→ Build → Deploy
Task B ─┘
```

---

## 5.3 State Machine Manager

职责：

> 管理合法状态、状态迁移条件和任务完成定义。

这是系统的权威状态源。

Agent 不应拥有直接将任务设置为 DONE 的权限。

---

## 5.4 Observer

职责：

> 持续观察 Agent 行为和系统真实状态。

Observer 关注：

- 重复失败；
- 无效循环；
- 目标漂移；
- 长时间停滞；
- Agent 间状态冲突；
- 声明状态与真实状态不一致；
- 上下文异常；
- 未验证假设。

Observer 不负责主要执行。

---

## 5.5 Executor Agent

真正执行具体工作的 Agent，例如：

- Coding Agent；
- Test Agent；
- Deployment Agent；
- Research Agent；
- Data Agent。

---

## 5.6 Verifier

职责：

> 判断任务是否真正满足目标。

Verifier 不应只验证“Agent 是否说完成”，而应验证真实结果。

---

## 5.7 Human Gate

职责：

> 在高风险、不可逆或需要主观判断的位置保留人工控制权。

例如：

- 生产部署；
- 数据库删除；
- 权限变更；
- 大规模重构；
- 视觉验收；
- 业务规则确认。

---

# 6. 参考架构

```text
                         Human
                           │
                           ▼
                        Router
                           │
                           ▼
                     Coordinator
                           │
             ┌─────────────┼─────────────┐
             ▼             ▼             ▼
      State Machine     Observer     Orchestrator
         Manager            │             │
             │              │      ┌──────┼──────┐
             │              │      ▼      ▼      ▼
             │              │    Coder  Tester Deploy
             │              │      │      │      │
             │              └──────┴──────┴──────┘
             │                       │
             └───────────────────────▼
                                  Verifier
                                     │
                                     ▼
                                Human / DONE
```

---

# 7. 状态机模型

## 7.1 基础生命周期

```text
NEW
 ↓
ANALYZING
 ↓
READY_FOR_EXECUTION
 ↓
EXECUTING
 ↓
EXECUTION_DONE
 ↓
REVIEWING
 ↓
READY_TO_DEPLOY
 ↓
WAITING_HUMAN_APPROVAL
 ↓
DEPLOYING
 ↓
DEPLOYED
 ↓
VERIFYING
 ↓
VERIFIED
 ↓
DONE
```

---

## 7.2 异常状态

```text
BLOCKED
FAILED
NEED_HUMAN_INPUT
DEPLOY_FAILED
VERIFICATION_FAILED
ROLLBACK_REQUIRED
REOPENED
```

例如：

```text
VERIFYING
   ├── VERIFIED → DONE
   └── FAILED
        ↓
     REOPENED
        ↓
     EXECUTING
```

---

## 7.3 Desired State 与 Observed State

ATCP 建议区分：

```text
Desired State
Observed State
```

例如：

```yaml
desired_state: VERIFIED
observed_state: DEPLOYED
```

表示任务目标尚未完成，因为部署之后仍需要验证。

如果：

```yaml
declared_state: DEPLOYED
observed_state: OLD_VERSION_RUNNING
```

Observer 应触发状态不一致事件。

---

# 8. Definition of Done

不同任务类型应拥有不同的完成定义。

## 8.1 前端功能

```text
[ ] 代码修改
[ ] lint / build
[ ] review
[ ] deploy
[ ] 页面验证
[ ] 无新增关键控制台错误
```

---

## 8.2 Bug 修复

```text
[ ] Bug 可复现
[ ] 修复完成
[ ] 原场景验证
[ ] 回归验证
[ ] 部署
[ ] 线上确认
```

---

## 8.3 数据库变更

```text
[ ] 变更方案
[ ] 风险评估
[ ] 备份
[ ] 人工审批
[ ] 执行
[ ] 数据校验
[ ] 回滚路径确认
```

状态机根据 Definition of Done 判断任务是否允许进入 DONE。

---

# 9. 可追溯性

每个任务都应拥有唯一 Task ID。

例如：

```text
Task T-018
```

一次完整链路：

```text
用户需求
 ↓
T-018
 ↓
Coding Agent
 ↓
src/pages/device/index.tsx
 ↓
Git Diff
 ↓
Commit a82f31
 ↓
Build #218
 ↓
Deploy prod-20260903-01
 ↓
Verification #V-018
```

这样，当用户看到某个页面时，可以直接反查：

```text
最近由哪个 Task 修改？
由哪个 Agent 执行？
为什么修改？
修改了哪些文件？
是否部署？
是否验证？
```

---

# 10. Human Attention Routing

ATCP 不应让人持续监控所有 Agent。

系统应该进行注意力调度。

例如：

```text
Active Tasks: 20

Normal: 15
Auto-recovering: 3
Need Human: 2
```

用户只看到：

```text
需要你处理：

1. T-031
   请求生产部署审批

2. T-044
   UI 视觉验证
```

设计目标：

> **Agent 数量增加时，人的当前工作集保持尽可能小。**

---

# 11. Observer 的关键检测能力

Observer 可以同时使用：

- LLM 语义判断；
- 状态机；
- Git 状态；
- Build 结果；
- 测试结果；
- 终端输出；
- 部署版本；
- 日志；
- HTTP 检查；
- 时间阈值；
- 重试次数。

例如：

```text
same_action_failures >= 4
AND
observed_state unchanged
```

可以触发：

```text
LOOP_SUSPECTED
```

再例如：

```text
Original Goal:
部署前端修改

Current Actions:
连续修改 SSH 配置
```

Observer 可以触发：

```text
GOAL_DRIFT_SUSPECTED
```

---

# 12. Agent 间通信原则

ATCP 不鼓励：

```text
Agent A 的完整聊天记录
        ↓
全部交给 Agent B
```

推荐：

```yaml
task_id: T-1024
goal: "部署最新前端构建"
current_state: READY_TO_DEPLOY

known_facts:
  - build_passed
  - artifact_id: build-218

constraints:
  - use_existing_deploy_path
  - do_not_change_application_code

expected_output:
  - deployment_result
  - deployed_version
```

即：

> **共享结构化状态，而不是共享整个脑子。**

---

# 13. 最小可行实现（MVP）

第一版 ATCP 不需要成为一个完整 Agent 平台。

建议只实现：

```text
Task
+
State Machine
+
Trace
+
Observer
+
Human Approval
```

## 13.1 MVP Dashboard

```text
Active Tasks: 12

Executing          4
Waiting Approval   2
Need Verification  3
Blocked            1
Failed             1
Done Today         8
```

每个任务显示：

```text
Task
Goal
Agent
Current State
Last Event
Changed Artifacts
Deploy Status
Verify Status
Need Human?
```

---

# 14. 非目标（Non-Goals）

ATCP v0.1 不试图：

- 定义统一的大模型协议；
- 定义通用 AGI 架构；
- 替代 Git；
- 替代 CI/CD；
- 替代现有项目管理系统；
- 让所有决策都由 AI 完成；
- 强制使用多模型；
- 强制所有角色都必须由独立 Agent 实现。

多个逻辑角色可以由：

- 软件规则；
- 状态机；
- 一个 LLM；
- 多个 LLM；
- 人工角色；

共同实现。

---

# 15. 设计原则总结

ATCP 的核心设计原则：

### 1. Task over Chat
任务优先于聊天。

### 2. State over Memory
状态应存储在系统，而不是人的脑子里。

### 3. Trace over Recall
结果应可追溯，而不是依赖回忆。

### 4. Software Rules over LLM Guessing
确定性流程优先交给状态机。

### 5. Isolation over Shared Context
默认隔离上下文。

### 6. Observation over Blind Trust
不要只相信 Agent 自我报告。

### 7. Human Governance over Full Autonomy
人工应该从操作层上升到治理层。

### 8. Attention Routing over Constant Monitoring
只有需要人决策的事情才占用人的注意力。

---

# 16. 未来方向

未来可以继续探索：

- Agent Task Protocol；
- Agent Trace 标准；
- Agent Observability Metrics；
- Goal Drift 检测；
- Agent Context Contamination 检测；
- Human Attention Scheduler；
- Agent Conflict Detection；
- Task DAG 自动生成；
- Agent 权限模型；
- Agent 安全沙箱；
- 多项目资源协调；
- Agent SLA；
- Agent 任务恢复；
- 分布式 Agent 控制平面；
- 与 Git / CI / IDE / Ticket System 集成。

---

# 17. 核心命题

本白皮书最终试图表达的不是：

> 如何让更多 Agent 同时运行。

而是：

> **当 Agent 数量增长时，如何让人仍然能够控制整个系统。**

Agent 可以扩展。

人的工作记忆不会。

因此：

> **多 Agent 的下一层基础设施，不只是 Orchestration，而是 Task Control Plane。**

---

# 18. Discussion

这是一个早期参考架构，而不是最终标准。

欢迎围绕以下问题讨论：

1. Task State 是否需要统一标准？
2. Observer 应该拥有多大的干预权限？
3. Desired State / Observed State 是否适用于通用 Agent？
4. Human Gate 应作为状态、事件还是策略？
5. 如何定义 Agent 的 Goal Drift？
6. 如何减少 Agent 之间的上下文污染？
7. 如何衡量 Human Cognitive Load？
8. 一个用户实际可以管理多少个 Agent？
9. Agent 是否应该拥有直接修改 Task State 的权限？
10. Task Control Plane 是否应该与 Agent Runtime 解耦？

---

## License

建议：

- 本白皮书：**CC BY 4.0**
- 后续参考实现：**Apache License 2.0** 或 **MIT License**

---

## Citation

如果后续发布正式版本，建议在 GitHub 仓库中加入：

```text
CITATION.cff
```

并通过 Zenodo 归档正式版本以获取 DOI。

---

**Agent Task Control Plane — v0.1**

> **Agents scale. Human working memory doesn't.**
