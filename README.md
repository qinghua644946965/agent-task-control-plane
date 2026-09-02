# Agent Task Control Plane

**English** | [简体中文](README.zh-CN.md)

**Agent Task Control Plane (ATCP)** is a reference architecture for governing parallel AI-agent work without forcing humans to manually track every task, context switch, deployment state, verification step, and approval point.

> **Agents scale. Human working memory doesn't.**

## Why this exists

Parallel AI agents create a new bottleneck: the human operator.

Once several long-running or asynchronous agents are active at the same time, users quickly encounter problems such as:

- forgetting which task changed which page or file;
- losing track of whether a task was built, deployed, or verified;
- sending instructions to the wrong agent and contaminating context;
- repeatedly reconstructing task history after context switches;
- allowing agents to declare "done" before the real Definition of Done is satisfied;
- manually acting as router, coordinator, observer, state store, and approval gate.

ATCP proposes moving those responsibilities from human working memory into a dedicated **task control plane**.

## Core idea

ATCP is **Task-first, not Chat-first**.

The core system abstractions are:

- **Task**
- **State**
- **Event**
- **Artifact**
- **Approval**
- **Trace**

The reference architecture introduces several logical roles:

- **Router** — routes user intent to the correct task.
- **Coordinator** — manages task dependencies, concurrency, conflicts, and priorities.
- **State Machine Manager** — owns authoritative lifecycle state and valid transitions.
- **Observer** — detects drift, loops, stalls, inconsistencies, and unverified assumptions.
- **Executor Agents** — perform coding, testing, deployment, research, or other work.
- **Verifier** — verifies that the real-world result satisfies the goal.
- **Human Gate** — inserts explicit human approval into high-risk or subjective transitions.

## Design principles

1. **Task over Chat**  
   Chat is an interface. Task is the unit of work.

2. **State over Memory**  
   System state should not depend on what a human remembers.

3. **Trace over Recall**  
   Every result should be traceable back to a task, change, deployment, and verification.

4. **Software Rules over LLM Guessing**  
   Deterministic workflow should be enforced by software and state machines.

5. **Isolation over Shared Context**  
   Agents should receive the minimum context required for their current task.

6. **Observation over Blind Trust**  
   Agent self-reports should be checked against observable system state.

7. **Human Governance over Full Autonomy**  
   Humans should approve critical transitions rather than continuously supervise execution.

8. **Attention Routing over Constant Monitoring**  
   Only exceptions, approvals, conflicts, and important decisions should interrupt the user.

## Example task lifecycle

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

A coding agent saying "finished" does **not** automatically mean the task is done.

For example:

```text
Code complete ✅
Build complete ✅
Deploy complete ❌
Verification complete ❌

Task state: INCOMPLETE
```

## Repository structure

```text
agent-task-control-plane/
├── README.md
├── README.zh-CN.md
├── LICENSE
├── whitepaper/
│   ├── agent-task-control-plane-whitepaper-v0.1.en.md
│   └── agent-task-control-plane-whitepaper-v0.1.zh-CN.md
├── diagrams/
└── specs/
```

Suggested future specs:

```text
specs/
├── task-model.md
├── state-machine.md
├── observer.md
├── trace-model.md
├── human-gate.md
└── task-protocol.md
```

## Whitepaper

The whitepaper is available in English and Simplified Chinese:

- [English](whitepaper/agent-task-control-plane-whitepaper-v0.1.en.md)
- [简体中文](whitepaper/agent-task-control-plane-whitepaper-v0.1.zh-CN.md)

Current status:

- Version: **v0.1**
- Status: **Draft / Open for Discussion**
- Focus: **Reference architecture and problem definition**

## Non-goals

ATCP is not intended to:

- define a universal AGI architecture;
- replace Git, CI/CD, IDEs, or project-management systems;
- require every logical role to be implemented as a separate LLM agent;
- make every decision autonomous;
- require multiple models;
- standardize all agent runtimes.

The goal is narrower:

> **Provide a control-plane model for making parallel agent work observable, stateful, traceable, governable, and cognitively manageable for humans.**

## Contributing

This project is intentionally early-stage.

Useful discussion topics include:

- What should the canonical task lifecycle look like?
- Should `Human Gate` be modeled as a state, policy, or event?
- How should `Desired State` and `Observed State` be represented?
- How should goal drift be measured?
- How can context contamination be detected?
- What is the right boundary between deterministic orchestration and LLM reasoning?
- How should task traceability connect prompts, diffs, commits, builds, deployments, and verification?

Issues, RFC-style proposals, diagrams, and counterexamples are welcome.

## License

- Repository reference implementation and specifications: **Apache License 2.0**
- Whitepaper text: if you want separate document licensing, consider **CC BY 4.0**

For simplicity, this repository currently uses **Apache License 2.0** as the top-level software/project license.

## Status

This is an exploratory reference architecture, not a finalized standard.

The project is driven by a simple observation:

> **The next scaling bottleneck of parallel agents may be human cognitive load, not agent execution capacity.**
