# Remote Naming & Ownership Policy

> Version: 1.1.0-frozen
> Date: 2026-07-21
> Phase: S0.10 — Remote Naming & Ownership Freeze / S0.11 — Namespace Authority Freeze
> Status: FROZEN

---

## 1. Repository Name Mapping

| Authority | Local Path | GitHub Name | Diff? |
|-----------|-----------|-------------|:----:|
| A0 Protocol | aise-standard | aise-standard | 否 |
| A1 Source | aos-runtime | aos-runtime | 否 |
| A2 Build | **aos-factory-new** | **aos-factory** | **是** |
| A4 Install | aos-installer | aos-installer | 否 |
| Archive | aos-migration | aos-migration | 否 |

**注意**: `aos-factory-new` 是迁移过渡名为本地目录。GitHub 仓库使用最终名 `aos-factory`。local_path ≠ remote name 已记录在 `remote-identity.json` 中。

---

## 2. GitHub Organization / Owner

### 2.1 已冻结决策

| 字段 | 值 | 状态 |
|------|-----|:----:|
| Namespace | Organization | **FROZEN** |
| Owner | `CENTRE-AOS` | **FROZEN** |
| 决定时间 | 2026-07-21 (S0.11) | — |

### 2.2 最终 Namespace 结构

```
github.com/CENTRE-AOS/
├── aise-standard          # A0 Protocol Authority — public
├── aos-runtime            # A1 Source Authority — public
├── aos-factory            # A2 Build Authority — private
├── aos-installer          # A4 Install Authority — private
├── aos-migration          # Archive — public
└── artifact-registry      # Phase 3: A3 Artifact Authority
```

### 2.3 选择理由

- **产品化**: 统一命名空间，独立于个人账号
- **社区化**: 支持未来外部开发者、Agent Plugin、Runtime Provider 加入
- **权限管理**: Organization 级别的团队访问控制
- **Artifact 信任链**: `CENTRE-AOS/aos-factory` → sign artifact → `CENTRE-AOS` registry → tenant
- **生态扩展**: 未来可自然添加 ecosystem projects

---

## 3. S1 Owner 验证规则

S1 创建 GitHub 时，验证所有 `remote-identity.json` 中 `repository.owner` 已为 `CENTRE-AOS`（S0.11 已冻结）。

验证范围：
- `repository.owner` === `CENTRE-AOS`
- `repository.url` === `github.com/CENTRE-AOS/<repo>`

---

## 4. 不可违背约束

1. GitHub 仓库名与 `remote-identity.json` 中 `name_frozen` 一致
2. `aos-factory-new` 本地目录名保留，GitHub 使用 `aos-factory`
3. Namespace Owner 已冻结为 `CENTRE-AOS` (Organization)，不可降级为个人账号
4. S1 创建时验证 Owner 一致性，不执行替换
5. Archive (aos-migration) 不在 S1 创建范围内