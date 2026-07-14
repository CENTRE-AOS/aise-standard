# Agent Identity & Capability Policy

> 版本：v1.1.0-beta.5
> 状态：Beta
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