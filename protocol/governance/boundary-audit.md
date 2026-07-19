# AOS Architecture Boundary Audit

> 版本：v4.0
> 目的：防止 Runtime 吞掉 Protocol，防止 Factory 吞掉 Instance Context。

## 一、Context 三区分

未来会产生三种 Context，不可混用：

| Context | 所属 | 描述 | 示例 |
|---------|------|------|------|
| Project Context | Runtime Layer | 项目当前运行上下文 | `PROJECT_CONTEXT.md` |
| Runtime Context | Runtime Instance | 运行时实例状态 | `context/runtime-context.json` |
| Agent Context | Agent Instance | Agent 当前会话状态 | `AOS_HOME/context/agent-context.json` |

当前 `PROJECT_CONTEXT.md` 属于 Project Context。

## 二、STATE 四层模型

| State | 所属 | 生命周期 | 回答 |
|-------|------|---------|------|
| Factory State | AOS-Factory | 生产阶段 | 我生产了什么？ |
| Artifact State | Build Pipeline | 构建阶段 | 这个包是什么？ |
| Runtime State | AOS-Runtime | 发行阶段 | 我安装运行的是什么？ |
| Instance State | Agent Instance | 运行阶段 | 一个 Agent 当前活着是什么状态？ |

四个 State 不是复制，描述不同生命阶段。

## 三、Truth 四层模型

| Truth | 所属 | 性质 |
|-------|------|------|
| Protocol Truth | AOS-Factory (protocol/) | 协议规范的定义源 |
| Source Truth | AOS-Factory (source/) | 源码的生产源 |
| Distribution Truth | AOS-Runtime | 发行产物的分发源 |
| Instance Truth | Agent Instance | 实例的运行状态 |

四层不可合并。Runtime 只持有 Distribution Truth。

## 四、Artifact Boundary

```
AOS-Factory (Source)
    │
    ▼
Build Pipeline
    │
    ▼
Artifact (immutable package)
    │
    │  centre-runtime-x.x.x.pkg
    │  manifest.json
    │  SHA256 signature
    │
    ▼
Install
    │
    ▼
AOS-Runtime (Distribution)
```

Artifact 是 Build 产物，不可变。不是源码。不是 Runtime。

## 五、资产归属清单

| 文件 | 归属 | 层 |
|------|------|-----|
| PROJECT_BLUEPRINT.md | Factory | Protocol/Source |
| PROJECT_EVOLUTION.md | Factory | Source |
| PROJECT_MEMORY/ | Factory | Source |
| PROJECT_STATE.json | Factory | Source |
| project.declaration.json | Factory | Source |
| gate-contract.md | Factory (protocol/) | Protocol |
| gate-engine | Factory (runtime/) | Runtime Source |
| PROJECT_CONTEXT.md | Runtime | Distribution |
| PROJECT_STATE.json | Runtime | Distribution |
| PROJECT_LINEAGE.md | Runtime | Distribution |
| adapter-registry.json | Runtime (registry/) | Distribution |
| STATE.json | Agent Instance | Instance |
| MEMORY/ | Agent Instance | Instance |
| CONTEXT/ | Agent Instance | Instance |

## 六、禁止的跨层引用

| 禁止 | 原因 |
|------|------|
| Runtime 读取 Factory MEMORY | 违反 Runtime ≠ Source |
| Factory 包含 Runtime Context | Factory 不知道 Runtime 当前状态 |
| Installer 持有 Adapter 定义 | Adapter 是生态基础设施 |
| Gate 实现混在 Contract 中 | Protocol 定义 WHAT，Runtime 执行 HOW |
| Runtime 自称"唯一真相源" | 只有 Distribution Truth，不是唯一 Truth |