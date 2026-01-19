# SDS Development Roadmap

**Version:** 1.0
**Last Updated:** 2026-01-19
**Status:** Active Development

---

## Overview

This roadmap outlines the development path for SDS (Secure Document System) from initial scaffolding through MVP completion and beyond. Each phase has clear deliverables and gate criteria that must be met before advancing.

---

## Development Phases

### Phase 0: Foundation & Tooling

**Goal:** Establish project infrastructure, development environment, and core CLI skeleton.

#### Deliverables

| Task ID | Task | Description | Priority |
|---------|------|-------------|----------|
| P0-001 | Project scaffolding | Initialize Go module, directory structure per spec | P0 |
| P0-002 | CI/CD pipeline | GitHub Actions for build, test, lint | P0 |
| P0-003 | CLI skeleton | Implement `sds` command structure with cobra/urfave | P0 |
| P0-004 | `sds doctor` command | Environment checks (core dumps, swap, tmux detection) | P0 |
| P0-005 | Capability tier detection | Detect and report A/B/C memory protection tiers | P0 |
| P0-006 | Logging framework | Structured logging (no sensitive data) | P0 |
| P0-007 | Configuration system | Config file loading, defaults, profiles | P1 |

#### Gate Criteria
- [ ] `go build` succeeds
- [ ] `sds doctor` reports environment status
- [ ] CI pipeline passes on main branch
- [ ] Capability tier correctly detected on Linux

---

### Phase 1: Vault & Storage Layer

**Goal:** Implement secure vault creation, unlocking, and document persistence with SQLCipher.

#### Deliverables

| Task ID | Task | Description | Priority |
|---------|------|-------------|----------|
| P1-001 | Argon2id integration | KDF with configurable parameters | P0 |
| P1-002 | Salt generation | Cryptographically secure per-vault salt | P0 |
| P1-003 | SQLCipher integration | Database encryption layer | P0 |
| P1-004 | Key derivation pipeline | Master key → K_db, K_audit, K_meta | P0 |
| P1-005 | Schema v1 | Documents, tags, doc_tags, audit tables | P0 |
| P1-006 | Migration framework | Versioned schema migrations | P0 |
| P1-007 | `sds init` command | Vault creation with passphrase | P0 |
| P1-008 | `sds open` command | Vault unlock flow | P0 |
| P1-009 | Document CRUD | Create, read, update, delete operations | P0 |
| P1-010 | Autosave mechanism | Periodic encrypted saves | P0 |
| P1-011 | Crash recovery | Startup integrity checks, FTS rebuild | P0 |
| P1-012 | Secure buffer management | Best-effort zeroing, mlock where available | P0 |
| P1-013 | Passphrase strength meter | Feedback on passphrase quality | P1 |

#### Gate Criteria
- [ ] Vault creates and opens successfully
- [ ] Documents persist across sessions
- [ ] Kill process mid-transaction → vault recovers
- [ ] WAL/DELETE mode tests pass
- [ ] Key material zeroed on lock/exit (verified via test hooks)

---

### Phase 2: TUI & Editor

**Goal:** Build the terminal user interface with document editing, search, and navigation.

#### Deliverables

| Task ID | Task | Description | Priority |
|---------|------|-------------|----------|
| P2-001 | tcell initialization | Terminal setup, event loop | P0 |
| P2-002 | Document list view | List, select, create documents | P0 |
| P2-003 | Text editor core | Basic editing (insert, delete, navigation) | P0 |
| P2-004 | Editor buffer management | Secure buffers, avoid string conversion | P0 |
| P2-005 | Keystroke handling | Immediate copy to SDS buffers | P0 |
| P2-006 | FTS5 integration | External content table, prepared statements | P0 |
| P2-007 | Search UI | Query input, results display | P0 |
| P2-008 | Lax Mode indicator | Visual status in UI | P0 |
| P2-009 | Structure health indicator | OK/WARN/FAIL display | P0 |
| P2-010 | Security state bar | Vault status, tier, clipboard mode | P0 |
| P2-011 | Keyboard shortcuts | Navigation, commands, help | P0 |
| P2-012 | Mouse support (optional) | Disabled by default, `-m` flag | P1 |
| P2-013 | Nested session detection | tmux/zellij warning | P0 |
| P2-014 | FTS benchmark harness | 100/1k/10k document tests | P0 |

#### Gate Criteria
- [ ] User can create, edit, save documents
- [ ] Search returns relevant results
- [ ] Lax Mode visible and functional
- [ ] Security state always displayed
- [ ] FTS benchmarks documented

---

### Phase 3: Export & Egress Control

**Goal:** Implement PDF export pipeline, clipboard control, and comprehensive audit logging.

#### Deliverables

| Task ID | Task | Description | Priority |
|---------|------|-------------|----------|
| P3-001 | Markdown parser | Document structure parsing | P0 |
| P3-002 | Maroto PDF generation | Render documents to PDF | P0 |
| P3-003 | PDF sanitization | Strip metadata, normalize timestamps | P0 |
| P3-004 | `sds export` command | Headless PDF export | P0 |
| P3-005 | `sds pdf-inspect` tool | Display PDF metadata | P0 |
| P3-006 | Export-time validation | Structure reconciliation, error display | P0 |
| P3-007 | Export confirmation UI | Preview, diff, byte counts | P0 |
| P3-008 | OSC52 clipboard support | Terminal clipboard integration | P0 |
| P3-009 | Copy confirmation gate | Threshold-based confirmation (>100 chars) | P0 |
| P3-010 | Copy disabled mode | High-security option | P0 |
| P3-011 | Audit log schema | Append-only with HMAC chain | P0 |
| P3-012 | Audit event emission | Export/copy events logged | P0 |
| P3-013 | `sds audit` command | View audit log | P0 |
| P3-014 | Audit chain verification | End-to-end HMAC verification | P0 |
| P3-015 | Audit JSON export | Export report (hashes only) | P0 |

