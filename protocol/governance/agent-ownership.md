# Agent Ownership Matrix

> Version: 1.0.0-frozen
> Date: 2026-07-21
> Phase: S2.7 — Agent Boundary Freeze
> Status: FROZEN

---

## 1. Ownership Model

```
CENTRE Architecture Governor (Root Agent)
        │
        │ owns: Protocol Integrity, Cross-Repo Architecture
        │
        ├── A0 Protocol Agent
        │       owns: aise-standard
        │       scope: RFC, SPEC, Governance, Contracts
        │
        ├── A1 Runtime Agent
        │       owns: aos-runtime
        │       scope: Kernel, Skills, Runtime Maps
        │
        ├── A2 Factory Agent
        │       owns: aos-factory
        │       scope: Build Pipeline, Signing, Artifact Production
        │
        └── A4 Installer Agent
                owns: aos-installer
                scope: CLI, Injection, Deployment, Bootstrap Engine
```

---

## 2. Authority Matrix

| Repository | Agent Owner | Authority Level | Allowed Operations |
|-----------|-------------|:--------------:|--------------------|
| aise-standard | protocol-agent | A0 | RFC creation, SPEC definition, Contract authoring, Governance docs |
| aos-runtime | runtime-agent | A1 | Kernel modification, Skill implementation, Runtime maps update |
| aos-factory | factory-agent | A2 | Build pipeline, Signing logic, Release workflow, CI/CD |
| aos-installer | installer-agent | A4 | CLI commands, Injection system, Deployment logic, Bootstrap engine |

---

## 3. Forbidden Operations

### installer-agent SHALL NOT

| Forbidden | Reason |
|-----------|--------|
| Modify Protocol (aise-standard) | Protocol owned by A0 |
| Modify Runtime Contract | Contract owned by A0 |
| Modify Factory Interface | Factory owned by A2 |
| Directly consume Runtime Source | Installer → Artifact Registry, not Runtime Source |
| Directly consume Factory Artifact | Installer → Artifact Registry, not Factory |
| Implement Artifact Registry | A3 is RESERVED |

### factory-agent SHALL NOT

| Forbidden | Reason |
|-----------|--------|
| Modify Protocol contracts | Protocol owned by A0 |
| Modify Runtime source | Runtime owned by A1 |
| Modify Installer | Installer owned by A4 |
| Modify Runtime kernel | Runtime owned by A1 |

### runtime-agent SHALL NOT

| Forbidden | Reason |
|-----------|--------|
| Modify Protocol contracts | Protocol owned by A0 |
| Modify Factory pipeline | Factory owned by A2 |
| Modify Installer logic | Installer owned by A4 |
| Implement build/signing | Factory owns Build Authority |

### protocol-agent SHALL NOT

| Forbidden | Reason |
|-----------|--------|
| Modify Runtime implementation | Runtime owned by A1 |
| Modify Factory implementation | Factory owned by A2 |
| Modify Installer implementation | Installer owned by A4 |
| Add implementation code to Protocol | Protocol is governance, not code |

---

## 4. Cross-Repository Interaction Rules

### Allowed Interactions

```
protocol-agent ──(contract release)──→ factory-agent
protocol-agent ──(contract release)──→ runtime-agent
runtime-agent  ──(source release)───→ factory-agent
factory-agent  ──(signed artifact)──→ artifact-registry (Phase 3)
artifact-registry ──(published artifact)──→ installer-agent
```

### Forbidden Interactions

```
installer-agent ──X──→ runtime-source        (must use artifact-registry)
installer-agent ──X──→ factory-artifact      (must use artifact-registry)
factory-agent   ──X──→ installer             (must go through artifact-registry)
any agent       ──X──→ cross-repo code mod   (Authority Separation)
```

### Interaction Mechanism

All cross-repository interactions SHALL use:
1. Git tags (versioned releases)
2. Artifact manifests with SHA256 signatures
3. Published artifacts through Artifact Registry (Phase 3+)

Direct source code references between repositories are PROHIBITED.

---

## 5. Bootstrap Protocol

When an agent enters any project, it SHALL:

1. Read `AGENT_BOOTSTRAP.md` in project root
2. Read `.github/remote-identity.json` — verify identity
3. Read `.github/dependency-lock.json` — verify dependencies
4. Read `aise-standard/protocol/rfc/bootstrap-contract.md` — governance rules
5. Read `aise-standard/protocol/rfc/authority-matrix.md` — this document
6. Verify `remote_status == active`
7. Execute Bootstrap Context Validation
8. Execute Authority Check

An agent that fails any step SHALL reject the session.

---

## 6. Conflict Resolution

| Conflict Type | Resolution Authority |
|---------------|---------------------|
| Cross-repo architecture dispute | CENTRE Architecture Governor |
| Protocol interpretation | aise-standard (A0) |
| Runtime behavior | aos-runtime (A1) |
| Build/Sign dispute | aos-factory (A2) |
| Install/Deploy dispute | aos-installer (A4) |

Agents SHALL NOT resolve cross-repository conflicts by themselves. Escalate to CENTRE Architecture Governor.
