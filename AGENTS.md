# AGENTS.md

## Project

Rust library crate implementing the Netease Cloud Music API. v2.2.2（以 `Cargo.toml` 为准）, GPLv3.
本地路径 `~/netease-cloud-music-api`，上游 `gmg137/netease-cloud-music-api`（`netease-cloud-music-gtk` 的 API 库）。

## Structure

- `src/lib.rs` — `MusicApi` struct with all public API methods, HTTP client, test suite
- `src/model.rs` — data models and JSON response parsers
- `src/encrypt.rs` — AES/RSA encryption for Netease's custom API protocol (Weapi, LinuxApi, Eapi)

Library crate only — no binary, no `main.rs`.

## Build & Test

```sh
cargo build
cargo test        # 测试统一用 async-std 运行时；2026-11-11 基线 = 13 passed / 1 ignored（旧「2 tests」是未含穿戴端用例的基线）
```

All public API methods are marked `#[allow(unused)]` — intentional for library consumption.

## Key Dependencies

- **HTTP**: `isahc` v2 with cookies (not reqwest)
- **Async runtime (tests)**: `async-std` with `#[async_std::test]` (not tokio)
- **Encryption**: `openssl` (AES-CBC/ECB, RSA), `base64`, `hex`
- **Serialization**: `serde` + `serde_json`
- **`rand` ~0.9** — recently upgraded from 0.8, notable breaking changes
- **`urlqstring`** — query parameter construction (custom)

## Conventions

- `CryptoApi` variants: `Weapi` (default), `LinuxApi`, `Eapi` — selected per-request
- `Parse` enum controls JSON response parsing strategy (Usl, Ucd, Rmd, Rmds, Search, etc.)
- `.gitignore` excludes `/target` and `/Cargo.lock`

## 项目记忆（2026-11-11 整理）

- 协议/加解密/风控只用 `Crypto`/`MusicApi` 现有实现，不自行重写加密或签名。
- 错误文案透传走 `model::api_error`；`to_captcha` 对 `code=200 但 data=false` 判失败。
- 穿戴端登录已合入上游（2026-11-03，merge `bcf36ff`；上游未出新 tag，最新仍 2.2.2）：`captcha_wearable` / `login_cellphone_wearable` 走 eapi 可绕过 PC 端 -462 风控。
- 细节见本仓 `.pi/memory/MEMORY.md` 与 `~/netease-cloud-music-gtk/.pi/memory/MEMORY.md`（主记忆：登录风控/易盾/eapi 协议/出口限流/PR 状态）。
