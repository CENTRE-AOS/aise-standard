# Federation Identity Resolution

> Protocol: CENTRE-FEDERATION v1.0.0
> Layer: Federation Governance
> Status: ACTIVE
> Related: Federation Admission Protocol §FG-1

---

## 1. Purpose

定义 Agent 在 Federation 层解析 Repository 身份时的**优先级规则**。

核心问题：

> **Given a directory, what is the authoritative source of its identity?**

---

## 2. Core Rule

```
Protocol Identity > Project Manifest > Remote Identity > Filesystem Name
```

此优先级链不可违反。Agent 必须始终从最高优先级开始解析，不得跳过。

---

## 3. Priority Chain

### Priority 1: Protocol Identity (HIGHEST)

**Source**: `.project/centre.protocol.json`

**字段**:
| 字段 | 说明 |
|------|------|
| `protocol_id` | 必须为 `"AISE"` |
| `protocol_version` | Protocol 规范版本 |
| `project_id` | 项目唯一标识 |
| `project_role` | 项目在 Federation 中的角色 |

**权威性**: 此文件由 Protocol Authority (aise-standard) 定义 schema，由 Federation Bootstrap 实例化。是最高权威的身份来源。

### Priority 2: Project Manifest (HIGH)

**Source**: `project.declaration.json` 或 Authority-specific manifest（如 `factory.manifest.json`, `installer.manifest.json`, `runtime.manifest.json`）

**字段**:
| 字段 | 说明 |
|------|------|
| `type` | 项目类型 (`protocol` / `runtime` / `factory` / `installer`) |
| `is_production_source` | 是否为生产源码 |
| `authority` / `role` | Authority 自声明 |
| `owns` / `does_not_own` | Authority 边界 |
| `migration.status` | 若存在且为 `pre_migration` — 拒绝 |

**权威性**: 项目自身的身份声明。若与 Priority 1 冲突，以 Priority 1 为准。

### Priority 3: Remote Identity (MEDIUM)

**Source**: `.github/remote-identity.json`

**字段**:
| 字段 | 说明 |
|------|------|
| `authority` | Authority 类型 |
| `authority_level` | A0/A1/A2/A3/A4 |
| `lifecycle` | `production` / `archived` / `transition` |
| `remote_status` | `active` / `inactive` |

**权威性**: Git 层面的远程身份声明。若 `local_path` 与文件系统路径不一致，以 `local_path` 为参考，但优先级低于 Priority 1 和 Priority 2。

### Priority 4: Filesystem Name (LOWEST)

**Source**: Directory name on disk

**权威性**: **仅供调试参考，不用于身份判断。** 目录名可能因迁移、重命名、本地约定等原因与实际仓库身份不一致。

---

## 4. Resolution Algorithm

```
function resolveIdentity(directory):
    // Step 1: Try Protocol Identity
    centre_protocol = read(directory + "/.project/centre.protocol.json")
    if centre_protocol is valid:
        identity = {
            project_id: centre_protocol.project_id,
            project_role: centre_protocol.project_role,
            protocol_version: centre_protocol.protocol_version
        }
        // Cross-validate with Priority 2 and 3
        validateAgainst(project_manifest, remote_identity)
        return identity

    // Step 2: Fallback to Project Manifest (for non-CENTRE projects)
    project_manifest = readProjectManifest(directory)
    if project_manifest is valid:
        identity = {
            type: project_manifest.type,
            authority: project_manifest.authority,
            is_production: project_manifest.is_production_source
        }
        return identity

    // Step 3: Fallback to Remote Identity (for non-initialized repos)
    remote = read(directory + "/.github/remote-identity.json")
    if remote is valid:
        identity = {
            authority_level: remote.authority_level,
            authority: remote.authority,
            lifecycle: remote.lifecycle
        }
        return identity

    // Step 4: UNKNOWN
    return UNKNOWN  // Do NOT use directory name for identity
```

---

## 5. Cross-Validation Rules

当多个 Identity Source 同时存在时，执行交叉验证：

| Source A | Source B | Rule |
|----------|----------|------|
| Protocol Identity | Project Manifest | `project_id` 必须一致 |
| Protocol Identity | Remote Identity | `authority` 语义必须一致（A0 ↔ protocol, A1 ↔ runtime, etc.） |
| Project Manifest | Remote Identity | `authority_level` 必须与 `type` 匹配 |
| All three | Filesystem Name | 不一致是允许的（如 `aos-factory-new` ≠ `aos-factory`），以 Manifest 为准 |

交叉验证失败 → FG-1 HALT。

---

## 6. Identity Conflicts

### Case 1: Filesystem Name ≠ Manifest Identity (ALLOWED)

```
Directory: aos-factory-new
remote-identity.json → local_path: "aos-factory-new"
remote-identity.json → repository.name: "aos-factory"
project.declaration.json → project_id: "aos-factory"
```

**Resolution**: Identity is `aos-factory` (A2 Build Authority). Directory name `aos-factory-new` is a local transition alias.

### Case 2: Transition Monorepo (REJECTED)

```
Directory: aos-factory
project.declaration.json → type: "transition_monorepo"
project.declaration.json → is_production_source: false
```

**Resolution**: REJECT. This is NOT an Authority Repository. It is a migration artifact.

### Case 3: Archive Artifact (READ-ONLY)

```
Directory: archive/aos-factory-transition
ARCHIVE_MANIFEST.json → status: "archived"
```

**Resolution**: READ-ONLY. Do not Bootstrap. Do not Mutate.

### Case 4: No Identity Manifest (UNKNOWN)

```
Directory: some-directory
No .project/centre.protocol.json
No remote-identity.json
No project.declaration.json
```

**Resolution**: UNKNOWN. READ-ONLY. Do NOT use directory name to guess identity.

---

## 7. Anti-Patterns

### Anti-Pattern 1: Name-Based Guessing

```
❌ Directory is named "aos-factory" → Must be A2 Factory
```

### Anti-Pattern 2: Git Remote URL Guessing

```
❌ origin = github.com/CENTRE-AOS/aos-factory → Must be A2 Factory
```
（多个本地目录可以指向同一 remote）

### Anti-Pattern 3: Single-File Guessing

```
❌ Found AGENTS.md with "Build Authority" → Must be A2 Factory
```
（必须读取 Identity Manifest，不可根据内容推测）

---

## 8. Relationship to FG-1

Identity Resolution 是 FG-1 (Identity Verification Gate) 的核心逻辑。FG-1 调用 Resolution Algorithm 获取身份后，再执行交叉验证和 Authority Admission。

参见 `admission-protocol.md` §6。

---

## 9. Version

| 属性 | 值 |
|------|-----|
| Protocol Name | CENTRE-FEDERATION Identity Resolution |
| Version | 1.0.0 |
| Status | ACTIVE |
| Part of | CENTRE-FEDERATION v1.0.0 |
