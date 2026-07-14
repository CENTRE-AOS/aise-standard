# Repository Contract

> 版本：v1.1.0-beta.5
> 状态：Beta
> 适用范围：所有项目（Repository Generic）

## 1. 总则

本合约定义所有项目必须遵守的 Git 仓库基础设施标准。目标是在任何 AI、任何平台、任何项目之间建立统一的 Repository 认知，避免多源冲突与数据丢失。

## 2. Single Source of Truth

- **GitHub 是所有项目的唯一开发源（Single Source of Truth）。**
- `origin` 必须指向 GitHub。
- 所有开发行为（Pull / Push / Archive / Handoff / Sync）基于 GitHub。

## 3. AI 不拥有用户资产

- AI 不得自动生成或配置 `origin` URL。
- 远程配置必须由用户显式提供。
- 如未提供，状态为 `NOT CONFIGURED`，等待用户输入。

## 4. Workspace 类型

- **Local**：用户本地机器，长期保存用户数据、配置、数据库。
- **Cloud**：Trae Cloud / 其他 Cloud Agent 临时环境，不保存长期资产。

## 5. Cloud Workspace 约束

Cloud Workspace 只是 Execution Workspace：

```
GitHub Clone → Development → Testing → Review → Archive → Push origin → Handoff → Destroy Session
```

Cloud Session 删除前必须：
- Commit
- Push origin（GitHub）
- 生成 HANDOFF.md

禁止 Cloud Workspace 保存唯一数据。

## 6. Local Workspace 约束

Local Workspace 通过 Sync 升级到最新版本：

```
Sync → Launch → Application
```

## 7. Remote 配置

```
origin   git@github.com:<org>/<repo>.git (fetch)
origin   git@github.com:<org>/<repo>.git (push)
```

**约束**：origin 必须由用户提供，AI 不自动生成。

## 8. Git Governance 约束

所有 Git 操作受 `Git-Governance/` 层保护：
- pre-commit：文件边界 + 敏感文件 + CHANGELOG 同步
- commit-msg：消息格式强制
- pre-push：Remote / Force Push / Tags / 分支删除 / Archive
- post-merge：版本 / 依赖 / 代码变更检测

使用 `aise fetch` / `aise update` / `aise sync` 替代直接 `git pull`。

## 9. 禁止行为

- 禁止修改 `origin` 指向非 GitHub 地址（P1 可豁免）
- 禁止使用 `--force`、`--mirror`、`--all` 推送（P1 可豁免）
- 禁止删除 main/master 分支（P0 不可豁免）
- 禁止历史重写（已推送的 commit）
- 禁止在 Archive 中自动触发其他 Skill

## 10. 变更控制

进入 Frozen 后：
- 允许新增说明
- 禁止修改核心约定