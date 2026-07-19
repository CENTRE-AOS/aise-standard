# AI Agent Entry Protocol

进入本项目的 AI Agent 必须首先读取：

1. `SYSTEM.md`
2. `Contracts/`
3. `Skills/`
4. `PROJECT_BLUEPRINT.md`
5. `CHANGELOG.md`

**未完成读取前禁止修改任何文件。**

## 支持 Agent

| Agent | 入口文件 | 说明 |
|-------|---------|------|
| Codex | `AGENTS.md` | 自动读取项目级规则 |
| Claude Code | `CLAUDE.md` | 启动时自动加载 |
| Trae | `.trae/rules/aise.md` | 项目规则文件 |
| Gemini CLI | `README.md` (顶部) | 通过项目 README 注入 |
| 其他 | `README.md` (顶部) | 通用入口协议 |