# Changelog

All notable changes to paseo-relay are documented here.

## [v0.5.1-security.1] - 2026-10-22

### Security
- Require Go 1.26.10 in `go.mod`. Release builds of v0.5.0 were compiled with Go 1.26.2 and miss every standard-library security fix released since, including the 2026-10-08 net/http, crypto/tls and net/textproto fixes (CVE-2026-94439, CVE-2026-97032, CVE-2026-78659, CVE-2026-94440, …) and the 2026-10-21 crypto/x509 fix (CVE-2026-111943). Older toolchains now switch to Go 1.26.10 automatically (`GOTOOLCHAIN=auto`) or refuse to build (`GOTOOLCHAIN=local`, e.g. a stale `golang:1.26-alpine` image; build with `docker build --pull`)
- `/health` only answers GET and HEAD; other methods get 405. Previously `CONNECT /health` got a 200, which made the HTTP/1 connection desync CVE-2026-94439 reachable

### Changed
- WebSocket message limit raised from 10 MB to 32 MiB to match the hosted relay (upstream PR #5, merged after v0.5.0 but not yet released upstream)

## [v0.5.0] - 2026-04-11

### Performance
- Reduce lock contention on forwarding hot-path: extract per-connectionId state into `pipe` struct with its own mutex; message forwarding no longer touches the session-level lock
- `connectedConnectionIds` now iterates pipes under a single lock instead of double-locking

### Fixed
- Fix race condition in `nudgeOrResetControl`: inner timer now re-acquires lock and compares control pointer before closing, preventing accidental close of a replaced connection

### Changed
- Session cleanup is now active (on handler exit) instead of relying solely on the 60s eviction ticker; eviction loop kept as a 5-minute backstop
- `frameBuffer` now enforces a 32 MB total byte limit in addition to the frame count limit, evicting oldest frames when the limit is reached
- Scope GitHub Actions release workflow permissions per job (least privilege)

### Security
- Remove weak time-based fallback in `randomHex`; `crypto/rand` failure now panics instead of silently degrading to predictable IDs

## [v0.4.0] - 2026-04-11

### Security
- Enforce 10 MB per-message size limit on all WebSocket connections to prevent memory exhaustion DoS
- Cap concurrent sessions at 10,000 to prevent unbounded session creation attacks

## [v0.3.0] - 2026-04-11

### Changed
- Change default listen port from 8080 to 8411
- Add curl install commands for each platform binary in README

## [v0.2.1] - 2026-04-11

### Docs
- Add IP address config example alongside domain example in README

## [v0.2.0] - 2026-04-11

### Changed
- Replace `log` with `log/slog` for structured logging
- Upgrade Go version to 1.26

## [v0.1.0] - 2026-04-11

### Added
- Initial release: zero-knowledge WebSocket relay server for Paseo
- Protocol v2 with server control socket, server data socket, and client socket roles
- In-memory session registry with periodic eviction
- Frame buffering while daemon data socket connects
- Graceful shutdown support
