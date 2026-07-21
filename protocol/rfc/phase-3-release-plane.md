# Phase 3 — Release Plane Boundary

> Version: 1.0.0-frozen
> Date: 2026-07-21
> Phase: S3.0 — Release Plane Boundary Freeze
> Status: FROZEN (boundary definition, not implementation)

---

## 1. Purpose

This document freezes the Release Plane boundary for S3+ development. It defines what S3 enables and, critically, **what it does NOT enable**. This is a boundary document, not an implementation plan.

---

## 2. Current State (S2 Complete)

```
Source Plane
        │
        ├── A0 Protocol (aise-standard)     → contract release
        ├── A1 Runtime (aos-runtime)         → source release
        ├── A2 Factory (aos-factory)         → build artifact
        └── A4 Installer (aos-installer)     → deployment

Release Plane
        │
        └── NOT YET ESTABLISHED
```

Current operations are Source Plane only: git commit and push. No tags, no releases, no CI.

---

## 3. S3 Target State

```
Source Plane                              Release Plane
        │                                        │
        ├── git tag ──────────────────────────→ GitHub Release
        │                                        │
        ├── build ────────────────────────────→ signed artifact
        │                                        │
        ├── verify ───────────────────────────→ artifact manifest
        │                                        │
        └── publish ──────────────────────────→ artifact registry (A3)
```

### S3 enables:
- Git tags on main branch
- GitHub Releases with changelogs
- CI gates (identity, contract, build validation)
- Release notes generation

### S3 does NOT enable:
- Artifact Registry implementation (A3 = RESERVED)
- Cross-repository artifact consumption
- Automated deployment from releases
- Installer consuming releases directly

---

## 4. Production Chain (S3 View)

```
A0 Protocol
aise-standard
        │
        │ git tag v2.0.0
        │ GitHub Release
        │
        ▼
A1 Runtime Source
aos-runtime
        │
        │ git tag v3.2.0
        │ GitHub Release
        │
        ▼
A2 Factory Build
aos-factory
        │
        │ consume: protocol-contract + runtime-source
        │ produce: signed-artifact
        │
        ▼
A3 Artifact Authority
RESERVED (Phase 3)
        │
        │ (future: artifact-registry)
        │
        ▼
A4 Installer
aos-installer
        │
        │ consume: published-artifact (via A3)
        │ produce: installed-runtime
        │
        ▼
Runtime Instance
```

---

## 5. S3 Boundary Constraints

### 5.1 Factory S3 Scope

```
S3 Factory:
    ✅ Create git tags (v1.1.0, v1.2.0, ...)
    ✅ Create GitHub Releases
    ✅ Add release.yml CI workflow
    ✅ Add identity-gate.yml + contract-gate.yml
    ❌ Implement Artifact Registry
    ❌ Sign artifacts (Phase 3+)
    ❌ Publish to external registry
```

### 5.2 Installer S3 Scope

```
S3 Installer:
    ✅ Create git tags (v1.1.0, v1.2.0, ...)
    ✅ Create GitHub Releases
    ✅ Add release.yml CI workflow
    ✅ Add identity-gate.yml + contract-gate.yml
    ❌ Consume factory artifacts directly
    ❌ Implement artifact registry
    ❌ Build runtime from source
```

### 5.3 A3 Artifact Authority

```
A3 = RESERVED
    Status: Not implemented
    Phase: 3.0+
    Scope: Artifact storage, versioning, signing, distribution
    Owner: TBD (new agent or factory-agent extension)
    Do NOT: let Installer evolve into Artifact Registry
```

---

## 6. Forbidden Evolution Paths

| Path | Why Forbidden |
|------|---------------|
| Installer → Artifact Registry | Authority violation — Installer is A4, Registry is A3 |
| Factory → Artifact Registry | Authority overlap — Factory builds, Registry stores |
| Runtime → Artifact Registry | Authority violation — Runtime is source, not storage |
| Direct Source consumption after S3 | Violates Release Plane separation |

---

## 7. S3 Exit Criteria

S3 is complete when:
1. All four repos have git tags on main
2. All four repos have GitHub Releases
3. All four repos have CI gates (identity + contract)
4. Release notes are generated for each release
5. CHANGELOG.md is updated with each release

S3 does NOT require:
- Artifact Registry implementation
- Cross-repository artifact consumption
- Signed artifacts
- Automated deployment

---

## 8. Next After S3

```
S3 Release Plane Bootstrap   ← current boundary
        │
        ▼
S4 CI Production Gates       ← next phase
        │
        ▼
S5 Supply Chain Verification ← final phase of Phase 2.8
        │
        ▼
Phase 3.0: Artifact Registry  ← A3 RESERVED → implemented
```
