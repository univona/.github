# Univona Phase 1：身份系统与端到端加密核心

## 开发进度追踪

最后更新：2026-02-13

---

### 平台支持

| 组件 | 支持平台 | 备注 |
|------|----------|------|
| 服务端（daemon） | Linux：Ubuntu、CentOS、Debian（最新版） | 不保证旧版本兼容 |
| 客户端 | macOS、Windows、Android（最新版） | 不保证旧版本兼容 |

---

### 任务完成状态

| 编号 | 任务 | 所属仓库 | 状态 | 备注 |
|------|------|----------|------|------|
| 准备 | univona-shared 增强（UnivonaId、认证类型） | univona | ✅ 已完成 | 下游仓库依赖此变更 |
| 1.1 | 安全密钥存储抽象（Easy Mode） | univona-client | ✅ 已完成 | 通过 `keyring` 访问 OS 密钥链 |
| 1.2 | 身份密钥生成 + UUID 推导 | univona-client | ✅ 已完成 | Ed25519 → X25519 转换，`nxl:` 前缀 |
| 1.3 | BIP-39 助记词恢复短语 | univona-client | ✅ 已完成 | 24 词，确定性恢复 |
| 1.4 | 设备签名密钥 + 证书 | univona-client | ✅ 已完成 | 与 1.9 合并实现 |
| 1.5 | X3DH 密钥协商 | univona-client | ✅ 已完成 | 完整协议（含 OPK） |
| 1.6 | 双棘轮协议 | univona-client | ✅ 已完成 | DH 棘轮 + 对称棘轮 |
| 1.7 | Sender Keys 群组加密 | univona-client | ✅ 已完成 | 成员离开时密钥轮换 |
| 1.8 | 本地加密存储 | univona-client | ✅ 已完成 | SQLCipher 加密数据库 |
| 1.9 | 交叉签名（IK→SSK→DSK） | univona-client | ✅ 已完成 | 与 1.4 合并实现 |
| 1.10 | Flutter 身份 UI（FFI 入口） | univona-client | ✅ 已完成 | 最小实现：创建/恢复/验证 |
| 1.11 | 服务端 UUID 认证模块 | univona-daemon | ✅ 已完成 | 挑战-响应 + JWT 令牌 |
| 1.12 | 预密钥存储服务 | univona-daemon | ✅ 已完成 | 上传/获取/消费/计数 |
| 1.13 | 服务端验证增强 | univona-daemon | ✅ 已完成 | IntoResponse + request-id |

---

### 任务依赖关系

```
准备 (shared) ──┬──→ 1.1 → 1.2 ──┬──→ 1.3 → 1.10 (Flutter FFI)
                │                  ├──→ 1.4+1.9 → 1.5 → 1.6 → 1.7
                │                  └──→ 1.8（可并行）
                └──→ 1.13 → 1.11 → 1.12
```

---

### 验证清单

- [x] `cargo build --workspace` 在 univona-client 编译通过
- [x] `cargo test -p univona-core` 71 个测试全部通过
- [x] 身份生成：UUID 格式 `nxl:` + Base58，24 词助记词
- [x] 助记词恢复：相同词组 → 相同 UUID（确定性恢复）
- [x] X3DH + 双棘轮：两个本地实例成功加解密
- [x] Sender Keys：3 人群组加解密正常
- [x] 服务端认证端点编译通过（需数据库环境做集成测试）
- [x] 预密钥 API 编译通过（需数据库环境做集成测试）
- [x] 所有端点返回统一 `ApiResponse` 格式

---

### 代码统计

| 仓库 | 新增文件 | 新增代码行数 | 测试数量 |
|------|----------|-------------|----------|
| univona（共享层） | 1 文件（auth.rs） | ~90 行 | 4 个测试 |
| univona-client（核心） | 21 文件 | ~2,827 行 | 71 个测试 |
| univona-daemon（服务端） | 12 文件 | ~733 行 | 编译通过（需 DB 集成测试） |
| .github（进度追踪） | 1 文件 | 本文档 | — |

---

### univona-core 模块结构

```
crates/univona-core/src/
  lib.rs                    # 模块导出 + FFI 入口函数
  error.rs                  # CoreError 错误类型
  identity/
    mod.rs                  # 身份模块入口
    keys.rs                 # IdentityKeyPair、PublicIdentityKey
    uuid.rs                 # UnivonaId 推导
    mnemonic.rs             # BIP-39 助记词
    certificate.rs          # DeviceCertificateBuilder 设备证书
    cross_sign.rs           # SelfSigningKey、CrossSignVerifier 交叉签名
  crypto/
    mod.rs                  # 加密模块入口
    x3dh.rs                 # X3DH 密钥协商协议
    ratchet.rs              # 双棘轮（Double Ratchet）
    sender_key.rs           # Sender Keys 群组加密
    kdf.rs                  # HKDF-SHA256 密钥派生
    aead.rs                 # AES-256-GCM 加解密
  keystore/
    mod.rs                  # KeyStore trait 抽象
    software.rs             # Easy Mode（OS 密钥链实现）
  storage/
    mod.rs                  # LocalStore trait 抽象
    encrypted_db.rs         # SQLCipher 加密数据库实现
    schema.sql              # 本地数据表定义
```

---

### univona-daemon 新增 API 端点

| 方法 | 路径 | 说明 | 认证 |
|------|------|------|------|
| POST | `/api/v1/auth/challenge` | 请求认证挑战 | 无需 |
| POST | `/api/v1/auth/verify` | 验证挑战响应，获取 JWT | 无需 |
| POST | `/api/v1/auth/register` | 注册新成员 | 无需 |
| POST | `/api/v1/auth/refresh` | 刷新访问令牌 | 无需 |
| POST | `/api/v1/prekeys` | 上传预密钥包 | 需要 |
| GET | `/api/v1/prekeys/{member_id}` | 获取成员预密钥包（消费一个 OPK） | 需要 |
| DELETE | `/api/v1/prekeys/{device_id}/{key_id}` | 删除指定预密钥 | 需要 |
| GET | `/api/v1/prekeys/{device_id}/count` | 查询剩余预密钥数量 | 需要 |

---

### 关键技术决策

| 决策 | 原因 |
|------|------|
| 使用 `StaticSecret` 替代 `EphemeralSecret` | X25519 的 `EphemeralSecret.diffie_hellman()` 消费所有权，无法多次 DH |
| `ApiError` 包装器 | Rust 孤儿规则：无法为外部 crate 的 `AppError` 实现 `IntoResponse` |
| `LocalStore` 不要求 `Sync` | `rusqlite::Connection` 内含 `RefCell`，不是 `Sync` |
| `bip39::Mnemonic::from_entropy()` | bip39 v2 移除了 `generate()` 方法 |
| 显式依赖 `curve25519-dalek` | Ed25519 → Montgomery 点转换需要直接访问 |
