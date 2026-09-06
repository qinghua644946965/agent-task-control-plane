# 项目定位 / Project positioning

**Reference Architecture / Open Discussion Draft**
**现实对照核查日期 / Comparison reviewed: 2026-09-06**

## 中文

本项目不是首创、优先权或独占概念声明。名称、公开日期和文档归档用于识别与引用，不证明率先提出或填补行业空白。本项目保留问题洞察与验证价值，不以“尚无人实现”为成立前提。它不是行业标准、完整市场调查或已验证的产品优势声明。

### 真正关注的问题

模型、长任务 runtime 与外部控制平面如何划分责任，才能减少跨任务切换、错误路由和过早完成带来的人工负担？Task / State / Event / Artifact / Approval / Trace 是组织既有能力的概念模型，不是新发明。

### 现实对照的解释范围

下表列出已核查官方文档中的相邻能力，不是完整竞品清单。文档记载的能力、配置后可用的机制、本仓库原型与端到端实测应分别标注；本次未对外部产品做部署测试。未核查的能力记为未知，不能记为不存在。基础能力已有工程实现，不表示每一种 Agent 产品或组合都已成熟。

## English

This project makes no claim of invention priority or exclusive ownership of a concept. Its name, publication date, and archive support identification and citation, not a claim to be first or to fill an industry void. Its value rests on problem analysis and validation rather than an assertion that nobody has implemented the idea. It is not an industry standard, exhaustive market survey, or demonstrated product advantage.

### The actual question

How should models, long-running runtimes, and an external control plane divide responsibility to reduce context switching, misrouting, and premature completion? Task / State / Event / Artifact / Approval / Trace organize existing capabilities; they are not new inventions.

### How to read the comparison

The table records adjacent capabilities described in official documentation, not an exhaustive competitor list. Distinguish documented capability, configured functionality, this repository’s prototype, and end-to-end measurements. No external product deployment tests were performed for this review. Unchecked means unknown, not absent. Established foundations do not imply that every agent product or combination is mature.

## 已有基础与边界 / Existing foundations and boundaries

| 方向 / Area | 官方来源 / Official source | 已有能力 / Existing capability | 对照边界 / Comparison boundary |
| --- | --- | --- | --- |
| Durable execution | [Temporal](https://docs.temporal.io/) | 持久执行与故障后恢复 / Durable execution and recovery | 业务副作用仍需幂等与对账；不证明本项目组合收益 / Side effects still need idempotency and reconciliation; no proof of this proposal’s benefit |
| Workflow state / checkpoint / resume / HITL | [LangGraph persistence](https://docs.langchain.com/oss/python/langgraph/persistence) · [interrupts](https://docs.langchain.com/oss/python/langgraph/interrupts) | 持久化状态、检查点与暂停后继续 / Persisted state, checkpoints, and interrupt/resume | 依赖存储与正确恢复配置；不等于任意厂商原生 UI 任务可被外部续接 / Requires storage and correct recovery setup; not access to arbitrary provider UI tasks |
| Trace / observability | [OpenTelemetry](https://opentelemetry.io/docs/what-is-opentelemetry/) | 追踪、指标与日志基础设施 / Traces, metrics, and logs | 遥测不自动证明业务验收或语义正确性 / Telemetry alone does not establish business acceptance or semantic correctness |
| Policy / governance | [Open Policy Agent](https://www.openpolicyagent.org/docs) | 策略决策与执行分离 / Separation of policy decisions and enforcement | 调用方仍须强制执行策略；不能仅靠模型遵循文字 / The caller must enforce decisions; model compliance with prose is insufficient |

## 验证与持续修订 / Validation and revision

用同一组并行任务比较现有 runtime 加任务看板与 ATCP 映射方案，记录人工干预次数、恢复时间、误报漏报、验收遗漏及总成本。没有可复现收益时，优先保留已有工具并收缩控制平面。

Compare an existing runtime plus task dashboard with an ATCP mapping on the same parallel tasks. Measure human interventions, recovery time, false and missed alerts, acceptance omissions, and total cost. Without reproducible benefit, retain existing tools and reduce the control-plane scope.

每次重要修订补充具体产品、版本/日期、访问入口、数据与权限范围、官方来源、复现步骤、结果及限制。先判断能否复用已有工具，再说明本项目增加了什么以及代价。发现已有方案覆盖需求时，更新对照、缩小范围或转为参考实现/用户倡议；不为了维护原创性而人为扩大缺口。保留现有免责声明、失败记录和证据等级。

For each substantive revision, record the product, version/date, interface, data and permission scope, official sources, reproduction steps, outcomes, and limits. Evaluate reuse before adding a layer, and explain its cost. When an existing solution covers the need, update the comparison, narrow scope, or focus on a reference implementation or user initiative. Do not manufacture gaps to defend originality. Preserve existing caveats, failures, and evidence levels.

## 责任边界 / Responsibility boundaries

Durable execution、checkpoint/resume、HITL、workflow state、trace、observability 与 agent governance 都不是本项目的新发明。治理中的身份、授权、审批、审计与职责分离已有长期工程基础；Agent 专用产品的覆盖范围仍应逐项核查。模型提出语义判断与计划；runtime 维护执行连续性；外部层仅在有需要时组织跨任务目标、证据、授权和注意力。图中的逻辑角色不要求单独部署，更不要求重建现有引擎。已有 runtime 若已覆盖所需治理，外部层可缩为适配或视图。

Durable execution, checkpoint/resume, HITL, workflow state, trace, observability, and agent governance are not inventions of this project. Identity, authorization, approvals, audit, and separation of duties have established engineering foundations; agent-specific product coverage still needs individual checks. Models propose semantic judgments and plans; runtimes maintain execution continuity; an external layer organizes cross-task goals, evidence, authorization, and attention only where needed. Logical roles need not be separate deployments or reimplement existing engines. When a runtime already supplies the required governance, the external layer may be only an adapter or view.

## 归档与当前稿 / Archive and current draft

已发布 v0.1 PDF 与 Zenodo DOI 保留为历史版本；它们未随本次定位修订重新发布。当前 Markdown 是在 v0.1 基础上修订的讨论稿，应与本定位说明一起阅读。引用本次修订时应附 Git 提交版本，不能声称旧 DOI 已包含本次修改。

The published v0.1 PDF and Zenodo DOI remain historical versions and were not republished for this positioning revision. Current Markdown is a discussion revision based on v0.1 and should be read with this note. Cite a Git commit when referring to these changes; do not imply that the existing DOI includes them.
