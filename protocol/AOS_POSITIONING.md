# AOS Positioning

> 版本: 1.0
> 状态: Frozen
> 生效日期: 2026-07-17
> 所属: AOS Architecture Constitution

---

## AISE 在 AOS 中的位置

AISE 不是 AOS 的升级版。AISE 是 AOS 默认安装的一号协议。

```
AOS (Agent Operating System)
├── AISE Protocol (工程协议)       ← 默认一号协议
├── [未来] Trading Protocol       ← 交易协议
├── [未来] Knowledge Protocol     ← 知识协议
├── [未来] Security Protocol      ← 安全协议
└── ...
```

## AOS 全系统架构

```
                        HUMAN
                          │
                     Goal / Value
                          │
                          ▼
                     ┌─────────┐
                     │ CENTRE   │  ← 定义秩序，不占有内容
                     │ (Order)  │     不是软件，是原则
                     └────┬─────┘
                          │
           ┌──────────────┼──────────────┐
           │              │              │
           ▼              ▼              ▼
      Protocol       Runtime       Intelligence
      (定义规则)     (执行规则)      (保存生命)
           │              │              │
           └──────────────┼──────────────┘
                          │
                          ▼
                      Project
                      (生命实例)
                          │
                          ▼
                       Agent
                      (执行动作)
                          │
                          ▼
                      Station
                    (观察 — Workbench)
                          │
                          ▼
                       Loop
                    (持续演进)
```

## 三层架构

```
GitHub Truth Source
        │
        │
        ├──────────────────────────┐
        │                          │
        ▼                          ▼
AOS Foundation              AOS Runtime
(协议工厂)                  (协议执行环境)
        │                          │
        │ 生产 Protocol Artifact    │ 安装、解释、执行
        │                          │
        ▼                          ▼
   Git 真相源                  AOS_HOME 真相源
        │                          │
        └──────────┬───────────────┘
                   │
                   ▼
              Project
              (生命实例)
```

## 职责边界

### AOS Foundation（协议工厂）

- **是**：协议规范的生产者、Schema 的定义者、Templates 的维护者
- **不是**：协议的执行者、项目的管理者、Agent 的控制器
- **产品**：AOS Protocol Artifact（不可变）

### AOS Runtime（协议执行环境）

- **是**：协议的安装者、规则的执行者、合规的验证者
- **不是**：协议的定义者、业务的理解者、项目的拥有者
- **模块**：Core（Interpreter, Registry, Lifecycle, Environment）、CLI、Installer、Adapter

### Project（生命实例）

- **是**：源代码的容器、业务资产的保存者、生命状态的记录者
- **不是**：协议的拥有者、Runtime 的存放处、Hook 的复制地
- **声明**：只声明需要的 AOS 协议版本

## 不在 AOS 范围内

| 模块 | 定位 | 说明 |
|------|------|------|
| **Gateway** | 网络入口 | 未来独立模块，是 Runtime 的网络化，不是新 Runtime |
| **Workbench** | Station | 观察层，AOS 的第一个 Reference Implementation |
| **业务协议** | 扩展 | Trading、Finance、ERP 等，通过 AOS 协议框架接入 |
| **Agent 平台** | 外部 | Claude、Trae、Cursor 等，通过 Adapter 接入 |

## 设计哲学

1. **CENTRE 定义秩序，但不占有内容** — 这是最高的架构原则
2. **Protocol 定义秩序，Runtime 执行秩序，Adapter 接入秩序，Project 保存生命，Agent 执行动作，Human 提供目标**
3. **Protocol 永远零代码** — 只有 .md、.json、.yaml
4. **Git 是 Truth Snapshot Engine** — 不是代码托管，是项目生命轨迹记录器
5. **AOS 不绑死 AISE** — AISE 只是第一个协议，未来可以扩展