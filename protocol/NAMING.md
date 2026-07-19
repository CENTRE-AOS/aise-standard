# AOS Naming System

> 版本: 1.0
> 状态: Frozen
> 生效日期: 2026-07-17
> 所属: AOS Architecture Constitution — Appendix A

---

## 四个永久身份

| 身份 | 名称 | 本质 | 真相源 |
|------|------|------|--------|
| **协议工厂** | AOS Foundation | 研发 AOS 协议，生产 Protocol Artifact | Git |
| **协议运行环境** | AOS Runtime | 安装、分发、执行协议，提供 CLI/Hook/Adapter | AOS_HOME |
| **真相快照引擎** | Git | 记录和冻结项目生命轨迹，不是代码托管 | Git |
| **生命实例** | Project | 遵循协议运行的生命体，由 Runtime 管理状态，由 Git 保存历史 | Git + AOS_HOME |

---

## 四个永久概念

1. **Protocol Factory（协议工厂）** → 研发协议，Git 是其真相源
2. **AOS Runtime（协议运行系统）** → 安装、分发、执行协议
3. **Git（Truth Snapshot Engine）** → 项目生命轨迹记录器，不是代码托管
4. **Project（Living Instance）** → 遵循协议的生命体

---

## 统一术语表

| 术语 | 定义 | 禁止的误解 |
|------|------|-----------|
| **AOS** | Agent Operating System，Agent 时代的基础设施协议层 | 不是软件、框架、工具集合 |
| **AISE** | Agent Software Engineering Protocol，AOS 默认一号协议 | 不是 AOS 本身，是 AOS 的一个协议 |
| **CENTRE** | 秩序原则，定义什么是合法 | 不是软件模块，不是 `centre.exe` |
| **Protocol** | 宪法，描述 WHAT | 没有执行能力，零代码 |
| **Runtime** | 执行环境，负责 HOW | 不定义规则，不关心业务 |
| **AOS_HOME** | 环境基础设施目录 | 不是 Git 仓库，是本地文件系统目录 |
| **Protocol Artifact** | 不可变协议产品 | 不是 Git 仓库，不是源码 |
| **Adapter** | 桥接层，连接 Runtime 和外部系统 | 不是 Runtime 本身 |
| **Hook** | Git 适配器，执行协议规则的薄包装 | 不包含规则，规则在 Protocol 中 |
| **Interpreter** | 统一解释器，所有 CLI 命令的入口 | 不各自解析规则 |
| **Station** | 观察层，人类交互界面 | 不是 OS 本体 |
| **Gateway** | 网络入口，Runtime 的网络化 | 未来独立模块，不是新 Runtime |

---

## 仓库命名规范

| 仓库 | 用途 | GitHub 名称 |
|------|------|-------------|
| 协议工厂 | 协议规范、Schema、模板、发布定义 | `aos-protocol-factory` |
| 运行时 | CLI、Core、Adapter、Installer | `aos-runtime` |

---

## 命名原则

1. **不混淆生产和产品**：GitHub 仓库是生产车间，AOS_HOME 是产品安装位置
2. **不混淆协议和执行**：Protocol 永远零代码，Runtime 永远不定义规则
3. **不混淆系统和业务**：AOS 不知道 Trading/Finance/Workbench
4. **不混淆中心和项目**：CENTRE 定义秩序，Project 保存生命