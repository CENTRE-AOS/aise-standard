# CENTRE-HANDOFF.md

> Root Architecture Agent → Project Agents
> Phase: S2 Complete
> Date: 2026-07-21
> Status: ACTIVE

---

## 1. Current Production State

### Foundation Complete

| Layer | Status |
|-------|:------:|
| Identity Freeze (S0→S0.11) | FROZEN |
| Repository Authority (S1) | ESTABLISHED |
| Remote Binding (S2) | SYNCHRONIZED |
| Production Baseline | RECORDED |

### Repository Snapshot

| Authority | Repo | Commit | Visibility |
|-----------|------|--------|:----------:|
| A0 Protocol | aise-standard | `a5dbaca` | public |
| A1 Source | aos-runtime | `d246657` | public |
| A2 Build | aos-factory | `7291438` | private |
| A4 Install | aos-installer | `eab16c3` | private |
| A3 Artifact | artifact-registry | RESERVED | Phase 3 |

### Reference

Full baseline at: `protocol/rfc/production-baseline-v1.json`

---

## 2. Agent Architecture

### Root Agent (Current — This Handoff)

**Role**: CENTRE Architecture Governor

**Responsibilities**:
- Protocol Integrity
- Architecture Review
- Cross-Repository Dependency Management
- Release Boundary Governance
- Agent Ownership Matrix

**After Handoff**: Degrades to read-only Architecture Governor. Does NOT enter individual project repos.

### Project Agents (After Handoff)

**Each project agent**:
- Owns exactly ONE repository
- Bootstraps from `AGENT_BOOTSTRAP.md` in its repo
- Reads `remote-identity.json` + `dependency-lock.json` + `bootstrap-contract.md`
- Respects the Agent Ownership Matrix (`protocol/governance/agent-ownership.md`)
- Does NOT cross repository boundaries

---

## 3. Agent Ownership Matrix

Reference: `protocol/governance/agent-ownership.md`

| Repo | Agent | Allowed Scope |
|------|-------|---------------|
| aise-standard | protocol-agent | RFC, SPEC, Governance documents |
| aos-runtime | runtime-agent | Runtime kernel, Skills, kernel maps |
| aos-factory | factory-agent | Build pipeline, signing, artifact production |
| aos-installer | installer-agent | CLI, injection, deployment, bootstrap engine |

### Forbidden Cross-Operations

- installer-agent SHALL NOT modify Protocol, Runtime, or Factory
- factory-agent SHALL NOT modify Runtime source or Protocol contracts
- runtime-agent SHALL NOT modify Installer or Factory
- protocol-agent SHALL NOT modify Runtime/Factory/Installer implementation

---

## 4. Next Phase: S3 Release Plane Bootstrap

### S3 Boundary (Frozen)

Reference: `protocol/rfc/phase-3-release-plane.md`

- S3 establishes the Release Plane: tag → release → artifact chain
- A3 Artifact Authority remains RESERVED
- Installer must NOT evolve into Artifact Registry
- CI gates to be added in S4

### Open Issues for Project Agents

| Issue | Repo | Owner | Severity | Phase |
|-------|------|-------|:--------:|:-----:|
| A4-ISSUE-001 | aos-installer | installer-agent | medium | S4 |

Reference: `aos-installer/issues/A4-ISSUE-001.md`

---

## 5. Bootstrap Reference

When any new agent enters a project:

1. Read `AGENT_BOOTSTRAP.md` in project root
2. Read `.github/remote-identity.json`
3. Read `.github/dependency-lock.json`
4. Read `aise-standard/protocol/rfc/bootstrap-contract.md`
5. Read `aise-standard/protocol/rfc/authority-matrix.md`
6. Read `aise-standard/protocol/rfc/production-baseline-v1.json`
7. Verify `remote_status == active`

---

## 6. Governance Documents Index

| Document | Location | Purpose |
|----------|----------|---------|
| Production Baseline | `protocol/rfc/production-baseline-v1.json` | Current state snapshot |
| Release Boundary | `protocol/rfc/RFC-0003-production-chain.md` | Full production chain architecture |
| Authority Matrix | `protocol/rfc/authority-matrix.md` | Five-level authority definition |
| Bootstrap Contract | `protocol/rfc/bootstrap-contract.md` | Agent entry requirements |
| Naming Policy | `protocol/rfc/naming-ownership-policy.md` | Namespace and naming rules |
| Upgrade Plan | `protocol/rfc/phase-2.8-upgrade-plan.md` | Phase 2.8 execution plan |
| Agent Ownership | `protocol/governance/agent-ownership.md` | Agent scope matrix |
| Release Plane | `protocol/rfc/phase-3-release-plane.md` | S3 boundary definition |

---

## 7. CONSTRAINTS (DO NOT VIOLATE)

1. **No code in Protocol layer** — Protocol owns governance, not implementation
2. **Cross-repo modifications forbidden** — each agent owns one repo
3. **A3 Artifact Authority is RESERVED** — do not implement before Phase 3
4. **Installer ≠ Artifact Registry** — Installer consumes artifacts, does not produce them
5. **Workspace is NOT an Authority** — no governance files in workspace root
6. **GitHub is Source Authority** — local disk is development convenience
7. **Tag ≠ Release** — release requires build + sign + verify + publish
8. **Runtime Consumer = Factory ONLY** — Installer must NOT consume Runtime Source
