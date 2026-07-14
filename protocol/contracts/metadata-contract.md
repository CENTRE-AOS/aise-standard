# Metadata Contract

> 版本：v1.1.0-beta.5
> 状态：Beta
> 适用范围：所有项目（Repository Generic）

## 1. 总则

本合约定义所有项目必须维护的元数据文件格式，确保 AI 之间可以无缝接续，无需依赖外部上下文。

## 2. 必维护文件（Autonomous Mode）

| 文件/目录 | 维护者 | 说明 |
|-----------|--------|------|
| AISE/VERSION | AI Agent | 语义化版本号，纯文本 |
| CHANGELOG.md | AI Agent | 版本变更记录 |
| PROJECT_BLUEPRINT.md | AI Agent | 项目蓝图与架构全景 |
| .project/ | AI Agent | 项目知识库（decisions/ audit/ mission/） |
| .handoff/ | AI Agent | Agent 交接上下文 |
| .sync/ | AI Agent | 环境同步状态 |
| AISE/Git-Governance/ | AI Agent | Git 原生出入口控制 |
| AISE/Registry/ | AI Agent | 注册表（version/compliance/skills/routing/git-governance） |

| 例外 | 维护者 | 说明 |
|------|--------|------|
| secrets/.env | 用户 | 密钥和私有凭证 |

## 3. VERSION 文件格式

```text
<MAJOR>.<MINOR>.<PATCH>[-<pre-release>]
```

示例：

```text
1.1.0-beta.2
```

## 4. CHANGELOG.md 格式

遵循 [Keep a Changelog](https://keepachangelog.com/)：

```markdown
# Changelog

## [v<version>] - <YYYY-MM-DD>

### Added
- <...>

### Changed
- <...>

### Fixed
- <...>
```

## 5. PROJECT_BLUEPRINT.md 格式

```markdown
# Project Blueprint

## 元信息
| 项目名称 | <name> | 当前版本 | v<version> | 存档次数 | <N> |

## 项目定位
<一句话描述>

## 技术栈
| 层级 | 技术 |

## 目录结构
<自动生成>

## 最近变更
### v<version> (YYYY-MM-DD) — <摘要>
- <变更点>

## 历史归档
| 版本 | 标签 | 日期 | 摘要 |

## Agent 交接记录
| 日期 | Agent | 类型 | 摘要 |
```

## 6. 审计日志格式

`.project/audit/` 目录下的 JSONL 日志：

```json
{"time":"2026-07-14T09:00:00","gate":"pre-commit","agent":"deepseek","branch":"main","result":"pass","duration_ms":120}
```

详见 `Policies/audit-policy.md`

## 7. 变更控制

进入 Frozen 后：
- 允许新增可选字段
- 禁止删除必填字段或改变其语义