# Repository Visibility Policy

> Version: 1.0.0-frozen
> Date: 2026-07-21
> Phase: S0.6 — Remote Ownership Freeze
> Status: FROZEN

---

## 1. Visibility Matrix

| Authority | Repository | Visibility | Reason |
|-----------|-----------|:----------:|--------|
| A0 Protocol | aise-standard | **public** | 治理标准，需要生态传播和公共审查 |
| A1 Source | aos-runtime | **public** | 执行引擎，需要生态传播和公共审查 |
| A2 Build | aos-factory | **private** | 包含生产链逻辑和签名密钥，不应公开 |
| A4 Install | aos-installer | **private** | 包含部署逻辑和环境配置，不宜公开 |
| Archive | aos-migration | **private** | 历史归档，仅供内部参考 |

## 2. Visibility 规则

| Visibility | 允许 | 禁止 |
|-----------|------|------|
| public | 任何人可以 clone/fork/star | — |
| private | 仅 Owner 和授权 Collaborator 可访问 | 公众可见 |

## 3. 不可违背约束

1. A0 (Protocol) 必须 public — 治理标准不可隐藏
2. A2 (Factory) 必须 private — 签名密钥不可暴露
3. A3 (Artifact Registry) Phase 3 实现时另行定义
4. Visibility 修改需要 Phase 2.8 Architecture Review

## 4. S1 GitHub Creation Scope

S1 仅创建四个生产仓库：

| 创建 | Repository | Lifecycle |
|:----:|-----------|-----------|
| ✅ | aise-standard | production |
| ✅ | aos-runtime | production |
| ✅ | aos-factory | production |
| ✅ | aos-installer | production |

**不在 S1 创建**：

| 不创建 | Repository | 原因 |
|:------:|-----------|------|
| ❌ | aos-migration | Archive — 不属于 Production Authority，生命周期不同（history → preserve → reference） |
| ❌ | artifact-registry | RESERVED — Phase 3.0 实现 |

## 5. Owner 设置

| Repository | Owner | 说明 |
|-----------|-------|------|
| aise-standard | `<github-org-or-user>` | S1 创建时确定 |
| aos-runtime | `<github-org-or-user>` | S1 创建时确定 |
| aos-factory | `<github-org-or-user>` | S1 创建时确定 |
| aos-installer | `<github-org-or-user>` | S1 创建时确定 |
| aos-migration | `<github-org-or-user>` | S1 创建时确定 |

**注意**: `<owner>` 占位符在 S1 创建 GitHub Repository 时替换为实际 GitHub 组织或用户名。

## 6. Remote Identity Schema（Frozen）

S0.7 冻结的 remote-identity.json 顶层字段：

```json
{
  "identity": "<repo>-v<version>",
  "authority": "protocol|runtime-source|factory|installer",
  "authority_level": "A0|A1|A2|A4",
  "lifecycle": "production",
  "remote_status": "pending",
  "repository": {
    "name": "<repo>",
    "owner": "<owner>",
    "url": "github.com/<owner>/<repo>",
    "visibility": "public|private"
  }
}
```

**S1 后只需更新**: `owner`（替换占位符）、`remote_url`、`remote_status` → `active`。