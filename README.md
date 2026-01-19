# SDS - Secure Document System

A terminal-native writing environment designed for security-conscious users who need strong guarantees around document encryption, controlled egress, and auditability.

---

## What is SDS?

SDS is a TUI (Terminal User Interface) application that provides:

- **Encrypted Storage** — Documents are encrypted at rest using SQLCipher with Argon2id key derivation
- **Controlled Egress** — All exports (PDF) and clipboard operations require explicit confirmation and are logged
- **Audit Trail** — Every document export and copy operation is recorded in an append-only, HMAC-chained audit log
- **Clean Exports** — PDFs are sanitized to remove vault metadata, timestamps, and other identifying information
- **Terminal Native** — Works in your terminal, integrates with tmux/zellij, no GUI required

SDS is built for users who need to draft sensitive documents while maintaining visibility into how and when content leaves the system.

---

## Key Features

### Vault-Based Storage
```
sds init          # Create a new encrypted vault
sds open          # Unlock and enter the TUI
```

Your documents live in a single encrypted SQLCipher database. One passphrase unlocks everything. Documents are encrypted at rest and decrypted only in memory during editing.

### Lax Mode Drafting

Write freely without structural constraints. SDS validates document structure only at export time, letting you focus on content during drafting. The UI shows a health indicator so you know if your document will export cleanly.

### Egress Control

Every path out of SDS is mediated:

| Action | Gate | Audit |
|--------|------|-------|
| PDF Export | Confirmation + structure validation | Logged with content hash |
| Clipboard Copy | Confirmation (>100 chars) | Logged with byte count |

### Security State Visibility

The UI always displays:
- Vault lock status
- Memory protection tier (A/B/C)
- Clipboard mode (enabled/disabled)
- Document structure health

### PDF Sanitization

Exported PDFs are cleaned:
- Metadata stripped (author, creator, timestamps)
- No vault information (tags, folders, IDs)
- Deterministic normalization for reproducibility

---

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                    TUI (tcell)                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Doc List    │  │ Editor      │  │ Search      │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
│              ┌─────────────────────────┐                │
│              │   Security State Bar    │                │
│              └─────────────────────────┘                │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────┴─────────────────────────────────┐
│                    Vault Core                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ Argon2id    │  │ Key Mgmt    │  │ Secure Buf  │     │
│  │ KDF         │  │ K_db/audit  │  │ mlock/zero  │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└───────────────────────┬─────────────────────────────────┘
                        │
┌───────────────────────┴─────────────────────────────────┐
│                 Storage Layer                            │
│  ┌─────────────────────────────────────────────────┐   │
│  │              SQLCipher Database                  │   │
│  │  ┌──────────┐ ┌──────────┐ ┌──────────┐        │   │
│  │  │documents │ │tags      │ │audit     │        │   │
│  │  └──────────┘ └──────────┘ └──────────┘        │   │
│  │  ┌────────────────────────────────────┐        │   │
│  │  │      FTS5 (full-text search)       │        │   │
│  │  └────────────────────────────────────┘        │   │
│  └─────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────┘
                        │
┌───────────────────────┴─────────────────────────────────┐
│                 Egress Pipeline                          │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐     │
│  │ PDF Export  │  │ OSC52 Copy  │  │ Audit Log   │     │
│  │ + Sanitize  │  │ + Confirm   │  │ + HMAC      │     │
│  └─────────────┘  └─────────────┘  └─────────────┘     │
└─────────────────────────────────────────────────────────┘
```

---

## CLI Commands

| Command | Description |
|---------|-------------|
| `sds init` | Create a new vault |
| `sds open` | Unlock vault and launch TUI |
| `sds export <doc>` | Export document to PDF (headless) |
| `sds audit` | View audit log |
| `sds doctor` | Check environment (core dumps, swap, sessions) |
| `sds pdf-inspect <file>` | Show PDF metadata |

---

## Threat Model

### What SDS Protects Against

| Threat | Mitigation |
|--------|------------|
| Offline vault theft | SQLCipher encryption + Argon2id KDF |
| Accidental clipboard leakage | Confirmation gates + audit logging |
| PDF metadata leakage | Sanitization + deterministic export |
| Untracked exports | Append-only HMAC-chained audit log |
| Memory persistence | Best-effort zeroing + mlock + no core dumps |
| Terminal scrollback | Controlled rendering, no debug logging |

### What SDS Does NOT Protect Against

- Compromised OS/kernel/hypervisor
- Hardware keyloggers / DMA attacks
- Screen capture / shoulder surfing
- Specialized side-channel attacks
- Forensic recovery after manual copy-paste

**SDS reduces persistence and increases auditability. It is not perfect secrecy.**

---

## Memory Protection Tiers

SDS detects and displays its memory protection capability:

| Tier | Description | Indicators |
|------|-------------|------------|
| **A** | Go runtime secret scope support | Experimental runtime features available |
| **B** | OS-level protection | mlock + madvise(DONTDUMP) working |
| **C** | Basic protection | Explicit zeroing only, warnings displayed |

The current tier is always visible in the security state bar.

---

## Installation

### Prerequisites

- Go 1.21+ (for build)
- SQLCipher library
- Linux (primary) or macOS (secondary support)

### Build from Source

```bash
git clone https://github.com/yourusername/SDS.git
cd SDS
go build -o sds ./cmd/sds
```

### Verify Installation

```bash
./sds doctor
```

This checks:
- Core dump status
- Swap configuration
- Memory locking capability
- Nested terminal session detection

---

## Quick Start

```bash
# 1. Create a vault
sds init --vault ~/secure.vault
# Enter a strong passphrase

