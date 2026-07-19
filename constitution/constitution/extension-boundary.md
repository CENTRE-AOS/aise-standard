# CENTRE Runtime Extension Boundary

> 版本: 2.0.0
> 状态: Frozen
> 日期: 2026-07-17
> 适用范围: CENTRE Gateway Runtime v2.1.0+
> 前置: Runtime Contract v1.0.0

---

## 1. 目的

定义 CENTRE Gateway Runtime 的扩展边界，防止 Runtime 无限膨胀为 Monolith。

核心原则：**Runtime 是解释器，不是能力仓库。**

## 2. 架构模型：Kernel + Extension Packages

```
                    CENTRE Runtime Kernel
                    (只负责基础运行时)
                          │
            ┌─────────────┼─────────────┐
            │             │             │
     Extension         Protocol        Tenant
     Packages          Packages        Packages
      (能力)            (规则)          (身份)
```

### 2.1 Kernel（不可变核心）

Runtime Kernel 只负责最基础的运行时能力：

```
runtime/core/
├── interpreter.ps1     ← 统一解释器
├── lifecycle.ps1       ← 生命周期管理
├── registry.ps1        ← 注册表
├── eventbus.ps1        ← 事件总线
├── statemachine.ps1    ← 状态机
├── admission.ps1       ← 准入控制
├── environment.ps1     ← 环境管理
├── identity.ps1        ← 身份（Phase 3）
└── context.ps1         ← 上下文（Phase 4）
```

**规则：**
- Kernel 只增加"不可变运行时基础能力"
- 不增加业务逻辑、不增加项目特定逻辑
- 不增加外部能力（Git, FileSystem, Network 等）
- 新增 Kernel 模块需要 Contract 更新 + Runtime 版本升级

### 2.2 Extension Packages（可扩展）

Runtime 通过 Extension System 加载外部能力，不内嵌：

```
Extension Packages
├── Capability Packages     ← 外部系统能力（Git, FileSystem, Network）
├── Skill Packages          ← 系统行为（通过 StateMachine 触发）
├── Protocol Packages       ← 协议定义（来自 Protocol Factory）
├── Adapter Packages        ← Agent 平台适配
└── Tenant Packages         ← 多租户身份（Phase 3+）
```

**规则：**
- Extension Package 独立版本，独立升级
- 不内嵌在 Runtime 源码中
- 通过 Registry 安装和管理

## 3. 扩展边界矩阵

| 扩展类型 | 归属 | 载体 | 生命周期 | 是否进入 Kernel |
|---------|------|------|---------|---------------|
| **Protocol 定义** | Protocol Package | aise-standard / Protocol Artifact | 慢（frozen） | 否 |
| **Runtime 核心** | Kernel | runtime/core/ | 中 | 是 |
| **Skill** | Skill Package | system-skills/（当前） → 独立 Package（未来） | 中 | 否 |
| **Capability** | Capability Package | 独立 Package | 中 | 否 |
| **Adapter** | Adapter Package | 独立 Package | 快 | 否 |
| **Tenant** | Tenant Package | 独立 Package（Phase 3+） | 中 | 否 |
| **CLI** | Runtime | runtime/cli/centre.ps1 | 快 | 否 |

## 4. 禁止的扩展

以下扩展类型禁止进入 Runtime Kernel：

| 禁止类型 | 归属 | 原因 |
|---------|------|------|
| 业务协议 | 独立 Protocol Package | 不属于 Runtime 基础设施 |
| 外部能力实现 | Capability Package | Runtime 是解释器，不是能力仓库 |
| Agent 平台逻辑 | Agent Platform | 平台特定逻辑不应进入 Runtime |
| 项目特定配置 | Project | 项目资产属于 Project |
| 网络网关 | Gateway（独立模块） | 是 Runtime 的网络化，非 Runtime 核心 |
| 用户数据 | Project/Station | Runtime 不存储用户数据 |
| 数据库/存储 | Capability Package | 外部能力 |

## 5. 为什么 Kernel 不包含 Capability

当前 `runtime/core/capability.ps1` 是调用接口，不是实现。

未来演进：

```
现在:
runtime/core/capability.ps1  ← 调用接口（在 Kernel 中）

未来:
capability-packages/
├── git/package.ps1          ← Git 实现
├── filesystem/package.ps1   ← 文件系统实现
└── network/package.ps1      ← 网络实现

Kernel 通过 ICapability 接口调用
```

**原因：**
- Capability 是外部系统的抽象
- 不同环境需要不同 Capability 实现
- Capability 可能独立升级
- 防止 Runtime 5 年后变成"上帝程序"

## 6. 扩展审批流程

新增 Runtime 能力前：

1. 确认归属：Kernel / Extension Package / 禁止
2. 检查是否与现有模块职责重叠
3. 如果属于 Kernel：更新 Contract + Runtime 版本升级
4. 如果属于 Extension：创建独立 Package + Registry 注册
5. 更新此文

## 7. 版本影响

| 层 | 扩展影响 | 说明 |
|----|---------|------|
| Protocol | 新增规则 → Protocol 版本升级 | 影响所有 Runtime |
| Kernel | 新增模块 → Runtime 版本升级 | 影响所有实例 |
| Skill | 新增 Skill → Skill Registry 更新 | 不影响 Runtime 版本 |
| Capability | 新增 Capability → Capability Registry 更新 | 不影响 Runtime 版本 |
| Adapter | 新增 Adapter → Adapter Registry 更新 | 不影响 Runtime 版本 |
| CLI | 新增命令 → CLI 版本升级 | 不影响 Runtime 版本 |
| Tenant | 新增 Tenant → Tenant Registry 更新 | 不影响 Runtime 版本 |