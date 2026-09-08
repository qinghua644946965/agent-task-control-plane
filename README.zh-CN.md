# Agent Task Control Plane

**项目定位：Reference Architecture / Open Discussion Draft。** 参见[定位与现实对照](POSITIONING.md)。本项目不主张首创或概念优先权；拟议收益仍需对照验证。

[English](README.md) | **简体中文**

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.22259004.svg)](https://doi.org/10.5281/zenodo.22259004)

**Agent Task Control Plane（ATCP，智能体任务控制平面）**是一套参考架构，用于治理并行 AI Agent 工作，让人类不必手动追踪每项任务、上下文切换、部署状态、验证步骤和审批节点。

> **Agent 可以扩展，人的工作记忆不会。**

## 为什么需要 ATCP

并行 AI Agent 可能放大已有的协调瓶颈：人类操作者的注意力与上下文管理负担。

当多个长时间运行或异步执行的 Agent 同时工作时，用户很快会遇到以下问题：

- 忘记哪个任务修改了哪个页面或文件；
- 无法确认任务是否已构建、部署或验证；
- 将指令发给错误的 Agent，造成上下文污染；
- 每次切换上下文后都要重新梳理任务历史；
- Agent 在真正满足完成定义之前就宣布“完成”；
- 人工承担路由、协调、观察、状态存储和审批等职责。

ATCP 主张将这些职责从人的工作记忆迁移到专门的**任务控制平面**。

## 核心思想

ATCP 强调 **Task-first，而不是 Chat-first（任务优先，而非聊天优先）**。

系统的核心抽象包括：

- **Task（任务）**
- **State（状态）**
- **Event（事件）**
- **Artifact（产物）**
- **Approval（审批）**
- **Trace（追踪）**

参考架构包含以下逻辑角色：

- **Router（路由器）**——将用户意图路由到正确任务。
- **Coordinator（协调器）**——管理任务依赖、并发、冲突和优先级。
- **State Machine Manager（状态机管理器）**——维护权威生命周期状态和合法状态迁移。
- **Observer（观察者）**——发现漂移、循环、停滞、不一致和未经验证的假设。
- **Executor Agents（执行 Agent）**——完成编码、测试、部署、研究等具体工作。
- **Verifier（验证者）**——验证真实结果是否满足目标。
- **Human Gate（人工关口）**——在高风险或主观性较强的状态迁移中加入明确的人工审批。

## 设计原则

1. **Task over Chat（任务优先于聊天）**  
   聊天是交互界面，任务才是工作单元。

2. **State over Memory（状态优先于记忆）**  
   系统状态不应依赖人的记忆。

3. **Trace over Recall（追踪优先于回忆）**  
   每项结果都应能够追溯到任务、变更、部署和验证。

4. **Software Rules over LLM Guessing（软件规则优先于 LLM 猜测）**  
   确定性工作流应由软件和状态机约束。

5. **Isolation over Shared Context（隔离优先于共享上下文）**  
   Agent 只应获得完成当前任务所需的最小上下文。

6. **Observation over Blind Trust（观察优先于盲目信任）**  
   Agent 的自我报告应通过可观测的系统状态进行核验。

7. **Human Governance over Full Autonomy（人类治理优先于完全自治）**  
   人类应审批关键迁移，而非持续监督整个执行过程。

8. **Attention Routing over Constant Monitoring（注意力路由优先于持续监控）**  
   只有异常、审批、冲突和重要决策才应打断用户。

## 示例任务生命周期

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

编码 Agent 说“已完成”，并不意味着任务已经完成。例如：

```text
代码完成 ✅
构建完成 ✅
部署完成 ❌
验证完成 ❌

任务状态：INCOMPLETE
```

## 仓库结构

```text
agent-task-control-plane/
├── README.md
├── README.zh-CN.md
├── LICENSE
├── drafts/
│   ├── Agent_Task_Control_Plane_v0.1_en.md
│   ├── Agent_Task_Control_Plane_v0.1_en.pdf
│   └── Agent_Task_Control_Plane_v0.1_zh-CN.md
├── diagrams/
└── specs/
```

## 优选基础能力：原生任务续接

ATCP 可以通过外部检查点、摘要和上下文重建运行，但其优选基础能力是 [Native Stateful Task Continuation（原生有状态任务续接）](https://github.com/qinghua644946965/native-stateful-task-continuation)：由用户授权的控制平面能够稳定寻址并继续已有的厂商原生任务，而不是把它重新构造成一次新调用。

两者形成清晰的职责分工：

- **ATCP** 负责全局目标、任务关系、生命周期状态、审批、验证与追踪。
- **原生有状态任务续接** 保持状态局部性，让每个任务的长期局部状态留在其原生厂商运行时中。

因此，ATCP 支持三个能力层级：

1. **无状态重建**——每次调用重新组织所需上下文。
2. **外部检查点恢复**——恢复由控制平面维护的摘要、状态、事件和产物。
3. **原生有状态续接**——直接寻址并继续原来的厂商原生任务，这是优选运行形态。

原生续接是增强 ATCP 的底层能力，而不是硬性依赖。ATCP 在前两个层级仍然可以运行，并在第三个层级获得最佳效果。

## 技术草案

v0.1 草案提供英文版和简体中文版：

- [English](drafts/Agent_Task_Control_Plane_v0.1_en.md)
- [简体中文](drafts/Agent_Task_Control_Plane_v0.1_zh-CN.md)
- [英文 PDF — v0.1 历史归档](drafts/Agent_Task_Control_Plane_v0.1_en.pdf)
- Zenodo DOI：[10.5281/zenodo.22259004](https://doi.org/10.5281/zenodo.22259004)

## 当前状态

- 版本：**v0.1**
- 状态：**Reference Architecture / Open Discussion Draft**
- 重点：**参考架构与问题定义**

## 非目标

ATCP 并不试图：

- 定义通用 AGI 架构；
- 取代 Git、CI/CD、IDE 或项目管理系统；
- 要求每个逻辑角色都由独立 LLM Agent 实现；
- 让所有决策完全自治；
- 强制使用多个模型；
- 标准化所有 Agent Runtime。

它的目标更加聚焦：

> **提供一种控制平面模型，使并行 Agent 工作对人类而言可观察、有状态、可追溯、可治理，并且认知负担可控。**

## 参与讨论

本项目目前仍处于早期阶段，欢迎围绕以下主题展开讨论：

- 标准任务生命周期应该是什么样？
- `Human Gate` 应建模为状态、策略还是事件？
- 如何表达 `Desired State` 与 `Observed State`？
- 如何衡量目标漂移？
- 如何检测上下文污染？
- 确定性编排与 LLM 推理的边界在哪里？
- 如何将提示词、差异、提交、构建、部署和验证连接成完整追踪链路？

欢迎提交 Issue、RFC 风格提案、架构图和反例。

## 引用

如需引用本项目，建议使用：

> Qinghua Ran. *Agent Task Control Plane: A Reference Architecture for Human-Governed Multi-Agent Systems.* Version 0.1, 2026. DOI: [10.5281/zenodo.22259004](https://doi.org/10.5281/zenodo.22259004)

## 许可证

- 仓库参考实现与规范：**Apache License 2.0**
- v0.1 技术草案：**CC BY 4.0**

为保持简单，本仓库目前统一使用 **Apache License 2.0** 作为顶层软件和项目许可证。

## 项目状态

这是一个探索性的参考架构，并非最终标准。

项目源自一个简单观察：

> **并行 Agent 的下一个扩展瓶颈，可能是人的认知负担，而不是 Agent 的执行能力。**
