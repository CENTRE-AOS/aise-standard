# Metadata Contract

> 版本：v2.5.0frozen
> 状态：Frozen
> 适用范围：所有项目（Repository Generic）
> 协议：AISE Protocol 1.0

## 1. 总则

本合约定义所有遵循 AISE Protocol 1.0 的项目必须维护的元数据文件格式，确保任何 Agent 进入项目后无需外部上下文即可理解项目结构与当前状态。

## 2. 必维护文件（Autonomous Mode）

| 文件/目录 | 维护者 | 说明 |
|-----------|--------|------|
| `.agent-entry.json` | Protocol Runtime | Protocol Manifest，声明项目遵循 AISE Protocol 1.0 |
| `VERSION` | AI Agent | 项目版本号，语义化版本或冻结版本（如 `v2.5.0frozen`） |
| `CHANGELOG.md` | AI Agent | 版本变更记录 |
| `PROJECT_BLUEPRINT.md` | AI Agent | 项目蓝图与架构全景 |
| `.project/context/` | AI Agent | 项目当前状态（state.json、mission.json、timeline.jsonl） |
| `.project/memory/` | AI Agent | 项目级知识资产（knowledge、patterns、glossary 索引） |
| `.project/decisions/` | AI Agent | ADR 架构决策记录 |
| `.project/architecture/` | AI Agent | 架构文档 |
| `.project/journal/` | AI Agent | Agent 活动日志 |
| `.project/audit/` | AI Agent | 合规审计日志 |

| 例外 | 维护者 | 说明 |
|------|--------|------|
| `secrets/.env` | 用户 | 密钥和私有凭证，Agent 禁止读取或修改 |

## 3. 禁止作为项目元数据的结构

以下结构**禁止**出现在普通项目中：

- `AISE/` 目录（协议规范由 aise-standard 集中维护）
- `.agent/` 目录（Agent 身份由 Governance Runtime 管理）
- `.trae/`、`.claude/`、`.workbuddy/` 等 Agent 私有目录
- `.handoff/`、`.sync/`、`.backup/` 等旧 Runtime 目录

## 4. VERSION 文件格式

```text
<MAJOR>.<MINOR>.<PATCH>[-<pre-release>]
```

冻结版本示例：

```text
2.5.0frozen
```

## 5. CHANGELOG.md 格式

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

## 6. PROJECT_BLUEPRINT.md 格式

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
```

## 7. 审计日志格式

`.project/audit/` 目录下的 JSONL 日志：

```json
{"time":"2026-07-14T09:00:00","gate":"pre-commit","agent":"deepseek","branch":"main","result":"pass","duration_ms":120}
```

详见 `Policies/audit-policy.md`

## 8. 变更控制

进入 Frozen 后：
- 允许新增可选字段
- 禁止删除必填字段或改变其语义