#### Gate Criteria
- [ ] PDF exports contain no vault metadata
- [ ] PDF metadata sanitized
- [ ] Every export/copy logged
- [ ] Audit chain verifies successfully
- [ ] OSC52 works (with nested session warnings)

---

### Phase 4: Hardening & Polish

**Goal:** Security hardening, performance optimization, and user experience refinement.

#### Deliverables

| Task ID | Task | Description | Priority |
|---------|------|-------------|----------|
| P4-001 | Core dump disable | rlimit at startup | P0 |
| P4-002 | Process non-dumpable | prctl/procfs flags | P0 |
| P4-003 | Swap warnings | Detect and warn if mlock unavailable | P0 |
| P4-004 | Memory residue tests | Sentinel scanning in debug builds | P1 |
| P4-005 | Export determinism tests | Same input → same output | P1 |
| P4-006 | Error handling review | User-friendly error messages | P0 |
| P4-007 | Help system | In-app help, man page | P0 |
| P4-008 | Performance profiling | Identify and fix bottlenecks | P1 |
| P4-009 | Cross-platform testing | Linux primary, macOS secondary | P0 |
| P4-010 | Documentation | User guide, threat model docs | P0 |

#### Gate Criteria
- [ ] Core dumps disabled (verified)
- [ ] All P0 tests pass
- [ ] Documentation complete
- [ ] Tested on target platforms

---

## Post-MVP Roadmap

### P1 Features (Strongly Recommended)

| Feature | Description |
|---------|-------------|
| HMAC-blinded tags | Store HMAC(tag) instead of plaintext |
| Deterministic export verification | Byte-level diff against prior export |
| Memory capability tiers | Runtime/secret experimental support |
| Clipboard sandbox | Internal encrypted clipboard |
| Separate audit store | Encrypted audit log file |
| Padding strategy | Document size obfuscation |

### P2 Features (Future/Research)

| Feature | Description |
|---------|-------------|
| Tree-sitter parsing | Incremental structure repair |
| In-memory FTS | Custom VFS for large corpora |
| FIDO2/TPM unlock | Hardware-backed authentication |
| tmpfs draft mode | Timed purge volatile drafts |
| PDF/A compliance | Archival format research |

---

## Issue Tracking Reference

Map these task IDs to GitHub issues for tracking:

### Memory & Security
- `PR-MEM-001` → Capability tiers (A/B/C)
- `PR-MEM-002` → Core dump disable
- `PR-MEM-003` → mlock + madvise
- `PR-MEM-004` → Memory sentinel scanning

### TUI & Terminal
- `PR-TUI-001` → Keystroke buffer handling
- `PR-TUI-002` → No string conversion of secrets
- `PR-TUI-003` → Mouse disabled by default
- `PR-TUI-004` → Nested session detection

### Storage & Database
- `PR-DB-001` → Transaction durability
- `PR-DB-002` → Migration framework
- `PR-DB-003` → FTS5 benchmarks

### Search & Metadata
- `PR-FTS-001` → External-content FTS5
- `PR-FTS-002` → HMAC-blinded tags

### User Experience
- `PR-UX-001` → Lax Mode toggle
- `PR-UX-002` → Export validation
- `PR-UX-003` → Tree-sitter parsing

### PDF Export
- `PR-PDF-001` → Metadata sanitization
- `PR-PDF-002` → pdf-inspect tool
- `PR-PDF-003` → Deterministic export

### Egress & Audit
- `PR-EGR-001` → OSC52 confirmation
- `PR-EGR-002` → HMAC audit chain
- `PR-EGR-003` → Separate audit key
- `PR-EGR-004` → Clipboard sandbox

---

## Open Questions

Track resolution in issues:

1. **Go version policy** — Stable only, or experimental runtime features?
2. **SQLCipher durability mode** — DELETE/FULL vs WAL with checkpointing?
3. **FTS indexing scope** — Full plaintext or redacted subset?
4. **PDF determinism target** — Metadata-only or byte-identical?
5. **Access pattern resistance** — Document padding / noise queries?

---

## Success Metrics

### MVP Complete When:
- User can create vault, unlock, edit documents
- Documents encrypted at rest (SQLCipher + Argon2id)
- Search works (FTS5 with benchmarks)
- PDF export produces clean documents
- Clipboard copy audited via OSC52
- Lax Mode functional with export validation
- Security state visible at all times

### Quality Gates:
- All P0 tasks completed
- Gate criteria met for all phases
- Security tests pass
- Documentation complete
- No critical bugs open

---

## Contributing

See the main README for contribution guidelines. When working on tasks:

1. Reference the Task ID in commits and PRs
2. Update this roadmap when tasks complete
3. Add new issues for discovered work
4. Follow the security review checklist

---

*This roadmap is a living document. Update as development progresses.*
