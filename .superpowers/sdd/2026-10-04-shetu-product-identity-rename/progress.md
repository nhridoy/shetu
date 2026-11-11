# SDD ledger — Shetu product identity rename

Plan: `docs/superpowers/plans/2026-10-04-shetu-product-identity-rename.md`  
Workspace: existing branch and submodules; no worktree or branch switch. No commits or pushes, per user instruction.

User-owned updater public key in `client/src-tauri/tauri.conf.json` was preserved while updating the Shetu app identity and feed.

## Implementation status

- Desktop: Shetu product/binary/package names, bundle ID `com.nhridoy.shetu`, separate `shetu.db` and local data namespace, Shetu UI/export branding, and `shetu-latest.json` updater feed are in place. Legacy server/wire/crypto identifiers are preserved where the approved design requires compatibility.
- Server: Go module/import paths and executable entry point are `github.com/nhridoy/shetu-server` and `cmd/shetu-server`; Shetu env vars take precedence with existing aliases retained. The default database is `shetu.db`; SQLite data at the previous filename is migrated before opening.
- Release: CI and release workflows build/publish Shetu desktop/server names and `ghcr.io/<owner>/shetu-server`. The manifest builder emits `shetu-latest.json`.
- Documentation: active setup, release, and manual verification docs describe Shetu and the separate desktop identity. Historical audit/spec/plan docs and the separate mobile store identity remain unchanged.

## Verification

- `GOCACHE=/tmp/shetu-go-cache go test ./...`, `go vet ./...`, and `go build -o /tmp/shetu-server ./cmd/shetu-server`: passed.
- `node --test --test-isolation=none scripts/build-update-manifest.test.mjs`: 6/6 passed.
- `cargo check --tests --manifest-path client/src-tauri/Cargo.toml`: passed with existing dead-code warnings.
- `git diff --check` at root and both submodules: passed.
- Tauri config JSON parsed and reports Shetu product/binary/bundle identity and Shetu updater endpoint; configured updater public key retained.
- Frontend Vitest/lint/build could not run because dependencies are not available locally and registry access fails DNS resolution. Rust unit tests were not run; earlier Tauri test linking did not complete. No desktop dev server was started.

## Manual verification remaining

Install and run the previous app and Shetu side by side; verify local DB/keychain separation and preserved old data; sign in to an existing account, sync/decrypt data, test OAuth, confirm server DB migration preserves records, and verify Shetu uses only its signed updater feed. Cross-platform installer/signing checks require builds on their target environments.

## Follow-up: complete active identifier rename

The user asked to rename remaining active identifiers after the original rename was pushed. Active desktop events, CSS selectors, export format labels, team-key write domains, session preference IDs, server environment variables, and the default database filename now use Shetu identifiers. The server opens the configured database URL directly, defaults to `shetu.db`, and does not migrate or discover another filename. Old encryption and preference identifiers remain read-only compatibility formats. Historical execution records remain historical.

- `GOCACHE=/tmp/shetu-go-cache go test ./internal/config ./cmd/shetu-server -count=1`: passed after DB migration implementation.
- `cargo check --tests --manifest-path client/src-tauri/Cargo.toml`: passed after adding legacy envelope coverage.
- `cargo test --manifest-path client/src-tauri/Cargo.toml --lib`: 118 passed.
