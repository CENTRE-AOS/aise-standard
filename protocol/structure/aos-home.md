# AOS_HOME Environment Specification

> 版本: 1.0
> 状态: Frozen
> 所属: AOS Architecture Constitution

## 1. AOS_HOME 是什么

AOS_HOME 是 AOS 在当前机器上的环境基础设施目录。它是 CENTRE 的第一次实体化——协议、运行时、适配器、注册表的统一存放位置。

AOS_HOME 不是 Git 仓库。它是纯文件系统目录。

## 2. 目录结构

```
AOS_HOME/
├── protocol/                        ← 宪法（纯文本，零代码）
│   └── aise/
│       ├── v2.5.0/                  ← 已安装的历史版本
│       │   ├── CONSTITUTION.md
│       │   ├── identity/
│       │   ├── structure/
│       │   ├── state/
│       │   ├── evolution/
│       │   └── manifest.json
│       ├── v2.6.0/                  ← 当前版本
│       │   ├── CONSTITUTION.md
│       │   ├── NAMING.md
│       │   ├── identity/
│       │   ├── structure/
│       │   ├── state/
│       │   ├── evolution/
│       │   └── manifest.json
│       └── current → v2.6.0/        ← 符号链接
│
├── runtime/                         ← 执行器（有代码）
│   ├── core/                        ← 核心引擎
│   │   ├── interpreter.ps1          ← 统一解释器
│   │   ├── registry.ps1             ← 注册表管理
│   │   ├── lifecycle.ps1            ← 生命周期管理
│   │   └── environment.ps1          ← 环境管理
│   ├── cli/                         ← 命令入口（薄包装）
│   │   ├── aise-install.ps1
│   │   ├── aise-release.ps1
│   │   ├── aise-bootstrap.ps1
│   │   ├── aise-verify.ps1
│   │   ├── aise-authority.ps1
│   │   ├── aise-init.ps1
│   │   ├── aise-sync.ps1
│   │   ├── agent_exit.ps1
│   │   └── install-hooks.ps1
│   ├── installer/
│   └── package-manager/
│
├── adapters/                        ← 适配器（薄桥接层）
│   ├── git/
│   │   ├── pre-commit.ps1
│   │   ├── commit-msg.ps1
│   │   ├── pre-push.ps1
│   │   └── post-merge.ps1
│   └── agents/
│       ├── trae.ps1
│       ├── claude.ps1
│       └── cursor.ps1
│
├── registry.json                    ← 全局注册表
├── constitution/                    ← 宪章副本（约束 Runtime）
│   └── CONSTITUTION.md
├── cache/
└── logs/
```

## 3. 关键边界

- **protocol/** 里没有任何 `.ps1`、`.py`、`.exe` — 只有 `.md`、`.json`、`.yaml`
- **runtime/core/** 是核心引擎，CLI 只是薄入口
- **adapters/** 是桥，Hook 不包含规则，规则在 protocol/evolution/git-policy.md 中
- **registry.json** 记录已安装的协议版本和项目绑定

## 4. 平台路径

| 平台 | 路径 |
|------|------|
| Windows | `%USERPROFILE%\.aos` |
| Linux | `~/.aos` |
| macOS | `~/.aos` |
| Cloud | `/workspace/.aos` |

## 5. 环境变量

| 变量 | 值 | 说明 |
|------|-----|------|
| `AOS_HOME` | `<平台路径>` | AOS 环境根目录 |
| `AOS_PROTOCOL` | `AOS_HOME/protocol/aise/current` | 当前协议路径 |
| `AOS_RUNTIME` | `AOS_HOME/runtime` | 运行时路径 |

## 6. 初始化流程

```
aise init
    │
    ▼
检测 AOS_HOME 是否存在
    │
    ├── 不存在 → 创建 AOS_HOME 目录结构
    │              │
    │              ▼
    │         安装 Runtime（Core + CLI + Adapter）
    │              │
    │              ▼
    │         安装默认 Protocol（aise-install）
    │              │
    │              ▼
    │         注册到 registry.json
    │
    └── 存在 → 验证版本 → 可能需要更新
```

## 7. 与 Project 的关系

Project 不拥有 AOS_HOME。Project 只声明它需要的协议版本：

```json
// .project/aos.json
{
  "aos_protocol": "aise",
  "version": "2.6.0",
  "runtime_required": true
}
```

Agent 进入 Project 时，从 AOS_HOME 读取协议，验证 Project 声明的版本兼容性，通过后准入。