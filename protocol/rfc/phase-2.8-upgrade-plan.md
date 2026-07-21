# Phase 2.8 — Production Civilization Upgrade Plan

> Date: 2026-07-21
> Phase: 2.8 — Repository Authority Establishment
> Precondition: Phase 2.7 Release Boundary Design COMPLETE
> RFC: RFC-0003 v3.0.0-draft (Release Boundary Architecture)
> Target: 四生产 Authority + 一归档 Repository

---

## 1. 升级目标

从 **Developer Machine Authority** 升级到 **Distributed Production Authority**。

```
升级前:  四个本地目录 + 独立 .git（物理拆分）
升级后:  四个 GitHub 生产 Authority + Release Plane + CI Gates
```

## 2. Authority 模型

### 2.1 四生产 Authority（Production Chain）

| Authority | 本地目录 | GitHub 仓库名 | 级别 |
|-----------|---------|-------------|:----:|
| Protocol Authority | aise-standard | aise-standard | A0 |
| Source Authority | aos-runtime | aos-runtime | A1 |
| Build Authority | aos-factory-new | aos-factory | A2 |
| Install Authority | aos-installer | aos-installer | A4 |

### 2.2 一归档 Repository（Civilization Archive）

| Repository | 本地目录 | GitHub 仓库名 | 说明 |
|-----------|---------|-------------|------|
| Migration Archive | aos-migration | aos-migration | 不属于生产链 |

---

## 3. 执行路径 S0→S5

### S0 — Identity Freeze（已完成）

S0 → S0.5 → S0.6 → S0.7 全链路冻结：

| Step | 内容 | 状态 |
|:----:|------|:----:|
| S0 | Repository Identity Freeze | ✅ |
| S0.5 | Repository Governance Freeze | ✅ |
| S0.6 | Remote Ownership Freeze | ✅ |
| **S0.7** | **Remote Identity Schema Freeze** | ✅ |
| **S0.8** | **Production Bootstrap Contract Freeze** | ✅ |
| **S0.9** | **Dependency Lock Freeze** | ✅ |
| **S0.10** | **Remote Naming & Ownership Freeze** | ✅ |
| **S0.11** | **Namespace Authority Freeze** | ✅ |

每个仓库已生成完整 `.github/remote-identity.json`，包含：identity / authority / lifecycle / remote_status / branch_policy / supply_chain / local_path。

**S0.10 冻结内容**：
- 仓库名映射表（仅 `aos-factory-new` 本地目录 ≠ GitHub `aos-factory`）
- GitHub Organization 推荐：`CENTRE-AOS`
- S1 Owner 替换规则（`<github-org-or-user>` → 实际 Owner）
- 策略文件：`protocol/rfc/naming-ownership-policy.md` v1.1.0-frozen

**S0.11 冻结内容**：
- Namespace Owner：`CENTRE-AOS` (Organization) — 不可降级
- 四个 `remote-identity.json` 中 `repository.owner` = `CENTRE-AOS`
- `repository.url` = `github.com/CENTRE-AOS/<repo>`
- 最终 Namespace：`github.com/CENTRE-AOS/` 下六个仓库（四生产 + 一 Archive + 一 Reserved）

### S1 — GitHub Repository Creation

创建 4 个生产仓库：

| 创建 | Repository | Lifecycle |
|:----:|-----------|-----------|
| ✅ | aise-standard | production |
| ✅ | aos-runtime | production |
| ✅ | aos-factory | production |
| ✅ | aos-installer | production |

**不创建**: aos-migration（Archive，单独处理）、artifact-registry（Phase 3.0）。

### S2 — Remote Binding

```
git remote add origin git@github.com:<org>/<repo>.git
git push --set-upstream origin main
验证 local HEAD == remote HEAD
```

### S3 — Release Plane Bootstrap

```
git tag v<version>
git push origin v<version>
→ GitHub Release
```

### S4 — Production Gates

每个仓库 `.github/workflows/`：
- `identity-gate.yml`
- `contract-gate.yml`
- `release.yml`

### S5 — Supply Chain Validation

验证完整闭环。

---

## 4. 资产清单

| 资产 | aise-standard | aos-runtime | aos-factory-new | aos-installer | aos-migration |
|------|:-----------:|:----------:|:--------------:|:------------:|:------------:|
| .git | ✅ | ✅ | ✅ | ✅ | ✅ |
| GitHub Remote | S1 | S1 | S1 | S1 | S1 |
| VERSION | 2.0.0-frozen | 3.2.0 | 1.1.0 | 1.1.0 | — |
| remote-identity.json | **S0** | **S0** | **S0** | **S0** | — |
| dependency-lock.json | S2 | S2 | S2 | S2 | — |
| identity-gate.yml | S4 | S4 | S4 | S4 | — |
| contract-gate.yml | S4 | S4 | S4 | S4 | — |
| release.yml | S4 | S4 | S4 | S4 | — |
| Release (GitHub) | S3 | S3 | S3 | S3 | — |

---

## 5. 约束

- 不修改核心代码
- 不创建 Artifact Registry
- 不创建新 Contract
- 每个仓库独立操作
- aos-migration 只做 remote + push
- **先 S0 Identity Freeze，后 S1 GitHub 创建**