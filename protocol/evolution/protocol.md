# Evolution Protocol

> **版本**: 1.0
> **状态**: Frozen
> **所属协议族**: AISE Protocol 1.0

## 1. 目的

回答 Agent 进入项目后的问题：

> **How does the project evolve?**

Evolution Protocol 定义项目如何演进——Git 操作规则、版本管理、发布流程、Handoff 交接。这是连接 Git（Truth Snapshot Engine）和 AOS Runtime 的桥梁。

## 2. 核心原则

- Evolution 不替代 Git，而是 Git 的上层协议。
- Evolution 不碰业务内容，只验证规则。
- Protocol 描述 WHAT（规则），Runtime 负责 HOW（执行）。
- 所有修改必须留下审计痕迹。
- 协议升级与项目升级分离。

## 3. Git 治理规则

Evolution Protocol 约束 Git 操作，规则定义在 `git-policy.md` 中：

- `main` 分支 = READ-ONLY（仅接受 merge 提交，必须有 frozen tag）
- `develop` 分支 = 开发分支
- `pre-commit`：敏感文件拦截、CHANGELOG 同步、Memory 归属检查
- `pre-push`：远程检查、force push 拦截、main 无 tag 拦截
- Frozen tag 不可变

## 4. 标准命令语义

| 命令 | 语义 |
|------|------|
| `aise init` | 初始化 AOS 环境（AOS_HOME） |
| `aise bootstrap` | Agent 进入项目，按 Identity→State→Blueprint→Decision→Code 路径恢复 |
| `aise install` | 安装协议到 AOS_HOME |
| `aise release` | 从 frozen tag 生成 Protocol Artifact |
| `aise verify` | 检查项目是否符合 AOS 协议 |
| `aise authority` | 验证协议产品一致性 |
| `aise sync` | 同步协议标准与项目资产 |
| `aise archive` | 冻结版本、更新 Changelog、打 Tag、Push |

## 5. Runtime 模块

```
AOS Runtime
├── Core                ← 核心引擎（Interpreter, Registry, Lifecycle, Environment）
├── CLI                 ← 命令入口（薄包装，调用 Core）
├── Installer           ← 安装器
├── Adapter             ← 适配器（Git, Agent）
├── Release Manager     ← 发布管理
└── Package Manager     ← 包管理
```

## 6. Agent 修改边界

Agent 允许：
- 读取 `.agent-entry.json`、`.project/`、AOS_HOME
- 按 Mission Boundary 修改业务代码
- 写入 `.project/state/` 相关文件

Agent 禁止：
- 修改 `.agent-entry.json`
- 直接操作 `.git/`
- 读取或修改 `secrets/.env`
- 绕过 AOS Runtime 准入

## 7. Schema

参见: `schema.json`