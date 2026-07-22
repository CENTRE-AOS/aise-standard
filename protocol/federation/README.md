# CENTRE Federation Protocol

> Protocol Family: CENTRE-FEDERATION
> Version: 1.0.0
> Status: ACTIVE
> Layer: Federation Governance
> Created: 2026-07-21
> Incident Reference: Post-Federation Bootstrap — Repository Identity Resolution Failure

---

## Architecture Position

Federation Layer 是 CENTRE 架构中此前缺失的独立层，位于 Protocol Layer 和 Runtime Layer 之间：

```
                     CENTRE Architecture

                          │

                  Protocol Layer
                 (AISE Standard)
                          │
                "什么规则？"

                          ▼
          ┌───────────────────────────┐
          │    Federation Layer       │
          │  (Repository Identity     │
          │   + Admission)            │
          │                          │
          │   CENTRE-FEDERATION v1.0  │
          └───────────────────────────┘
                "这个仓库是谁？"

                          │
                          ▼

                  Runtime Layer
            (Gateway Runtime Execution)
                "怎么执行？"

                          │
                          ▼

                  Project Layer
            (Factory / Runtime / Installer)
                          │
                          ▼

                  Agent Layer
```

**为什么独立？**

- Protocol 解决：什么规则？
- Runtime 解决：怎么执行？
- **Federation 解决：这个仓库是谁？**

没有 Federation Layer 时，Agent 看到的是 `filesystem name`，而不是 `authority identity`。导致跨仓库操作中，"正确动作作用于错误对象"。

---

## Protocols

| Protocol | File | Description |
|----------|------|-------------|
| **Federation Admission Protocol** | `admission-protocol.md` | FG-0~FG-4 五 Gate 完整验证链 |
| **Federation Identity Resolution** | `identity-resolution.md` | 身份解析优先级规则（Identity Manifest > Folder Name > Git Remote） |
| **Federation Certificate** | `federation-certificate.md` | PROJECT_BOOTSTRAP_READY 正式语义定义 |

---

## Gate Architecture

```
Federation Layer (FG-0 ~ FG-4):

     FG-0: Repository Discovery Gate
          │
          ├── Archive Artifact? → READ-ONLY, STOP
          ├── Non-CENTRE?       → READ-ONLY, STOP
          ├── UNKNOWN?          → READ-ONLY, STOP
          │
          ▼ (Federation Repository)

     FG-1: Identity Verification Gate  ← Core Gate
          │  Identity Manifest > Folder Name
          │
          ▼

     FG-2: Authority Admission Gate
          │  Authority match + Protocol compat + Dependency Lock
          │
          ▼

     FG-3: Bootstrap Instantiation Gate
          │  Template → Instance
          │
          ▼

     FG-4: Federation Certificate Gate
          │  PROJECT_BOOTSTRAP_READY.md generated
          │
          ▼

Agent Layer (Gate 0-7):
     Gate 0: Identity Check
        ↓
     Gate 1: Project Admission
        ↓
     ...
```

---

## Key Principle

```
Identity Manifest > Folder Name > Git Remote Name
```

优先级链：

| Priority | Source | Weight |
|:--------:|--------|:------:|
| 1 | Protocol Identity (`.project/centre.protocol.json`) | HIGHEST |
| 2 | Project Manifest (`project.declaration.json`) | HIGH |
| 3 | Remote Identity (`.github/remote-identity.json`) | MEDIUM |
| 4 | Filesystem Name | LOWEST — 仅供参考 |

---

## Incident Record

2026-07-21: Post-Federation Bootstrap 阶段，`aos-factory`（`transition_monorepo`, `is_production_source: false`）被错误识别为 A2 Build Authority。

此事故暴露了 Federation Layer 的缺失，直接催生了 CENTRE-FEDERATION v1.0.0。

---

## Related

| Reference | Location |
|-----------|----------|
| Gate Contract (0-7) | `protocol/governance/gate-contract.md` |
| Identity Protocol | `protocol/identity/protocol.md` |
| Bootstrap Contract | `protocol/rfc/bootstrap-contract.md` |
| Admission Certificate Template | `templates/project-bootstrap/PROJECT_BOOTSTRAP_READY.template.md` |
| Federation Sync Record | `CENTRE-FEDERATION-SYNC-v1.0.0.md` (Root workspace) |

---

## Version

| 属性 | 值 |
|------|-----|
| Protocol Family | CENTRE-FEDERATION |
| Version | 1.0.0 |
| Status | ACTIVE |
| Gates | FG-0, FG-1, FG-2, FG-3, FG-4 |
| Foundation Compatibility | CENTRE v3.2.0 |
| Protocol Compatibility | AISE 2.0.0-frozen |
