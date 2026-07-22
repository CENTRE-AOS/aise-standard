# Federation Certificate

> Protocol: CENTRE-FEDERATION v1.0.0
> Layer: Federation Governance
> Status: ACTIVE
> Related: Federation Admission Protocol §FG-4

---

## 1. Purpose

定义 `PROJECT_BOOTSTRAP_READY.md` 的正式语义：它不是"项目初始化完成标记"，而是 **Project Admission Certificate**——项目获得 Federation 身份的正式凭证。

---

## 2. Semantic Redefinition

| 旧含义 | 新含义 |
|--------|--------|
| "项目初始化完成" | "项目通过 Federation Admission，获得 Authority 身份" |
| "Bootstrap 结束标记" | "Admission Certificate — 项目进入 Federation 的通行证" |
| 临时标记，可随意修改 | 正式凭证，修改必须重新通过 FG-1/FG-2 |

---

## 3. Certificate as Contract

Project Admission Certificate 是一个**三方契约**：

```
Protocol Authority (A0)
        │
        │ 定义：Authority 边界、版本要求、Bootstrap 规则
        │
        ▼
Project Admission Certificate (PROJECT_BOOTSTRAP_READY.md)
        │
        │ 声明：本项目遵守上述规则，确认自身 Authority 身份
        │
        ▼
Agent (A1/A2/A4)
        │
        │ 验证：Certificate 内容与 Manifest 一致，然后执行
```

---

## 4. Required Sections

每个 Project Admission Certificate 必须包含以下 Structured Sections：

### 4.1 Identity

Repository 的基本身份信息：

| 字段 | 必填 | 说明 |
|------|:--:|------|
| Repository (local) | YES | 本地目录名 |
| Repository (remote) | YES | GitHub 仓库全名 |
| Authority Level | YES | A0/A1/A2/A3/A4 |
| Authority Name | YES | Protocol / Runtime / Build / Distribution |
| Own Version | YES | 本项目的独立版本号 |
| Protocol Version | YES | 兼容的 Protocol 版本 |
| Foundation Freeze | YES | CENTRE Foundation 版本 |
| Lifecycle | YES | `production` |
| Manifest | YES | Authority-specific manifest 文件名 |

### 4.2 Authority

明确的边界声明。三个维度：

- **Owns**: 本项目拥有/负责的域
- **Does NOT Own**: 本项目不拥有/属于其他 Authority 的域
- **Is NOT** (optional): 明确否认的常见误解身份

### 4.3 Federation Admission Gates

FG-0/FG-1/FG-2 通过状态表。每个 Gate 必须标注 `✅ PASS`。

### 4.4 Owned Resources

具体的文件/目录路径列表。Agent 在此列表外的目录执行 Mutation 操作前必须确认权限。

### 4.5 Forbidden Resources

绝对禁止修改的路径/域。包括：
- 属于其他 Authority 的目录
- Frozen artifacts
- Protocol 定义

### 4.6 Dependencies

- **Upstream**: Provider + Repository + Consumes + Version
- **Downstream**: Consumer + Repository + Consumes + Version
- **Forbidden Consumers**: 明确禁止消费本仓库产物的下游

### 4.7 Next Execution Phase

Agent 下一步执行的具体动作列表。这是 Agent 进入本仓库后的第一个 Mission。

---

## 5. Certificate Lifecycle

```
[未初始化]
     │
     │ FG-0/FG-1/FG-2/FG-3 通过
     ▼
[Certificate Generated]     ← PROJECT_BOOTSTRAP_READY.md created
     │
     │ FG-4 验证通过
     ▼
[Certificate Active]        ← Federation 承认
     │
     │ Authority 变更、Version 升级、Repository 迁移
     ▼
[Certificate Invalidated]   ← 必须重新执行 FG-0~FG-4
```

---

## 6. Certificate Validation

Agent 进入已有 Certificate 的仓库时，必须执行 Certificate Validation：

1. Certificate 中的 `Identity` 字段与当前 Manifest 是否一致？
2. Certificate 中的 `Authority` 声明是否仍然有效？
3. Certificate 中的所有 FG Gates 是否仍然 PASS？
4. Certificate 中的 `Dependencies` 是否与当前 `dependency-lock.json` 一致？

任何一项不匹配 → Certificate Invalidated → 重新执行 FG-0~FG-4。

---

## 7. Template

标准模板位于 `aise-standard/templates/project-bootstrap/PROJECT_BOOTSTRAP_READY.template.md`。

---

## 8. Version

| 属性 | 值 |
|------|-----|
| Protocol Name | CENTRE-FEDERATION Certificate |
| Version | 1.0.0 |
| Status | ACTIVE |
| Part of | CENTRE-FEDERATION v1.0.0 |
