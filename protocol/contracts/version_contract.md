# Version Contract

> 版本：v1.0
> 状态：Frozen
> 适用范围：所有项目

## 1. 总则

本合约定义所有项目的版本号格式、递增规则与标签规范，确保跨项目、跨 AI 的版本语义一致。

## 2. 版本号格式

采用语义化版本控制（Semantic Versioning）：

```text
MAJOR.MINOR.PATCH[-prerelease][+build]
```

示例：

- `v1.2.3` — 稳定版本
- `v1.2.3-alpha` — 预发布版本
- `v1.2.3-alpha.1` — 带迭代号的预发布版本
- `v1.2.3+20260710` — 带构建元数据

## 3. 版本递增规则

| 变更类型 | 递增字段 | 示例 |
|---------|---------|------|
| 破坏性变更 | MAJOR | `v1.2.3` → `v2.0.0` |
| 新增功能 | MINOR | `v1.2.3` → `v1.3.0` |
| Bug 修复 | PATCH | `v1.2.3` → `v1.2.4` |
| 预发布迭代 | prerelease | `v1.3.0-alpha` → `v1.3.0-alpha.1` |

## 4. 预发布标识

常用预发布标识：

- `alpha` — 内部测试，不稳定
- `beta` — 公开测试，功能基本完整
- `rc` — Release Candidate，候选发布版本

## 5. Git Tag 规范

- 每个发布版本必须对应一个 Git Tag。
- Tag 名称与版本号一致，前缀 `v`。
- 示例：`git tag v1.2.3`
- Tag 必须指向已提交的 Commit。

## 6. 文档同步

版本号变更时，必须同步更新：

- `CHANGELOG.md`
- `PROJECT_BLUEPRINT.md`
- Git Tag

## 7. 版本比较

版本比较规则遵循 Semantic Versioning：

```text
v1.2.3 < v1.3.0 < v2.0.0
v1.3.0-alpha < v1.3.0-beta < v1.3.0-rc < v1.3.0
```

## 8. 变更控制

本合约进入 Frozen 状态后：

- 允许新增预发布标识或构建元数据说明。
- 禁止修改版本号基本格式与比较规则。
