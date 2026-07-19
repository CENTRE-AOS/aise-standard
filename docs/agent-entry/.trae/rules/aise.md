> WARNING: This repository implements AISE Protocol.
> Before any analysis or modification:
>
> 1. Locate `.agent-entry.json` in project root
> 2. Read `.agent-entry.json` to confirm protocol version and governance provider
> 3. Read `PROJECT_BLUEPRINT.md` and `CHANGELOG.md`
> 4. Load `.project/memory/` and `.project/context/` if present
> 5. Execute `aise bootstrap` via the declared governance provider if available
>
> **Do not use default agent behavior before completing bootstrap.**

---

# Trae Agent Rules — AISE Protocol 1.0 / v2.5.0frozen

进入本项目后，首先读取 `.agent-entry.json`，确认项目遵循 AISE Protocol。

然后读取 `PROJECT_BLUEPRINT.md` 了解项目架构，读取 `.project/context/mission.json` 了解当前任务。

所有行为受 AISE Contracts 与 Policies 约束。Autonomous Agent Engineering 模式。
