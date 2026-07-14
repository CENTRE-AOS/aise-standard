# Git Policy

> 版本：v1.1.0-rc.1
> 状态：Frozen
> 适用范围：所有项目（Repository Generic）

## 1. 总则

本策略定义 Git 操作的约束，遵循 Single Source of Truth (GitHub)。

## 2. 单源真理

- **GitHub 是唯一开发源**
- `origin` 必须指向 GitHub
- 所有 Pull/Push/Archive/Handoff/Sync 都基于 GitHub
- 不允许其他开发源

## 3. 远程配置

```
origin   git@github.com:<org>/<repo>.git (fetch)
origin   git@github.com:<org>/<repo>.git (push)
```

**约束**：

- 协议：SSH 首选，HTTPS 备选
- AI 不自动配置 origin（必须由用户提供）
- 禁止 origin 指向非 GitHub

## 4. 分支策略

| 规则 | 内容 |
|------|------|
| 禁止删除 | 不得删除 main/master |
| 保护分支 | main/master 受保护 |
| 禁止 Force Push | 禁止 force push 到受保护分支 |
| Push 范围 | 仅推送当前分支 + 当前版本标签 |
| Tag 推送 | `git push origin <current-tag>`，禁止 `--tags` |
| 默认分支 | main |

## 5. 提交策略

**提交格式强制**：

```
<type>(<scope>): <summary> [test:<pass>/<total>] [hint:<keyword>] (by AI-<model>)
```

**类型**：

- `feat` — 新增功能
- `fix` — Bug 修复
- `chore` — 基础设施变更
- `docs` — 文档变更
- `refactor` — 重构
- `test` — 测试变更
- `infra` — 基础设施
- `perf` — 性能优化

**文件边界**：

- 仅提交 CHANGELOG.md、PROJECT_BLUEPRINT.md、.project/ 及已跟踪文件
- 禁止提交非必要文件

**暂存**：

```bash
git add -u
git add CHANGELOG.md PROJECT_BLUEPRINT.md .project/
```

## 6. 标签策略

- 标签格式：`v<M>.<m>.<p>`
- 标签必须指向已提交的 commit
- 推送仅推送当前标签
- 禁止修改已推送的标签

## 7. 凭证策略

- SSH key：`~/.ssh/id_ed25519`
- GitHub CLI：`gh auth` 认证
- 凭证助手：`manager + gh auth git-credential`
- 禁止：代码中存放 PAT

## 8. 变更控制

进入 Frozen 后：

- 允许新增分支策略。
- 禁止修改核心约定。