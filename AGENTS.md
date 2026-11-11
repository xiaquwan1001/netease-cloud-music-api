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

- 本仓库 `gmg137/netease-cloud-music-api`（crate 名 `netease-cloud-music-api`）是 `netease-cloud-music-gtk` 的 API 库，本地路径 `~/netease-cloud-music-api`。
- 与客户端配套开发：登录风控、穿戴端 eapi、网页登录、出口限流、PR 状态的主记忆在 `~/netease-cloud-music-gtk/.pi/memory/MEMORY.md`，本段只做指向。
- 穿戴端登录已落地并合入：`MusicApi::captcha_wearable` / `login_cellphone_wearable`，走 eapi 可绕过 PC 端 -462 风控。
- 穿戴端响应是原始二进制密文，解密入口 `Crypto::eapi_res_decrypt_body`（不能按普通 eapi JSON 解）。
- 错误文案透传走 `model::api_error`；`to_captcha` 对 `code=200 但 data=false` 判为失败。
- PR #21 于 2026-11-03 合入上游 master（merge 提交 `bcf36ff`），fork 分支已删、本地 master 已快进；上游未出新 tag，最新仍是 2.2.2。
- 合并后 master 上 `cargo test` = 13 passed / 1 ignored；旧的「2 tests」是未含穿戴端用例的基线。
- 联调/探测工具在 `~/netease-cloud-music-gtk/.dev/probe`，以 path 依赖本仓库。
