# Dependency DAG Validation

> Version: 1.0.0-frozen
> Date: 2026-07-21
> Phase: S2.9 — Dependency DAG Integrity Check
> Status: FROZEN
> Purpose: Validate that all dependency directions are correct and no reverse dependencies exist.

---

## 1. Dependency Directed Acyclic Graph (DAG)

```
                    A0 Protocol
                  aise-standard
                        │
                        │ provides: protocol-contracts
                        │ consumed by: A2
                        │
                        ▼
                    A1 Runtime Source
                    aos-runtime
                        │
                        │ provides: runtime-source
                        │ consumed by: A2 ONLY
                        │
                        ▼
                    A2 Build Factory
                    aos-factory
                        │
                        │ consumes: protocol-contracts + runtime-source
                        │ provides: signed-artifact
                        │ consumed by: A3 (RESERVED)
                        │
                        ▼
                    A3 Artifact Registry
                    RESERVED (Phase 3)
                        │
                        │ stores: signed-artifact
                        │ provides: published-artifact
                        │ consumed by: A4
                        │
                        ▼
                    A4 Installer
                    aos-installer
                        │
                        │ consumes: published-artifact (from A3)
                        │ provides: installed-runtime
                        │ consumed by: tenant
                        │
                        ▼
                  Runtime Instance
```

---

## 2. Valid Dependency Paths

| From | To | Type | Status |
|------|----|------|:------:|
| A0 Protocol | A2 Factory | protocol-contracts | VALID |
| A1 Runtime | A2 Factory | runtime-source | VALID |
| A2 Factory | A3 Registry | signed-artifact | RESERVED |
| A3 Registry | A4 Installer | published-artifact | RESERVED |

---

## 3. Forbidden Dependency Paths

| From | To | Reason | Status |
|------|----|--------|:------:|
| A4 Installer | A1 Runtime | Installer MUST NOT consume Runtime Source | BLOCKED |
| A4 Installer | A2 Factory | Installer MUST NOT consume Factory Artifact | BLOCKED |
| A2 Factory | A4 Installer | Factory MUST NOT push to Installer | BLOCKED |
| A1 Runtime | A4 Installer | Runtime MUST NOT know Installer | BLOCKED |
| Any Agent | Cross-repo direct | MUST use artifact chain | BLOCKED |

---

## 4. Current dependency-lock.json Validation

### aise-standard/dependency-lock.json

```json
{
  "provides": "protocol-contracts",
  "downstream": ["aos-factory"],
  "forbidden_consumers": []
}
```

**Status**: VALID — Protocol only consumed by Factory.

### aos-runtime/dependency-lock.json

```json
{
  "consumes": ["protocol-contracts"],
  "provides": "runtime-source",
  "downstream": ["aos-factory"],
  "forbidden_consumers": ["aos-installer", "tenant"]
}
```

**Status**: VALID — Runtime consumed by Factory only, Installer blocked.

### aos-factory/dependency-lock.json

```json
{
  "consumes": ["protocol-contracts", "runtime-source"],
  "provides": "signed-artifact",
  "downstream": ["artifact-registry"],
  "forbidden_consumers": ["aos-installer", "tenant"]
}
```

**Status**: VALID — Factory consumes Protocol + Runtime, outputs to Registry. Installer blocked.

### aos-installer/dependency-lock.json

```json
{
  "consumes": "artifact-registry",
  "provides": "installed-artifact",
  "downstream": ["tenant"],
  "forbidden_input": ["aos-runtime-source", "aos-factory-artifact"]
}
```

**Status**: VALID — Installer consumes from Registry only, Runtime Source blocked.

---

## 5. DAG Integrity Score

| Check | Result |
|-------|:------:|
| No circular dependencies | PASS |
| No reverse dependencies | PASS |
| No forbidden consumers violated | PASS |
| No forbidden inputs violated | PASS |
| All downstream paths correct | PASS |
| Supply chain order matches DAG | PASS |

**Overall**: DAG INTEGRITY VERIFIED — 6/6 PASS

---

## 6. CI Gate Template (for S4)

The future `contract-gate.yml` in each repo SHALL validate:

```yaml
dependency-check:
  - verify dependency-lock.json exists
  - verify consumed dependencies declared
  - verify forbidden_consumers match authority-boundary-audit.md
  - verify forbidden_input match authority-boundary-audit.md
  - reject push if any forbidden path is detected
```

---

## 7. Evolution Rules

1. New dependencies MUST be declared in dependency-lock.json BEFORE implementation
2. Any new dependency crossing the Release Plane MUST go through Architecture Governor review
3. Reverse dependencies (downstream→upstream) are PERMANENTLY FORBIDDEN
4. dependency-lock.json changes require Architecture Governor approval until S4 CI gates are active
