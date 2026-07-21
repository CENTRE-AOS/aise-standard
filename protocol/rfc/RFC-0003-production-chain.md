# RFC-0003: Release Boundary — Production Civilization

> RFC ID: RFC-0003
> Title: Release Boundary Architecture — Four Production Authorities + Dual Plane
> Version: 3.0.0-draft
> Status: Draft
> Type: Architecture Document（非 Contract）
> Date: 2026-07-21
> Category: Constitutional
> Author: CENTRE Protocol Authority
> Requires: RFC-0001 (Foundation Freeze), RFC-0002 (Kernel Contract)
> Scope: aise-standard (Protocol Layer only)
> Freeze Target: v2.0.0-frozen → v2.1.0-frozen

---

## 1. Abstract

RFC-0003 定义 CENTRE Production Civilization 的 Release Boundary Architecture：

1. **Four Production Authorities** — A0 Protocol / A1 Runtime Source / A2 Factory / A4 Installer（+ 一个 Archive Repository aos-migration）
2. **Dual Plane Model** — Source Plane ≠ Release Plane，下游只能消费 Release Plane
3. **Three Production Gates** — Identity Gate / Contract Gate / Supply Chain Gate

这标志着 CENTRE 从「多仓库工程管理」进入「分布式生产文明」。

**核心原则**: GitHub 是 Source Authority，不是本地磁盘。Release Plane 是唯一合法消费入口。没有 Gate 验证的产物不得进入生产链。

---

## 2. 问题陈述

### 2.1 当前状态

四个仓库已完成物理拆分：

```
E:\Development\AOS
├── aise-standard/    .git ✅   GitHub Remote ❌
├── aos-runtime/       .git ✅   GitHub Remote ❌
├── aos-factory-new/   .git ✅   GitHub Remote ❌
├── aos-installer/     .git ✅   GitHub Remote ❌
└── aos-migration/     .git ✅   GitHub Remote ❌
```

但当前仍处于 **Developer Machine Authority**，未达到 **Distributed Production Authority**。

缺失项：

| 缺失 | 说明 |
|------|------|
| GitHub Remote | 无远程仓库。Source Authority 在本地磁盘而非 GitHub |
| CI Gate | 无门禁。任何 commit 可直接被下游视为"可用" |
| Release Plane | 无 tag / release / artifact。Source Plane 和 Release Plane 未分离 |
| Supply Chain Lock | 无依赖锁定。Factory 可随时拉取任意 Runtime commit |
| Remote Identity | Agent 不知道自己在哪个远程仓库上工作 |

### 2.2 核心问题

不是「缺几个 tag、缺 CI」，而是：

```
四个本地目录 ≠ 分布式生产节点
```

**本地磁盘是开发空间，不是 Source Authority。GitHub 才是 Source Authority。**

### 2.3 目标

1. 建立 GitHub Repository Boundary — 五个仓库严格一一对应
2. 实现 Dual Plane Model — Source Plane 和 Release Plane 完全分离
3. 建立 Three Production Gates — Identity / Contract / Supply Chain
4. 定义 Remote Identity Contract — Agent 启动时验证远程身份
5. 创建第一批 Release — 四个仓库首次发布

---

## 3. GitHub Repository Boundary

### 3.1 生产 Authority 映射（四生产 + 一归档）

```
生产 Authority（Production Chain）:
─────────────────────────────────
aise-standard/          ←→         github.com/<org>/aise-standard        (A0 Protocol)
aos-runtime/             ←→         github.com/<org>/aos-runtime          (A1 Source)
aos-factory-new/         ←→         github.com/<org>/aos-factory           (A2 Build)
aos-installer/           ←→         github.com/<org>/aos-installer        (A4 Deploy)

归档 Authority（Civilization Archive）:
─────────────────────────────────
aos-migration/           ←→         github.com/<org>/aos-migration        (Archive)
```

每个仓库拥有完整的独立身份：

```
<repo>/
├── .git/                        # 独立 Git
├── .git/config (remote origin)  # 独立 GitHub Remote
├── VERSION                      # 独立版本号
├── CHANGELOG.md                 # 独立变更日志
├── AGENTS.md                    # 独立 Agent 行为规则
├── AGENT_CONTEXT.md             # 独立身份契约
├── .handoff/                    # 独立交接
├── .github/
│   ├── workflows/
│   │   ├── identity-gate.yml    # Gate 1: 身份一致性
│   │   ├── contract-gate.yml    # Gate 2: 契约合规
│   │   └── release.yml          # Release Pipeline
│   ├── remote-identity.json     # Remote Identity Contract
│   └── dependency-lock.json     # Supply Chain Lock
└── src/
```

### 3.2 禁止模式

