# Agent Identity & Capability Policy

> 版本：v2.0.2
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

Agent Identity 确保在多 Agent 环境中每个 Agent 的身份可追溯。Capability Discovery 确保 AISE 能检测 Agent 能力是否满足执行要求。

## 2. Agent Identity

### identity.json

位置：`.agent/identity.json`

```json
{
    "agent": "deepseek-v4-pro",
    "version": "2025-07",
    "provider": "deepseek",
    "session": "6a55c6a2f696fd5b00a3ee77",
    "started_at": "2026-07-14T08:00:00",
    "workspace": "local",
    "project": "aise-standard"
}
```

### 多 Agent 环境

```
Project
  ├── Agent A (Codex)     → identity.json { agent: "codex", ... }
  ├── Agent B (Claude)    → identity.json { agent: "claude", ... }
  └── Agent C (Trae)      → identity.json { agent: "trae", ... }
```

### .agent/ 目录结构限制

`.agent/` 目录仅允许以下文件：

| 文件 | 用途 |
|------|------|
| `identity.json` | Agent 身份标识 |
| `capability.json` | 能力声明 |
| `session.json` | 会话状态 |

禁止在 `.agent/` 中创建任何知识存储：

- 禁止 `.agent/memory.md`
- 禁止 `.agent/notes.md`
- 禁止 `.agent/decisions/`
- 禁止 `.agent/knowledge/`
- 禁止任何其他非上述三文件的持久化存储

Memory 所有权规则详见 `Policies/memory-ownership.md`。

所有审计日志自动关联 Agent Identity。

## 3. Capability Discovery

### capability.json

```json
{
    "required_capabilities": [
        "file_read",
        "file_write",
        "terminal",
        "git_exec",
        "test_run"
    ],
    "optional_capabilities": [
        "web_search",
        "mcp_tools",
        "network_access"
    ],
    "compatible_agents": [
        "codex",
        "claude",
        "trae",
        "cursor",
        "copilot"
    ],
    "minimum_requirements": [
        "file_read",
        "file_write",
        "git_exec"
    ]
}
```

### 能力检测

Agent 进入项目后：

```
读取 .agent/capability.json
    ↓
检测 Agent 能力
    ├── 满足 minimum_requirements → 进入完整模式
    ├── 满足部分 optional → 限制部分功能
    └── 不满足 minimum → 只读模式
    ↓
输出 Capability Report
```

### 能力报告

```
=== AISE Capability Check ===

Agent: DeepSeek-V4-Pro
Required: 5/5 ✓
Optional: 2/3 (missing: web_search)

Mode: Autonomous Agent Engineering ✓

Limitations:
  - Web search disabled

=== End ===
```

## 4. 只读模式

不满足最低要求的 Agent 只能：

- 读取文件
- 查看项目信息
- 生成分析报告

不能：

- 修改文件
- 执行 Git 操作
- 触发 Skill

## 5. 审计

Agent Identity 变化记录在 `.project/audit/agent-events.jsonl`：

```json
{"time":"2026-07-14T09:00:00","agent":"deepseek-v4-pro","action":"bootstrap","capabilities":"5/5","mode":"autonomous","result":"pass"}
{"time":"2026-07-14T09:15:00","agent":"claude-code","action":"bootstrap","capabilities":"4/5","mode":"restricted","result":"warning"}
```

## 6. 变更控制

- 最低能力要求由 AISE 版本定义
- Agent Identity 不可伪造

## 7. Agent 私有目录限制

Agent 不得创建以下目录作为长期知识存储：

| 禁止目录 | 所属 Agent | 原因 |
|----------|-----------|------|
| `.trae/` | Trae | 知识归属项目，不属于 Agent |
| `.claude/` | Claude | 知识归属项目，不属于 Agent |
| `.workbuddy/` | WorkBuddy | 知识归属项目，不属于 Agent |
| `.cursor/` | Cursor | 知识归属项目，不属于 Agent |
| `.codex/` | OpenAI Codex | 知识归属项目，不属于 Agent |
| `.gemini/` | Gemini | 知识归属项目，不属于 Agent |

所有知识必须写入统一格式：`.project/memory/`。

Memory 所有权规则详见 `Policies/memory-ownership.md`。

## 8. Memory 所有权

### 核心原则

Project Memory 属于项目，不属于任何 Agent。

### 统一入口

所有 Agent 读写 Memory 的统一入口：

```
.project/memory/
```

### 生命周期

```
Agent 进入项目
    ↓
自动加载 Memory Index (.project/memory/index.md)
    ↓
Agent 执行任务（按需读取/写入 Memory）
    ↓
Agent 退出项目
    ↓
自动更新 Memory Index
```

### 约束

- 任何 Agent 不得拥有独立的 Memory 存储
- 所有 Agent 共享同一套 `.project/memory/` 知识库
- Memory 格式和结构由项目统一定义，不与特定 Agent 绑定