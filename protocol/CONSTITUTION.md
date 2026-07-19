# AOS Architecture Constitution

> 版本: 1.0
> 状态: Frozen
> 生效日期: 2026-07-17
> 适用范围: AOS 全体系统

---

## Chapter 1 — Vision

### 1.1 AOS 是什么

AOS（Agent Operating System）是 Agent 时代的基础设施协议层。它不是软件，不是框架，不是工具集合。它是一个**协议操作系统**——定义 Agent 如何进入、理解、操作、退出一个受治理项目。

### 1.2 AISE 的定位

AISE（Agent Software Engineering Protocol）是 AOS 默认安装的一号协议。它负责工程治理——定义项目如何被创建、组织、理解、恢复和演进。

AOS 不绑死 AISE。未来可以存在其他协议：

```
AOS
├── AISE Protocol (工程协议)       ← 默认一号协议
├── [未来] Trading Protocol       ← 交易协议
├── [未来] Knowledge Protocol     ← 知识协议
├── [未来] Security Protocol      ← 安全协议
└── ...
```

### 1.3 核心目标

> 将 AISE 从"项目工程规范"升级为"Agent Operating System 的工程协议运行环境"，建立 Source → Runtime → Environment → Project 的永久边界，并完成协议、解释器、安装器、版本治理、项目生命周期的统一闭环。

---

## Chapter 2 — Universe Model

### 2.1 Bubble Universe（球宇宙模型）

```
                        HUMAN
                          │
                     Goal / Value
                          │
                          ▼
                     ┌─────────┐
                     │ CENTRE   │  ← 定义秩序，不占有内容
                     │ (Order)  │     不是软件，是原则
                     └────┬─────┘
                          │
           ┌──────────────┼──────────────┐
           │              │              │
           ▼              ▼              ▼
      Protocol       Runtime       Intelligence
      (定义规则)     (执行规则)      (保存生命)
           │              │              │
           └──────────────┼──────────────┘
                          │
                          ▼
                      Project
                      (生命实例)
                          │
                          ▼
                       Agent
                      (执行动作)
                          │
                          ▼
                      Station
                    (观察 — Workbench)
                          │
                          ▼
                       Loop
                    (持续演进)
```

### 2.2 各层定义

| 层级 | 身份 | 职责 | 不做什么 |
|------|------|------|---------|
| **CENTRE** | 秩序原则 | 定义什么是合法 | 不拥有代码、数据、资产 |
| **Protocol** | 宪法 | 描述 WHAT — 规则、Schema、生命周期 | 没有执行能力，零代码 |
| **Runtime** | 执行环境 | 负责 HOW — 安装、解释、验证、执行 | 不定义规则，不关心业务 |
| **Intelligence** | 项目智能 | 保存生命状态 — Blueprint、Decision、Memory | 不拥有协议 |
| **Project** | 生命实例 | 保存源代码、业务资产、项目历史 | 不拥有 Runtime、Hook、CLI |
| **Agent** | 行动执行者 | 读取状态，产生变更 | 不绕过 AOS 准入 |
| **Station** | 观察层 | 人类交互界面（Workbench） | 不是 OS 本体 |
| **Loop** | 演进引擎 | 持续进化 | — |

---

## Chapter 3 — Core Components

### 3.1 三层分离（永久边界）

```
Source (协议工厂)
    │
    │ 负责：研发、版本、发布
    │ 不能：执行协议
    ▼
Runtime (协议执行环境)
    │
    │ 负责：安装、解释、执行、升级
    │ 不能：定义协议
    ▼
Project (生命实例)
    │
    │ 负责：保存生命、资产、状态
    │ 不能：拥有协议、Runtime、Hook
```

### 3.2 AOS Foundation（协议工厂）

- 身份：Protocol Producer（协议生产者）
- 真相源：Git
- 内容：Constitution、Protocol Specification、Schema、Templates、Release Definition
- 产品：AOS Protocol Artifact（不可变）

### 3.3 AOS Runtime（协议执行环境）

- 身份：Protocol Executor（协议执行者）
- 真相源：AOS_HOME
- 模块：Core（Interpreter、Registry、Lifecycle、Environment）、CLI、Installer、Adapter、Package Manager
- 职责：安装协议、解释规则、验证合规、管理生命周期

### 3.4 AOS_HOME（环境基础设施）

```
AOS_HOME/
├── protocol/          ← 宪法（纯文本，零代码）
├── runtime/           ← 执行器（Core + CLI）
├── adapters/          ← 适配器（Git, Agent）
├── registry.json      ← 全局注册表
├── constitution/      ← 宪章副本（约束 Runtime）
└── cache/
```

关键边界：`protocol/` 里没有任何 `.ps1`、`.py`、`.exe` —— 只有 `.md`、`.json`、`.yaml`。

### 3.5 Git（Truth Snapshot Engine）

Git 在 AOS 中的身份不是"代码托管"，而是**真相快照引擎**：

- Snapshot（快照）
- Version（版本）
- Branch（演进）
- Merge（融合）
- Tag（冻结）
- History（历史）

Git 保存 Project 的生命轨迹。Protocol 自己也用 Git，但那是 Protocol Factory 自己的生命状态。两个 Git 不混。

### 3.6 GitHub（Truth Source）

GitHub 是 AOS 的全球分布式真相源，承载：
- Protocol Factory Repository（协议生产）
- Runtime Repository（运行时源码）
- Release Artifacts（协议产品分发）
- Project Repositories（项目生命实例）

