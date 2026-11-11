# AGENTS.md

## Project

Rust library crate implementing the Netease Cloud Music API. v1.5.2, GPLv3.

## Structure

- `src/lib.rs` — `MusicApi` struct with all public API methods, HTTP client, test suite
- `src/model.rs` — data models and JSON response parsers
- `src/encrypt.rs` — AES/RSA encryption for Netease's custom API protocol (Weapi, LinuxApi, Eapi)

Library crate only — no binary, no `main.rs`.

## Build & Test

```sh
cargo build
cargo test        # 2 tests, both use async-std runtime
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
- `#[allow(unused)]` on public API methods is by design
- `.gitignore` excludes `/target` and `/Cargo.lock`

## 项目记忆（2026-11-11 整理）

- 穿戴端登录已落地并合入：`MusicApi::captcha_wearable` / `login_cellphone_wearable`，走 eapi 可绕过 PC 端 -462 风控。
- 协议/加解密/风控只用 `Crypto`/`MusicApi` 现有实现，不自行重写加密或签名。
- 错误文案透传走 `model::api_error`；`to_captcha` 对 `code=200 但 data=false` 判失败。
- 细节见 `.pi/memory/notes-api.md`（本仓）与 `~/netease-cloud-music-gtk/.pi/memory/`。