```
FORBIDDEN:
  本地磁盘 = Source Authority          ← GitHub 才是
  git init --bare 在本地                ← 必须 push 到 GitHub
  无 remote origin                      ← 无法验证远程一致性
  Monorepo 包含多个 Authority           ← 物理隔离
```

---

## 4. Dual Plane Model

### 4.1 定义

```
Source Plane                          Release Plane
─────────────────                     ─────────────────
git repository                        git tag v<version>
branch (main/develop/feature/fix)     GitHub Release
git commit                            signed artifact
PR / review / merge                   Release Notes
VERSION file                          artifact.manifest.json

开发者可以修改                       开发者不可修改（只读）
下游不能消费                         下游只能消费 Release Plane
```

### 4.2 转换规则

```
Source Plane → Release Plane 必须经过 CI Gate：

main HEAD
    │
    │ git tag v<version>
    ▼
Identity Gate ──fail──▶ REJECT
    │
    ▼
Contract Gate ──fail──▶ REJECT
    │
    ▼
Supply Chain Gate ──fail──▶ REJECT
    │
    ▼
Release Plane
    │
    ├── GitHub Release
    └── Signed Artifact
```

### 4.3 消费规则

| 消费者 | 允许消费 | 禁止消费 |
|--------|---------|---------|
| Factory | Protocol Release (v2.0.0-frozen), Runtime Release (v3.2.0) | Protocol main branch, Runtime main branch |
| Installer | Factory Signed Artifact | Runtime Source, Factory main branch |
| Runtime Instance | Installed Artifact (via Installer) | Any Source Plane resource |

---

## 5. Three Production Gates

### 5.1 Gate 1: Identity Gate

**检查**: VERSION == tag == release 三者一致。

```
VERSION file:    3.2.0
git tag:         v3.2.0
release name:    3.2.0

→ 不一致 → REJECT
```

**CI 实现**:

```yaml
name: Identity Gate
on:
  push:
    tags: ['v*']
jobs:
  validate:
    runs-on: ubuntu-latest
    steps:
      - name: Check VERSION == tag
        run: |
          V=$(cat VERSION)
          T=${GITHUB_REF#refs/tags/v}
          if [ "$V" != "$T" ]; then
            echo "VERSION=$V != tag=$T" && exit 1
          fi
```

### 5.2 Gate 2: Contract Gate

**检查**: 禁止跨仓库直接引用 Source Plane。

```
FORBIDDEN:
  Factory → git clone aos-runtime main branch
  Factory → git checkout aos-runtime develop

ALLOWED:
  Factory → download aos-runtime Release v3.2.0 artifact
```

**CI 实现**:

```yaml
- name: Contract Gate — no source-plane references
  run: |
    if grep -r "git clone.*aos-runtime" .; then
      echo "FORBIDDEN: direct source reference" && exit 1
    fi
    if grep -r "git checkout.*aise-standard" .; then
      echo "FORBIDDEN: direct protocol source ref" && exit 1
    fi
```

### 5.3 Gate 3: Supply Chain Gate

**检查**: `dependency-lock.json` 中的版本与实际消费的版本一致。

```json
{
  "repository": "aos-factory",
  "version": "1.1.0",
  "dependencies": {
    "protocol": {
      "repository": "aise-standard",
      "version": "2.0.0-frozen",
      "source": "github-release",
      "release_tag": "v2.0.0-frozen"
    },
    "runtime": {
      "repository": "aos-runtime",
      "version": "3.2.0",
      "source": "github-release",
      "release_tag": "v3.2.0"
    }
  }
}
```

**CI 验证**:

```yaml
- name: Supply Chain Gate — verify dependency lock
  run: |
    for dep in $(jq -r '.dependencies | keys[]' dependency-lock.json); do
      LOCKED=$(jq -r ".dependencies.$dep.version" dependency-lock.json)
      echo "Checking $dep: expected=$LOCKED"
    done
```

---

## 6. Remote Identity Contract

### 6.1 定义

每个仓库的 `.github/remote-identity.json`：

```json
{
  "repository": "github.com/<org>/aos-runtime",
  "authority": "runtime",
  "authority_level": "A1",
  "release_source": "github-release",
  "allowed_consumer": ["aos-factory"],
  "bootstrap_contract": {
    "required_files": ["AGENTS.md", "AGENT_CONTEXT.md"],
    "version": "1.0.0"
  },
  "supply_chain": {
    "upstream": {
      "protocol": {
        "repository": "aise-standard",
        "version": "2.0.0-frozen"
      }
    },
    "downstream": {
      "factory": {
        "repository": "aos-factory",
        "consumes": "runtime-release"
      }
    }
  }
}
```

### 6.2 Agent Bootstrap 规则

Agent 进入任何仓库时：

