# Univona

**开源混合架构即时通讯平台** — 中枢服务器 + 去中心化 P2P

## 愿景

Univona（"Uni" 统一 + "Vona" 声音）致力于构建一个兼具中心化便利和去中心化自主的即时通讯平台。用户可自主选择连接模式：官方目录、自建社区、或纯 P2P。

## 安全架构

- **端到端加密** — Signal Protocol (X3DH + Double Ratchet + ChaCha20-Poly1305)
- **身份自主** — BIP-39 助记词生成 Ed25519 密钥对，无需手机号或邮箱
- **零知识服务器** — 服务端仅路由密文，不具备解密能力

## 仓库导航

| 仓库 | 说明 | 技术栈 |
|------|------|--------|
| [univona](https://github.com/univona/univona) | 共享类型库 + Protobuf 协议定义 | Rust · Protobuf |
| [univona-client](https://github.com/univona/univona-client) | 跨平台客户端（Android/iOS/桌面） | Flutter · Rust FFI |
| [univona-daemon](https://github.com/univona/univona-daemon) | 社区服务端 — 轻量级消息路由核心 | Rust · Axum · PostgreSQL |
| [univona-panel](https://github.com/univona/univona-panel) | 管理面板 — 多 Daemon 统一管理 | Rust · React · Ant Design |
| [univona-admin-server](https://github.com/univona/univona-admin-server) | 官方一体化服务端 | Rust · Axum |
| [univona-docker-dev](https://github.com/univona/univona-docker-dev) | 开发环境 Docker 配置 | Docker Compose |

## 三种连接模式

```
模式 A: 官方目录 ← Client → Admin Server
模式 B: 自建社区 ← Client → Daemon (由 Panel 管理)
模式 C: 点对点   ← Client → Client (libp2p)
```

## 状态

Phase 1 (核心基础设施) — 已完成

---

采用 **Rust + Flutter** 构建 · 许可证 **AGPL-3.0**
