# Runtime Boundary Definition

> 版本: 1.0
> 状态: Frozen
> 所属: AOS Architecture Constitution

## 1. Runtime 是什么

AOS Runtime 是 AOS 协议的执行环境。它负责 HOW（执行），不负责 WHAT（定义）。Protocol 是宪法，Runtime 是司法系统。

## 2. Runtime 模块边界

### 2.1 Core（核心引擎）

```
core/
├── interpreter.ps1     ← 统一解释器，所有 CLI 命令调用此入口
├── registry.ps1        ← 版本注册表管理
├── lifecycle.ps1       ← 项目生命周期管理
└── environment.ps1     ← AOS_HOME 环境管理
```

**职责**：
- Protocol 加载与解析
- 版本兼容性验证
- 生命周期状态机
- 环境变量管理

**不在 Core 中的**：
- Git 操作（属于 Adapter）
- 安装逻辑（属于 Installer）
- 产品发布（属于 Release Manager）

### 2.2 CLI（命令入口）

CLI 是薄包装器，调用 Core/Interpreter 执行实际逻辑。

**职责**：提供用户/Agent 命令入口，解析参数，调用 Interpreter

**不在 CLI 中的**：业务逻辑、规则解析、状态管理（全部在 Core 中）

### 2.3 Installer（安装器）

**职责**：安装协议到 AOS_HOME，初始化环境

**不在 Installer 中的**：协议验证（属于 Core）、项目 Bootstrap（属于 CLI）

### 2.4 Adapter（适配器）

**职责**：连接 Runtime 与外部系统（Git、Agent 平台）

**关键原则**：Adapter 是薄桥接层，不包含规则。规则在 Protocol 中。

### 2.5 Release Manager（发布管理）

**职责**：从 frozen tag 生成 Protocol Artifact

### 2.6 Package Manager（包管理）

**职责**：协议版本管理、依赖解析

## 3. 不属于 Runtime 的

| 模块 | 定位 | 说明 |
|------|------|------|
| **Gateway** | 网络入口 | 未来独立模块，是 Runtime 的网络化 |
| **Workbench** | Station | 观察层，AOS 的第一个 Reference Implementation |
| **业务协议** | 扩展 | Trading、Finance 等，通过 AOS 协议框架接入 |
| **Agent 平台** | 外部 | Claude、Trae、Cursor 等，通过 Adapter 接入 |

## 4. Hook 的正确定位

Hook 是 Adapter，不是 Runtime，更不是 Protocol。

```
git commit
    │
    ▼
Hook (Adapter)              ← 薄包装，只做路由
    │
    ▼
AOS Runtime Interpreter     ← 规则执行
    │
    ▼
Protocol/git-policy.md      ← 规则定义
    │
    ▼
执行结果
```

**Hook 的职责**：捕获 Git 事件，调用 Runtime Interpreter

**Hook 不包含**：规则、策略、验证逻辑

## 5. 与 Protocol 的边界

```
Protocol (WHAT)                 Runtime (HOW)
─────────────────               ─────────────────
定义规则                        执行规则
描述 Schema                     验证 Schema
定义生命周期                    管理生命周期状态机
定义 Git 策略                   执行 Git 策略检查
纯文本 (.md, .json, .yaml)     有代码 (.ps1, .py)
永远不变更执行逻辑               可以升级执行逻辑
```

## 6. 与 Project 的边界

```
Runtime                         Project
─────────────────               ─────────────────
安装在 AOS_HOME                 安装在任意位置
被所有项目共享                  每个项目独立
不保存项目资产                  保存源代码和业务资产
不关心业务内容                  不拥有 Runtime
```