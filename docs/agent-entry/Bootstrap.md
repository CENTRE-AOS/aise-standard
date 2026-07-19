# AISE Bootstrap Protocol

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有 Agent 进入任何 AISE 项目
> 协议：AISE Protocol 1.0

---

## 0. Agent Entry Protocol（Frozen — 不可修改）

以下 4 步入口顺序为 AISE Protocol 1.0 最终冻结协议。**任何 Agent（Codex、Claude Code、Trae、Cursor、Gemini 等）进入项目后必须严格按此顺序执行**。

```text
Step 1: Identity Protocol
        ├── 发现 .agent-entry.json
        ├── 确认 protocol = "AISE"
        ├── 确认 version = "1.0"
        ├── 读取 project_id
        └── 确认 governance.provider
    ↓
Step 2: Asset Protocol
        ├── 读取 PROJECT_BLUEPRINT.md
        ├── 读取 CHANGELOG.md
        └── 加载 .project/ 资产目录
    ↓
Step 3: Context Protocol
        ├── 读取 .project/context/state.json
        ├── 读取 .project/context/mission.json
        └── 读取 .project/context/timeline.jsonl
    ↓
Step 4: Governance Protocol
        ├── 调用 Governance Runtime 执行 aise bootstrap
        ├── 执行 AISE Verify（V0-V10）
        └── 输出 Compliance Report
```

**约束**：
- 步骤顺序不可调整
- 步骤不可跳过
- 步骤不可新增
- 此协议为 Frozen，随 AISE v2.5.0frozen 锁定

---

## 1. 总则

Bootstrap Protocol 定义 Agent 进入 AISE 项目后的初始化流程。确保任何 Agent 不依赖模型默认行为，进入统一工程协议。

AISE 是 **Agent Software Engineering Protocol**，不是框架、不是模板、不是工具。它只定义 Agent 与项目之间的交互契约。

---

## 2. Bootstrap 流程

### Step 1: Identity Protocol — Who am I?

```text
Agent 进入项目
    ↓
发现 .agent-entry.json
    ↓
读取 Protocol Manifest：
    {
      "protocol": "AISE",
      "version": "1.0",
      "project_id": "...",
      "governance": {
        "provider": "agent-governance"
      }
    }
    ↓
确认：
    - protocol 必须为 "AISE"
    - version 必须为 "1.0"
    - project_id 必须非空
    - governance.provider 必须非空
    ↓
输出：Identity PASS / FAIL
```

**判定**：
- `.agent-entry.json` 不存在 → 终止，报告"未启用 AISE"
- `.agent-entry.json` 解析失败 → 终止，报告"Protocol Manifest 非法"
- 字段缺失 → 终止，报告缺失字段

### Step 2: Asset Protocol — What do I own?

```text
读取 PROJECT_BLUEPRINT.md
读取 CHANGELOG.md
加载 .project/ 目录结构：
    ├── context/
    ├── memory/
    │   ├── knowledge/
    │   ├── patterns/
    │   └── glossary/
    ├── decisions/
    ├── architecture/
    ├── journal/
    └── audit/
    ↓
输出：Asset PASS / FAIL
```

**判定**：
- `PROJECT_BLUEPRINT.md` 或 `CHANGELOG.md` 缺失 → 警告，记录到审计
- `.project/` 不存在 → 终止，报告"项目资产缺失"

### Step 3: Context Protocol — What is happening?

```text
读取 .project/context/timeline.jsonl（规范事件源）
读取 .project/context/state.json（当前状态物化视图）
读取 .project/context/mission.json（当前任务/目标）
    ↓
重建项目当前上下文
    ↓
输出：Context PASS / FAIL
```

**判定**：
- `timeline.jsonl` 不存在 → 可创建空文件，记录审计
- `mission.json` 不存在 → 警告，可降级为空 mission

### Step 4: Governance Protocol — How can I change it?

```text
调用 Governance Runtime：
    aise bootstrap
    ↓
执行 AISE Verify（V0-V10）：
    V0: Identity
    V1: Asset
    V2: Context
    V3: Governance
    V4: Contracts
    V5: Policies
    V6: Skills
    V7: Registry
    V8: Version
    V9: Runtime
    V10: Template Sync
    ↓
输出 Compliance Report
    ↓
进入 Autonomous Agent Engineering 模式，等待用户 Mission
```

**判定**：
- Governance Runtime 不存在 → 警告，继续最小基线模式
- Verify 任意 FAIL → 输出修复建议，由用户决定是否继续

---

## 3. 最小基线模式

如果项目未启用 AISE，Agent 仍必须遵守以下最小约束：

| 规则 | 内容 |
|------|------|
| C1 | 任何 Git 操作前执行 `git status --short` + `git diff HEAD` |
| C2 | 覆盖 `.md` 文件前创建 `.bak`，写入后校验非空 |
| C3 | 失败时优先恢复，不破坏 Git 历史 |
| C4 | 不修改 `secrets/.env` 等用户资产 |
| C5 | `git push` 仅推当前分支与当前版本标签，禁止 `--tags` |
| C6 | 不删除 main/master，不使用 `-D/--force` |
| C7 | 编辑前确认改动范围，`old_str` 精确匹配 |
| C8 | 每步修改后测试确认 |
| C9 | 可解释、模块化、零外部依赖 |

---

## 4. Bootstrap Report 模板

```markdown
## AISE Bootstrap Report

- Workspace: <path>
- AISE Detected: YES / NO
- AISE Protocol: 1.0 / N/A
- aise-standard: v2.5.0frozen / N/A
- agent-governance: v2.5.0frozen / N/A
- Governance Provider: <provider> / N/A
- Agent Mode: Autonomous Agent Engineering / General
- Identity: PASS / FAIL / N/A
- Asset: PASS / FAIL / N/A
- Context: PASS / FAIL / N/A
- Governance: PASS / FAIL / N/A
- Compliance: PASS / FAIL / N/A
- Status: READY / BLOCKED
```

---

## 5. 变更控制

- 此文档为 Frozen，随 AISE v2.5.0frozen 锁定
- Agent Entry Protocol（Section 0）不可修改
- 禁止新增、调整、删除 Step
