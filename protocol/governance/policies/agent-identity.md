# Agent Identity & Capability Policy

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有项目
> 协议：AISE Protocol 1.0

## 1. 总则

Agent Identity 确保在多 Agent 环境中每个 Agent 的身份可追溯。Capability Discovery 确保 AISE 能检测 Agent 能力是否满足执行要求。

根据 AISE Protocol 1.0，**Agent 身份不由项目存储，而由 Governance Runtime 统一管理**。项目内不创建 `.agent/` 目录。

## 2. Agent Identity

### 2.1 身份来源

Agent 进入项目后：

1. 读取 `.agent-entry.json` 确认项目身份与治理入口
2. Governance Runtime 为当前会话生成 Agent Identity
3. 所有审计日志关联该会话身份

### 2.2 身份字段

Governance Runtime 维护的 Agent Identity 示例：

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

### 2.3 禁止 `.agent/` 目录

`.agent/` 目录**禁止**出现在普通项目中。Agent 身份与能力声明由 Governance Runtime 在会话期间管理，不写入项目文件系统。

**禁止**：
- `.agent/identity.json`
- `.agent/capability.json`
- `.agent/session.json`
- 任何 `.agent/` 下的文件或目录

Memory 所有权规则详见 `Policies/memory-ownership.md`。

## 3. Capability Discovery

### 3.1 Capability 声明

Governance Runtime 在会话开始时检测当前 Agent 能力：

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

### 3.2 能力检测

Agent 进入项目后：

```text
读取 .agent-entry.json
    ↓
Governance Runtime 检测 Agent 能力
    ├── 满足 minimum_requirements → 进入完整模式
    ├── 满足部分 optional → 限制部分功能
    └── 不满足 minimum → 只读模式
    ↓
输出 Capability Report
```

### 3.3 能力报告

```text
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

- 最低能力要求由 AISE Protocol 版本定义
- Agent Identity 不可伪造
- 禁止 Agent 绕过 Governance Runtime 创建私有身份存储

## 7. Agent 私有目录限制

Agent 不得创建以下目录作为长期知识存储：

| 禁止目录 | 所属 Agent | 原因 |
|----------|-----------|------|
| `.agent/` | 任意 | Agent 身份由 Governance Runtime 管理，不属于项目资产 |
| `.trae/` | Trae | 知识归属项目，不属于 Agent |
| `.claude/` | Claude | 知识归属项目，不属于 Agent |
| `.workbuddy/` | WorkBuddy | 知识归属项目，不属于 Agent |
| `.cursor/` | Cursor | 知识归属项目，不属于 Agent |
| `.codex/` | OpenAI Codex | 知识归属项目，不属于 Agent |
| `.gemini/` | Gemini | 知识归属项目，不属于 Agent |

所有知识必须写入统一格式：`.project/memory/`。

Memory 所有权规则详见 `Policies/memory-ownership.md`。
