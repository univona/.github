# Univona Phase 1: Identity System & End-to-End Encryption Core

## Progress Tracker

Last updated: 2026-02-13

### Platform Support

| Component | Platforms | Notes |
|-----------|-----------|-------|
| Server (daemon) | Linux: Ubuntu, CentOS, Debian (latest) | No legacy version support |
| Client | macOS, Windows, Android (latest) | No legacy version support |

### Task Status

| # | Task | Repository | Status | Notes |
|---|------|-----------|--------|-------|
| Prep | univona-shared enhancement (UnivonaId, auth types) | univona | ✅ Complete | Required before downstream tasks |
| 1.1 | Secure key store abstraction (Easy Mode) | univona-client | ✅ Complete | OS keychain via `keyring` |
| 1.2 | Identity Key generation + UUID derivation | univona-client | ✅ Complete | Ed25519 → X25519, `nxl:` prefix |
| 1.3 | BIP-39 mnemonic recovery phrase | univona-client | ✅ Complete | 24-word, deterministic recovery |
| 1.4 | Device Signing Key + certificates | univona-client | ✅ Complete | Merged with 1.9 |
| 1.5 | X3DH key agreement | univona-client | ✅ Complete | Full protocol with OPK |
| 1.6 | Double Ratchet | univona-client | ✅ Complete | DH + symmetric ratchet |
| 1.7 | Sender Keys (group encryption) | univona-client | ✅ Complete | Key rotation on member leave |
| 1.8 | Local encrypted storage | univona-client | ✅ Complete | SQLCipher / AES-GCM fallback |
| 1.9 | Cross-signing (IK→SSK→DSK) | univona-client | ✅ Complete | Merged with 1.4 |
| 1.10 | Flutter identity UI (FFI) | univona-client | ✅ Complete | Minimal: create/recover/validate |
| 1.11 | Server UUID authentication | univona-daemon | ✅ Complete | Challenge-response + JWT |
| 1.12 | PreKey storage service | univona-daemon | ✅ Complete | Upload/fetch/consume/count |
| 1.13 | Server validation enhancement | univona-daemon | ✅ Complete | IntoResponse + request-id |

### Dependency Graph

```
Prep (shared) ──┬──→ 1.1 → 1.2 ──┬──→ 1.3 → 1.10 (Flutter FFI)
                │                  ├──→ 1.4+1.9 → 1.5 → 1.6 → 1.7
                │                  └──→ 1.8 (parallel)
                └──→ 1.13 → 1.11 → 1.12
```

### Verification Checklist

- [ ] `cargo build --workspace` compiles in univona-client
- [ ] `cargo test -p univona-core` passes with >90% coverage
- [ ] Identity generation: UUID format `nxl:` + Base58, 24-word mnemonic
- [ ] Mnemonic recovery: same words → same UUID
- [ ] X3DH + Double Ratchet: two local instances encrypt/decrypt
- [ ] Sender Keys: 3-person group encrypt/decrypt
- [ ] Server `POST /api/v1/auth/challenge` → `POST /api/v1/auth/verify` flow
- [ ] PreKey API: upload → fetch → consume → count
- [ ] All endpoints return unified `ApiResponse` format

### Module Structure (univona-core after Phase 1)

```
crates/univona-core/src/
  lib.rs                    # Module exports + FFI entry points
  error.rs                  # CoreError
  identity/
    mod.rs
    keys.rs                 # IdentityKeyPair, PublicIdentityKey
    uuid.rs                 # UnivonaId
    mnemonic.rs             # BIP-39 mnemonic
    certificate.rs          # DeviceCertificateBuilder
    cross_sign.rs           # SelfSigningKey, CrossSignVerifier
  crypto/
    mod.rs
    x3dh.rs                 # X3DH key agreement
    ratchet.rs              # Double Ratchet
    sender_key.rs           # Sender Keys group encryption
    kdf.rs                  # KDF operations
    aead.rs                 # AES-256-GCM
  keystore/
    mod.rs                  # KeyStore trait
    software.rs             # Easy Mode (OS keychain)
  storage/
    mod.rs                  # LocalStore trait
    encrypted_db.rs         # SQLCipher implementation
    schema.sql              # Table definitions
```
