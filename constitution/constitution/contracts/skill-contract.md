# CENTRE Skill Contract

> 版本: 1.0.0
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: AOS System Skill Layer v3.0
> 前置: SKILL_ARCHITECTURE.md v3.0.0frozen, Runtime Contract v1.0.0

---

## 1. 定义

CENTRE Skill Contract 定义 System Skill 与 Runtime 之间的交互契约。Skill 是 Runtime 的系统服务，不是 Agent 的工具。

## 2. Skill 身份

每个 Skill 必须声明：

```json
{
  "skill_id": "aise-<name>",
  "type": "system",
  "version": "1.0.0",
  "layer": "00-kernel | 10-lifecycle | 20-governance | 30-protocol | 40-adapter | 50-maintenance"
}
```

**Skill 类型：**

| 类型 | 说明 |
|------|------|
| `system` | Runtime 内置 System Skill，不可删除 |
| `extension` | 扩展 Skill，可通过 Registry 安装 |

**Layer 分层：**

| Layer | 目录 | 职责 |
|-------|------|------|
| 00-kernel | 内核层 | 准入控制 |
| 10-lifecycle | 生命周期层 | 进入/退出/交接/归档 |
| 20-governance | 治理层 | Git 操作、审计 |
| 30-protocol | 协议层 | 协议同步、获取 |
| 40-adapter | 适配器层 | Agent 注入 |
| 50-maintenance | 维护层 | 健康检查、更新 |

## 3. Skill Manifest

### 3.1 必填字段

```json
{
  "skill_id": "string",
  "version": "string",
  "runtime_min": "string",
  "protocol": "string",
  "protocol_version": "string",
  "schema_version": "3.0",
  "state_triggers": ["string"],
  "immutable": true,
  "produced_by": "aos-runtime",
  "installed_at": "ISO8601",
  "compatibility": {
    "runtime": "string",
    "protocol": "string",
    "skill_api": "string"
  },
  "integrity": {
    "algorithm": "sha256",
    "hash": "string"
  },
  "trust": {
    "issuer": "aos-runtime",
    "algorithm": "rsa-sha256"
  },
  "requires": ["string"],
  "entry_point": "skill.ps1",
  "actions": ["string"],
  "description": "string"
}
```

### 3.2 Skill 输入/输出

Skill 通过 Runtime 接口获取上下文，不得自行收集：

```
Skill 输入:
  - IContext    → 项目上下文
  - IEvent      → 触发事件
  - IIdentity   → 身份信息
  - ICapability → 外部能力（Git 等）
  - IBootstrap  → Bootstrap Context（Authority Level, Context Validity）

Skill 输出:
  - exit code 0 → 成功
  - exit code 非0 → 失败
  - stdout → 审计日志
  - stderr → 错误信息
```

### 3.3 Skill 权限

System Skill 的权限由 Runtime 授予，不由 Skill 自行声明：

| 权限 | 说明 |
|------|------|
| `read:project` | 读取项目文件 |
| `write:project` | 写入项目资产 |
| `read:registry` | 读取 Registry |
| `write:registry` | 写入 Registry |
| `invoke:capability` | 调用外部能力 |
| `emit:event` | 发射事件 |
| `bootstrap:validate` | 验证 Bootstrap Context 完整性（AGENTS.md + AGENT_CONTEXT.md） |

## 4. Skill 生命周期

```
Install → Verify → Bootstrap Context Check → Activate → Execute → Deactivate → Archive
```

- **Install**: 通过 `centre skills install` 安装 Bundle
- **Verify**: `centre skills verify` 验证 manifest 完整性
- **Bootstrap Context Check**: 验证 AGENTS.md + AGENT_CONTEXT.md 完整性，确认 Authority Level，建立 Context Boundary。必须在 Execute 前执行。
- **Activate/Deactivate**: `centre skills activate/deactivate`
- **Execute**: 由 StateMachine 通过 SkillManager 触发，不可直接调用
- **Archive**: 停用后移至 archive 目录

## 5. 不可违背约束

1. Skill 归属 Runtime，不属于 Agent
2. Skill 不定义协议 — 协议定义属于 aise-standard
3. Skill 不拥有项目资产 — 资产属于 Project
4. Skill 不绕过 Runtime 接口 — 必须通过 Frozen Interfaces 获取上下文
5. Skill 不对 Agent 暴露内部实现 — Agent 只通过 StateMachine 触发
6. Skill 不可变 — 安装后 manifest 不可修改
7. Skill 不直接调用外部工具 — 通过 ICapability 接口
8. Skill 执行前必须通过 Bootstrap Context Check — 验证 AGENTS.md + AGENT_CONTEXT.md