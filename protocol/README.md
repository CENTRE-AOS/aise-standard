# AISE Protocol 1.0

> **全称**: Agent Software Engineering Protocol  
> **状态**: Frozen — 1.0  
> **生效日期**: 2026-07-19  
> **所有者**: AISE Protocol Specification (aise-standard v2.6.0frozen)  

---

## 1. 定位

AISE 不是框架、不是模板、不是文件夹系统。

AISE 是一套**协议（Protocol）**，定义 Agent 如何与项目交互。

类似：

- Git 定义版本控制协议
- HTTP 定义网络通信协议
- MCP 定义工具调用协议

AISE 定义 Agent 工程项目协议。

---

## 2. 核心思想

> **Project First, not Agent First.**

- 项目是资产的唯一所有者。
- Agent 是协议的消费者（Protocol Consumer）。
- Governance Runtime 是协议的执行者（Protocol Runtime）。
- AISE 只定义契约，不拥有实现。

任何 Agent、任何 IDE、任何平台，只要遵循 AISE Protocol，就能进入项目、理解项目、贡献项目，而不破坏项目结构。

---

## 3. 四协议结构（v2.6.0frozen 语义重命名）

按 Agent 进入项目的认知顺序：

```text
1. Identity Protocol
        ↓  "Who am I?"
2. Structure Protocol
        ↓  "How is the project organized?"
3. State Protocol
        ↓  "What is the current state?"
4. Evolution Protocol
           "How does the project evolve?"
```

| 协议 | 解决的问题 | 对应文件/目录 |
|------|-----------|--------------|
| Identity Protocol | Agent 如何识别项目 | `.agent-entry.json`, `repository-identities.json` |
| Structure Protocol | 项目如何组织 | `.project/`, `PROJECT_BLUEPRINT.md`, `CHANGELOG.md` |
| State Protocol | 项目当前状态 | `.project/context/`, `timeline.jsonl` |
| Evolution Protocol | 项目如何演进 | Git Policy, Branch Policy, Archive Policy, Freeze Policy |

> v2.5.0frozen 旧名：Identity/Asset/Context/Governance。v2.6.0frozen 重命名为 Identity/Structure/State/Evolution。详见 MAPPING.md。

---

## 4. Protocol Manifest

每个遵循 AISE 的项目，根目录必须包含：

```text
.agent-entry.json
```

这是**协议声明文件（Protocol Manifest）**，不是配置文件。

最小形态：

```json
{
  "protocol": "AISE",
  "version": "1.0",
  "project_id": "uuid-or-identity",
  "governance": {
    "provider": "agent-governance"
  }
}
```

含义：

- 本项目遵循 AISE Protocol 1.0
- 项目的唯一标识是 `project_id`
- 由 `agent-governance` 作为 Protocol Runtime

Agent 读取此文件后，即知道如何加载 Identity / Structure / State / Evolution 四个协议。

---

## 5. 项目结构（协议视角）

```text
project/
│
├── .agent-entry.json          # Protocol Manifest
├── .project/                  # Asset Protocol + Context Protocol
│   ├── context/
│   │   ├── state.json
│   │   ├── mission.json
│   │   └── timeline.jsonl
│   ├── memory/
│   ├── decisions/
│   ├── architecture/
│   ├── knowledge/
│   ├── patterns/
│   ├── glossary/
│   └── journal/
│
├── PROJECT_BLUEPRINT.md       # Asset: 项目蓝图
├── CHANGELOG.md               # Asset: 版本历史
└── src/                       # 业务代码（协议不干涉）
```

注意：

- 项目根目录**不**应该有 `AISE/` 文件夹。
- AISE Protocol 不拥有 `src/`，只规范 `.project/` 和 `.agent-entry.json`。
- 所有 Agent 共享 `.project/`，不存在私有 Memory。

---

## 6. Protocol Runtime

`agent-governance` 是 AISE Protocol 1.0 的参考运行时实现。

运行时职责：

| 命令 | 协议 | 语义 |
|------|------|------|
| `aise init` | Identity + Asset + Governance | 初始化项目，生成 Protocol Manifest 和最小资产骨架 |
| `aise bootstrap` | Identity + Context | 进入项目，恢复上下文 |
| `aise sync` | Asset + Governance | 同步标准与项目资产 |
| `aise audit` | Governance | 检查项目合规性 |
| `aise migrate` | Governance | 升级协议版本 |
| `aise validate` | Governance | 校验协议结构 |

运行时**不**负责：

- 生成业务代码
- 修改项目架构
- 替代 Git
- 替代 Agent 的推理能力

---

## 7. 与 Git 的关系

```text
Git
 |
 | 管代码版本（source code version）
 |
AISE
 |
 | 管 Agent 工程状态（project intelligence）
 |
Agent
```

未来：

```bash
aise commit
```

等价于：

```bash
git commit
+ context snapshot
+ memory update
+ decision archive
```

AISE 是 Git 的上层协议，不替代 Git。

---

## 8. 冻结声明

AISE Protocol 1.0 冻结以下内容：

- 四协议的定义与边界
- `.agent-entry.json` 最小 Schema
- `.project/` 目录结构
- Protocol Runtime 的命令语义

不冻结：

- 具体 Runtime 实现
- IDE 适配器
- UI 形态
- Gateway 部署方式

---

## 9. 相关文档

- `Protocols/identity/protocol.md`
- `Protocols/structure/protocol.md`
- `Protocols/state/protocol.md`
- `Protocols/evolution/protocol.md`
- `Protocols/MAPPING.md` — 术语映射与版本历史