---

## Chapter 4 — Five Constitutional Principles

### Principle 1: Source、Runtime、Project 三权分离

任何时候 Source / Runtime / Project 不能混。

- Project 不存放 Runtime、Hook、CLI
- Runtime 不定义协议规则
- Source 不执行协议

### Principle 2: Protocol 永远没有执行能力

Protocol 只描述 WHAT（规则、Schema、生命周期），Runtime 负责 HOW（安装、解释、验证、执行）。

Protocol 永远只有 `.md`、`.json`、`.yaml`，没有任何 `.ps1`、`.py`、`.exe`。

### Principle 3: Runtime 永远不关心业务

Runtime 不知道 Trading、Finance、Workbench、ERP、CRM。Runtime 只知道 Project、Blueprint、State、Identity、Decision。

### Principle 4: Project 永远不拥有协议

Project 只声明它需要哪个版本的 AOS 协议。不复制 Runtime、Hook、CLI。

```json
{
  "aos_protocol": "aise",
  "version": "2.6.0",
  "runtime_required": true
}
```

### Principle 5: 解释器统一治理

所有 CLI 命令（Bootstrap、Verify、Archive、Update、Install、Fetch、Sync）全部经过统一 Interpreter，不各自解析规则。

---

## Chapter 5 — Interaction Protocol

### 5.1 "协议证书"模型

Agent 不能直接进入 Project。必须先通过 AOS Runtime 验证：

```
Agent → AOS Runtime → 验证协议身份 → Project 准入 → 开始工作
```

没有获得 AOS Runtime 身份认证的 Agent，没有资格操作受治理项目。

### 5.2 Agent 标准进入流程

```
1. 检测 AOS_HOME 是否存在
2. 读取 AOS_HOME/protocol/aise/current/     → 知道宪法
3. 读取 Project/.project/aos.json           → 知道项目声明
4. 版本兼容性验证                            → 准入检查
5. Identity  → .agent-entry.json             → 我是谁
6. State     → .project/state/               → 当前状态
7. Blueprint → PROJECT_BLUEPRINT.md          → 架构全貌
8. Decision  → .project/decisions/           → 为什么这样设计
9. Changelog → CHANGELOG.md                  → 最近变化
10. Git      → branch/tag/commit             → 真实快照
11. Code     → src/                           → 开始工作
```

### 5.3 Git 操作门禁

每次 commit/push，`.git/hooks/` 作为薄适配器，调用 AOS Runtime 读取协议规则并执行：

```
git commit
    │
    ▼
Hook (薄适配器)
    │
    ▼
AOS Runtime Interpreter
    │
    ▼
AOS_PROTOCOL/evolution/git-policy.md
    │
    ▼
执行规则 → 通过 / 拒绝
```

---

## Chapter 6 — Lifecycle

### 6.1 项目生命周期

```
Create ──→ Bootstrap ──→ Develop ──→ Checkpoint ──→ Freeze ──→ Handoff ──→ Continue
  │           │            │            │              │           │
  │           │            │            │              │           │
aise-init  aise-bootstrap  git flow   aise-verify   aise-archive  agent_exit
                                                │
                                          vX.Y.Zfrozen
                                          (immutable tag)
```

### 6.2 协议生命周期

```
ACP+AISE Factory (develop)
    │
    │ 研发、测试、迭代
    ▼
ACP+AISE Factory (main)
    │
    │ aise-archive
    ▼
vX.Y.Zfrozen tag (immutable)
    │
    │ aise-release
    ▼
Protocol Artifact Package
    │
    │ aise-install
    ▼
AOS_HOME/protocol/aise/vX.Y.Z/
```

---

## Chapter 7 — Evolution

### 7.1 协议升级与项目升级分离

- 协议升级：通过 AOS Foundation 的 develop → main → frozen 流程
- 项目升级：通过 `aise-install` 更新 AOS_HOME 中的协议版本
- 已有项目不受协议升级影响，除非主动声明新版本

### 7.2 Frozen 版本不可变

- `vX.Y.Zfrozen` tag 一旦推送，永不移动
- 补丁使用新版本号（`v2.6.1frozen`）
- 紧急热修复使用 `v2.6.0frozen-patch-N`，不覆盖原 tag

### 7.3 向后兼容

- Runtime 最低版本检查：`compatibility.minimum_runtime`
- 协议版本兼容性：`compatibility.supported_protocols`
- 项目声明版本与已安装协议版本必须匹配

---

## Appendix A: 命名体系（永久冻结）

| 身份 | 名称 | 本质 | 真相源 |
|------|------|------|--------|
| **协议工厂** | AOS Foundation | 研发 AOS 协议，生产 Protocol Artifact | Git |
| **协议运行环境** | AOS Runtime | 安装、分发、执行协议 | AOS_HOME |
| **真相快照引擎** | Git | 记录和冻结项目生命轨迹 | Git |
| **生命实例** | Project | 遵循协议运行的生命体 | Git + AOS_HOME |

## Appendix B: 不在 AOS 范围内的

- Gateway（网络入口，未来独立模块，是 Runtime 的网络化，不是新 Runtime）
- Workbench（Station，观察层，AOS 的第一个 Reference Implementation）
- 任何业务协议（Trading、Finance、ERP 等）
- 任何 Agent 平台自身（Claude、Trae、Cursor 等，它们通过 Adapter 接入）