```
Step 1: 读取 AGENTS.md
Step 2: 读取 AGENT_CONTEXT.md
Step 3: 读取 .github/remote-identity.json
Step 4: 验证 local HEAD == remote HEAD
Step 5: 验证 dependency-lock.json 与 remote-identity.json 一致
Step 6: 确定 Authority Level 和 Mutation Scope
Step 7: 进入工作
```

### 6.3 Remote Integrity Check

```
每次 Agent 工作前：

git fetch origin
if local HEAD != origin/main HEAD:
    → SYNC REQUIRED
    → 禁止修改
```

---

## 7. 供应链完整模型

```
┌─────────────────────────────────────────────────────────────┐
│                    CENTRE Production Civilization            │
│                                                              │
│  Human Authority                                              │
│       │                                                       │
│       │ Review / Approve                                      │
│       ▼                                                       │
│  ┌─────────────────┐                                         │
│  │  Source Plane    │  GitHub Repository                      │
│  │  ─────────────  │  branch / commit / PR / merge           │
│  │  aise-standard   │                                         │
│  │  aos-runtime     │  ← Developer Authority                  │
│  │  aos-factory     │                                         │
│  │  aos-installer   │                                         │
│  └────────┬────────┘                                         │
│           │                                                   │
│           │ git tag v<version>                                │
│           ▼                                                   │
│  ┌─────────────────┐                                         │
│  │  CI Gates        │  Identity / Contract / Supply Chain     │
│  │  ─────────────  │                                         │
│  │  Gate 1: Identity│  VERSION == tag == release              │
│  │  Gate 2: Contract│  禁止 Source Plane 引用                 │
│  │  Gate 3: Supply  │  dependency-lock 验证                   │
│  └────────┬────────┘                                         │
│           │                                                   │
│           │ PASS                                              │
│           ▼                                                   │
│  ┌─────────────────┐                                         │
│  │  Release Plane   │  GitHub Release                         │
│  │  ─────────────  │  tag / release / artifact               │
│  │                  │  ← Release Authority（不可修改）        │
│  └────────┬────────┘                                         │
│           │                                                   │
│           │ Signed Artifact                                   │
│           ▼                                                   │
│  ┌─────────────────┐                                         │
│  │  Artifact Plane  │  Artifact Registry                      │
│  │  ─────────────  │  ← Artifact Authority（存储分发）        │
│  └────────┬────────┘                                         │
│           │                                                   │
│           │ Published Artifact                                 │
│           ▼                                                   │
│  ┌─────────────────┐                                         │
│  │  Consumer Plane  │  Installer → Runtime Instance           │
│  │  ─────────────  │  ← Consumer Authority（验证加载）        │
│  └─────────────────┘                                         │
│                                                              │
│  =============================================               │
│  每条边界不可跨越。每个 Authority 不可混合。                  │
└─────────────────────────────────────────────────────────────┘
```

---

## 8. 不可违背约束

| # | 约束 |
|---|------|
| 1 | GitHub 是 Source Authority — 本地磁盘是开发空间 |
| 2 | Source Plane ≠ Release Plane — 下游只能消费 Release Plane |
| 3 | 无 Gate 不 Release — 三个 Gate 任一失败则拒绝 |
| 4 | Tag ≠ Release — Tag 是入口，Release 是完整产物 |
| 5 | dependency-lock 必须验证 — 不允许浮动依赖 |
| 6 | Agent Bootstrap 必须检查 Remote Identity |
| 7 | 跨仓库引用只能通过 Release Plane — 禁止 git clone Source Plane |
| 8 | 每个仓库独立的五项资产：.git / remote / VERSION / CI / dependency-lock |

---

## 9. Implementation Path

### 9.1 阶段

```
Phase 2.7: Release Boundary Design        ← RFC-0003 v2.0（已完成）
Phase 2.8: Production Civilization Upgrade ← 当前 RFC v3.0
Phase 3.0: Artifact Civilization           ← 下一阶段
```

### 9.2 Phase 2.8 步骤（S0-S5）

| Step | 内容 | 产出 |
|:----:|------|------|
| **S0** | **Repository Identity Freeze** — 四仓库冻结 remote-identity.json | Identity 锁定 |
| S1 | 创建 4 个 GitHub 生产仓库 + 1 个归档仓库 | GitHub Repository Boundary |
| S2 | Remote Binding — git remote add + push + 验证 local==remote | 远程绑定 |
| S3 | Release Plane Bootstrap — tag + release + CI | Release Plane |
| S4 | Production Gates — identity-gate / contract-gate / release | CI Gates |
| S5 | Supply Chain Validation — 完整闭环验证 | Production Chain Verified |

---

## 10. References

- RFC-0001: CENTRE Foundation Freeze v3.0.0
- RFC-0002: CENTRE Kernel Contract
- [release-boundary-implementation-plan.md](file:///e:/Development/AOS/aise-standard/protocol/rfc/release-boundary-implementation-plan.md)