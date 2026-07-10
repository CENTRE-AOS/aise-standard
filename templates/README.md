# <project-name>

> 一句话描述项目。

## 简介

<项目详细介绍。>

## 快速开始

```bash
git clone git@github.com:<org>/<repo>.git
cd <repo>
```

## 开发工作流

本项目遵循 AISE（AI Software Engineering Standard）：

```text
Idea → Architecture → Implementation → Testing → Archive → Publish → Mirror → Handoff → Sync Workspace → Run → Feedback → Idea
```

## Repository Policy

```yaml
Single Source of Truth: GitHub
origin: GitHub
Workspace: Local | Cloud
Archive: Commit → Tag → Push origin
Handoff: Generate HANDOFF.md
Sync Workspace: Pull origin
Release Mirror（可选）: GitHub → Gitee
Forbidden: Gitee → GitHub 任何反向同步
```

## 目录结构

```text
<project>/
├── src/
├── tests/
├── docs/
├── .project/
├── .sync/
├── .backup/
├── CHANGELOG.md
├── PROJECT_BLUEPRINT.md
└── README.md
```

## 贡献指南

1. 所有开发基于 GitHub `origin`。
2. 提交前遵循 AISE Archive 流程。
3. Cloud Workspace 销毁前必须 Commit、Push、HANDOFF。

## 许可证

<许可证信息。>
