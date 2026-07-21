# Release Boundary Implementation Plan v2.0

> Date: 2026-07-21
> Phase: 2.8 — Production Civilization Upgrade
> Status: Draft
> RFC: RFC-0003 v3.0.0-draft
> Predecessor: v1.1 (Phase 2.7 Release Identity Freeze — SUPERSEDED)

---

## 修订记录

| 版本 | Phase | 修订 |
|------|-------|------|
| v1.0 | 2.7 | 初始审计 + P0-P3 优先级 |
| v1.1 | 2.7 | P0 升级为 Release Identity；执行顺序改为生产流；Runtime Consumer 修正 |
| **v2.0** | **2.8** | **从本地 Release Identity 升级到 GitHub Distributed Production Authority；新增 GitHub Boundary / Dual Plane / CI Gates / Remote Identity** |

---

## 1. 审计总览（v2.0）

| 仓库 | VERSION | .git | GitHub Remote | CI Gate | Release | dependency-lock |
|------|---------|:----:|:-------------:|:-------:|:-------:|:---------------:|
| aise-standard | 2.0.0-frozen | ✅ | ❌ | ❌ | ❌ | ❌ |
| aos-runtime | 3.2.0 | ✅ | ❌ | ❌ | ❌ | ❌ |
| aos-factory-new | 1.1.0 | ✅ | ❌ | ❌ | ❌ | ❌ |
| aos-installer | 1.1.0 | ✅ | ❌ | ❌ | ❌ | ❌ |
| aos-migration | — | ✅ | ❌ | — | — | — |

**v1.1 只看到了「缺 tag、缺 CI」。v2.0 发现根本问题：本地磁盘不是 Source Authority，GitHub 才是。**

---

## 2. 升级路径

```
Phase 2.7 (v1.1)          →     Phase 2.8 (v2.0)
Local Release Identity    →     Distributed Production Authority

git tag (本地)             →     git tag + push + GitHub Release
release.yaml              →     .github/workflows/ (CI Gates)
VERSION 一致性             →     remote-identity.json + dependency-lock.json
四仓库独立 tag             →     五仓库 GitHub Repository Boundary
```

---

## 3. 执行步骤

### S1: GitHub Repository Boundary

五个仓库添加 remote origin 并 push：

```
git remote add origin git@github.com:<org>/<repo>.git
git push --set-upstream origin main
git push --all
```

### S2: Remote Identity + Dependency Lock

每个仓库创建：

**`.github/remote-identity.json`**:
- `repository`: GitHub URL
- `authority`: protocol | runtime | factory | installer
- `authority_level`: A0 | A1 | A2 | A4
- `allowed_consumer`: [下游仓库列表]
- `bootstrap_contract`: { required_files, version }
- `supply_chain`: { upstream, downstream }

**`.github/dependency-lock.json`**:
- `dependencies.<name>.version`: 锁定版本
- `dependencies.<name>.source`: "github-release"
- `dependencies.<name>.release_tag`: 对应 tag

### S3: CI Gates

每个仓库创建三个 Gate：

| Gate | 文件 | 触发条件 | 检查内容 |
|------|------|---------|---------|
| Identity | identity-gate.yml | push tag v* | VERSION == tag == release |
| Contract | contract-gate.yml | push tag v* | 禁止 Source Plane 引用 |
| Release | release.yml | push tag v* | 构建 + 签名 + 发布 |

### S4: First Release

```
git tag v<version>
git push origin v<version>
→ CI Gates 自动执行
→ GitHub Release 创建
```

| 仓库 | Tag | Artifact 类型 |
|------|-----|-------------|
| aise-standard | v2.0.0-frozen | GitHub Release（治理制品） |
| aos-runtime | v3.2.0 | GitHub Release（源码稳定版） |
| aos-factory-new | v1.1.0 | GitHub Release + Signed Artifact |
| aos-installer | v1.1.0 | GitHub Release（部署工具） |

### S5: Supply Chain Verification

验证完整闭环：
```
aise-standard v2.0.0-frozen  →  Factory 可消费 ✅
aos-runtime v3.2.0           →  Factory 可消费 ✅
Factory v1.1.0 Artifact      →  Registry 可存储 ✅
Registry Artifact            →  Installer 可获取 ✅
```

---

## 4. 资产清单

| 资产 | aise-standard | aos-runtime | aos-factory-new | aos-installer |
|------|:-----------:|:----------:|:--------------:|:------------:|
| remote-identity.json | ✅ | ✅ | ✅ | ✅ |
| dependency-lock.json | ✅ | ✅ | ✅ | ✅ |
| identity-gate.yml | ✅ | ✅ | ✅ | ✅ |
| contract-gate.yml | ✅ | ✅ | ✅ | ✅ |
| release.yml | ✅ | ✅ | ✅ | ✅ |
| GitHub Release | v2.0.0-frozen | v3.2.0 | v1.1.0 | v1.1.0 |

---

## 5. 约束

- 不修改核心代码
- 不创建 Artifact Registry
- 不创建新 Contract
- 每个仓库独立操作
- aos-migration 只做 remote + push
- 标签模板见 [remote-identity.template.json](file:///e:/Development/AOS/aise-standard/protocol/rfc/templates/remote-identity.template.json) 和 [dependency-lock.template.json](file:///e:/Development/AOS/aise-standard/protocol/rfc/templates/dependency-lock.template.json)