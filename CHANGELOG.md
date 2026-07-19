# Changelog

All notable changes to aise-standard will be documented in this file.

## [2.0.0-frozen] — 2026-07-19

### Initial Release (Repository Extraction)

- Repository extracted from legacy monorepo (`legacy-monorepo-freeze-v3.1.0`)
- AOS Repository Separation Architecture v1.1.0 FROZEN
- Five-role isolation model established
- Three core boundaries: Authority, Artifact, Lifecycle
- Bootstrap Contract ratified (three-layer: Protocol defines, Runtime executes, Factory validates)
- Installer Bootstrap Layer concept introduced (Bootstrap Discovery Registry + Runtime Adapter Registry)
- 22 ADRs documented
- Protocol Registry established (compliance, routing, skills, version)
- 14 System Governance Skills defined
- Project Bootstrap Templates provided

### Migration Baseline

- Source: `legacy-monorepo-freeze-v3.1.0` (commit `ffded4c`)
- Method: git filter-repo extraction
- Preserved: 30 commits of git history