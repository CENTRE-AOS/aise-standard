# Memory Ownership Contract

> 版本：v2.5.0frozen
> 状态：P0 强制
> 适用范围：所有 Agent 进入任何 AISE 项目

## 核心原则

**Project Memory 是项目资产，不属于任何 Agent。**

```
     Agent A (Trae)          Agent B (Claude)       Agent C (Codex)
          │                       │                      │
          │  写入                 │  读取                 │  写入
          ▼                       ▼                      ▼
     ┌─────────────────────────────────────────────────────┐
     │              .project/memory/                       │
     │  (统一格式，Agent 无关，项目级知识资产)                │
     └─────────────────────────────────────────────────────┘
```

## 禁止规则

### P0 禁止：Agent 私有 Memory 目录

以下目录**禁止**作为长期知识存储：

| 禁止目录 | Agent | 原因 |
|----------|-------|------|
| `.trae/` | Trae | Agent 私有，其他 Agent 不可见 |
| `.claude/` | Claude Code | 同上 |
| `.workbuddy/` | WorkBuddy | 同上 |
| `.cursor/` | Cursor | 同上 |
| `.codex/` | Codex | 同上 |
| `.gemini/` | Gemini | 同上 |
| `docs/agent-note.md` | 任意 | 无结构化，不可追溯 |

### 例外：.agent/ 目录

`.agent/` 目录**允许**存在，但**仅限**以下内容：

```
.agent/
├── identity.json       # Agent 身份声明
├── capability.json     # Agent 能力声明
└── session.json        # 当前会话状态（临时）
```

**禁止**：
- `.agent/memory.md`
- `.agent/notes.md`
- `.agent/decisions/`
- 任何知识存储内容

## 统一结构

所有 Agent 知识统一存储在：

```
.project/
├── memory/
│   ├── index.json          # Memory 索引
│   ├── knowledge/          # 知识条目
│   │   ├── K001-*.md
│   │   └── K002-*.md
│   ├── decisions/          # ADR 架构决策
│   │   ├── ADR-001-*.md
│   │   └── ADR-002-*.md
│   ├── architecture/       # 架构文档
│   │   ├── runtime.md
│   │   └── data-flow.md
│   ├── patterns/           # 设计模式与约定
│   │   └── coding-standards.md
│   └── glossary/           # 术语表
│       └── terms.md
├── journal/                # 活动日志（时间维度）
│   └── 2026/
│       └── 07/
│           └── 14.json
├── mission/                # Mission Boundary
├── handoff/                # Agent 交接
├── audit/                  # 审计日志
└── releases/               # 发布记录
```

## Memory Index 格式

`.project/memory/index.json`：

```json
{
  "schema_version": "1.1.0",
  "last_updated": "2026-07-14T12:00:00+08:00",
  "knowledge": {
    "K001": {
      "title": "Runtime Layer Architecture",
      "path": "knowledge/K001-runtime.md",
      "status": "accepted",
      "created_by": "claude-code",
      "created_at": "2026-07-14T10:00:00+08:00"
    }
  },
  "decisions": {
    "ADR-001": {
      "title": "Database Choice: PostgreSQL",
      "path": "decisions/ADR-001-database-choice.md",
      "status": "accepted",
      "created_by": "codex",
      "created_at": "2026-07-14T11:00:00+08:00"
    }
  },
  "architecture": {
    "count": 2,
    "files": ["architecture/runtime.md", "architecture/data-flow.md"]
  },
  "patterns": {
    "count": 1,
    "files": ["patterns/coding-standards.md"]
  },
  "glossary": {
    "count": 1,
    "files": ["glossary/terms.md"]
  }
}
```

## Journal vs Memory

| 维度 | Journal | Memory |
|------|---------|--------|
| **性质** | 时间日志 | 知识提炼 |
| **格式** | JSON（结构化事件） | Markdown（文档） |
| **内容** | 谁、什么时候、做了什么 | 学到了什么、为什么 |
| **生命周期** | 追加，不可修改 | 可更新、可废弃 |
| **查询方式** | 按时间 | 按主题 |

```
Journal Entry:
{
  "timestamp": "2026-07-14T14:30:00+08:00",
  "agent": "claude-code",
  "action": "modify",
  "summary": "修复登录模块 NPE 异常"
}
         │
         │ extract
         ▼
Memory Entry:
# K003: Login Module Exception Handling
## Context
NPE 发生在 SessionManager.getUser() 未检查 null...
## Solution
增加 Optional 包装...
```

## 合规检查

Agent 进入项目时，AISE Verify 检查：

1. **禁止目录检查**：是否存在 `.trae/`、`.claude/`、`.workbuddy/` 等私有目录含知识内容
2. **Memory 结构检查**：`.project/memory/` 目录结构是否完整
3. **Memory 所有权检查**：`memory/index.json` 记录的 `created_by` 字段是否正确

## 迁移规则

已有 Agent 私有 memory 的项目迁移：

1. 扫描所有 Agent 私有目录
2. 提取知识内容
3. 转换为统一格式写入 `.project/memory/`
4. 删除 Agent 私有目录中的知识文件
5. 保留 `.agent/` 中的 identity/capability/session

## 变更控制

- 此 Policy 为 P0 强制，不可豁免
- 禁止 Agent 绕过此 Policy 创建私有知识存储
- 违反者记录 Audit Event，工作区标记为 DIRTY