# 2. Open the vault
sds open --vault ~/secure.vault

# 3. Create and edit documents in the TUI

# 4. Export when ready
# Use the export command in the TUI or:
sds export "My Document" --output document.pdf

# 5. Review audit log
sds audit
```

---

## Configuration

SDS uses sensible defaults. Configuration options:

| Option | Default | Description |
|--------|---------|-------------|
| `--vault` | `~/.sds/vault.db` | Vault file path |
| `--argon-memory` | 64MB | Argon2id memory parameter |
| `--argon-time` | 3 | Argon2id time parameter |
| `--argon-threads` | 4 | Argon2id parallelism |
| `-m` | disabled | Enable mouse support |
| `--copy-threshold` | 100 | Chars before copy confirmation |

Low-resource environments can use `--profile low-ram` for reduced Argon2id parameters (with security trade-off warning).

---

## Development

### Project Structure

```
sds/
├── cmd/sds/           # CLI entrypoint
├── internal/
│   ├── ui/            # tcell views + input routing
│   ├── vault/         # unlock, key lifecycle, capabilities
│   ├── store/         # SQLCipher, migrations, FTS
│   ├── export/        # parse → Maroto → sanitize
│   ├── egress/        # OSC52, confirmation, audit emission
│   ├── audit/         # HMAC chain, verification
│   └── doctor/        # environment checks
├── documentation/
│   ├── MVP.md         # MVP specification
│   └── ROADMAP.md     # Development roadmap
└── README.md
```

### Running Tests

```bash
go test ./...

# With benchmarks
go test -bench=. ./internal/store/...

# Security tests (debug build)
go test -tags=debug ./internal/vault/...
```

### Contributing

1. Check the [ROADMAP.md](documentation/ROADMAP.md) for current priorities
2. Reference task IDs in commits (e.g., `P1-003: Implement SQLCipher integration`)
3. Run `sds doctor` on your dev environment
4. Follow secure coding guidelines:
   - No `string` conversion of sensitive `[]byte`
   - Zero buffers after use
   - No debug logging of document content

---

## Documentation

- [MVP Specification](documentation/MVP.md) — Full technical specification
- [Development Roadmap](documentation/ROADMAP.md) — Phase-by-phase implementation plan

---

## Security Considerations

### For Users

- Use a strong, unique passphrase
- Run `sds doctor` to verify your environment
- Review the audit log periodically
- Understand that Tier C provides limited protection
- Be aware of your terminal multiplexer's scrollback settings

### For Developers

- Never log document content
- Never convert sensitive bytes to strings
- Zero buffers explicitly after use
- Test memory residue in debug builds
- Review the threat model in MVP.md

---

## FAQ

**Q: Why terminal-based?**
A: Terminals provide a controlled rendering environment with less attack surface than GUI frameworks. Integration with existing terminal workflows (tmux, ssh) is seamless.

**Q: Is this a replacement for full-disk encryption?**
A: No. SDS provides application-level encryption for documents. Use FDE as a complementary layer.

**Q: Why not just use GPG?**
A: GPG encrypts files but doesn't provide egress control, audit logging, or integrated editing. SDS is an integrated environment, not just encryption.

**Q: What happens if I forget my passphrase?**
A: Your documents are unrecoverable. There is no backdoor. Back up your passphrase securely.

**Q: Can I use SDS over SSH?**
A: Yes. OSC52 clipboard support depends on your terminal emulator and SSH configuration.

---

## License

GNU General Public License v3.0 — see [LICENSE](LICENSE)

---

## Status

**Current:** Pre-MVP Development

See the [roadmap](documentation/ROADMAP.md) for implementation progress.

---

*SDS: Write securely. Export intentionally. Audit everything.*
