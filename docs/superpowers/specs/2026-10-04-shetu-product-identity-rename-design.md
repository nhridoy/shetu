# Shetu product identity rename design

**Status:** Approved by user.
**Date:** 2026-10-04
**Scope:** Rename the TermVault desktop and server project identity to Shetu, using a fresh desktop identity, while keeping existing server accounts and encrypted data compatible.

## Goal

Make Shetu the product name across the desktop application, server-facing release artifacts, source package/module identities, deployment documentation, and active release workflow. The rename should be coherent for new installations and self-hosters, not just a replacement of visible text.

The user approved a fresh Shetu desktop install. A TermVault installation and its local data/keychain entries remain separate and untouched. Shetu signs into the existing server API and can synchronize the user's existing server-side account data after login.

## Identity changes

The implementation should rename current, active product identifiers:

- Desktop visible name, title bars, auth pages, email wording, window titles, and recovery/settings export labels become **Shetu**.
- Tauri `productName` and main executable become `Shetu`; propose the new bundle identifier `com.nhridoy.shetu`.
- Frontend package and Rust crate/binary names become `shetu` where those names are application-owned.
- The Go module/import path follows the existing server repository identity, proposed as `github.com/nhridoy/shetu-server`; the built server executable becomes `shetu-server`.
- The container image becomes `ghcr.io/nhridoy/shetu-server`, with `latest` and release-version tags. Release tarball names and manifest asset fixtures use Shetu naming.
- Current product docs, examples, release notes, and environment-variable documentation use Shetu. Dated audits and historical design/plan documents remain historical records and are not bulk-rewritten.

Exact bundle/module/image names are proposals in this spec; the reviewer can adjust them before the implementation plan.

## Fresh desktop identity and local data

Shetu uses its own bundle ID, application-support directory, default database filename, and keychain service namespace. It does not migrate the old TermVault database or keychain entries. First launch creates Shetu-local state; signing in and syncing may repopulate server-backed encrypted data. The old TermVault installation and local state are not deleted or modified by the Shetu app. The current desktop OAuth flow uses loopback callbacks, so this rename does not introduce or register a custom URI handler.

The updater endpoint/manifest must be specific to Shetu (`shetu-latest.json`) so a TermVault build cannot mistake a Shetu release for a compatible update. Shetu's updater public key is embedded in its configuration and its matching private key is provisioned to the Shetu release workflow before its first signed release. The release tag/version checks and output asset names must match the renamed binary.

## Server compatibility

Rename the Go module, server executable, container image, OAuth app-scheme fallback, and TermVault-prefixed server environment variables to Shetu names. New settings are `SHETU_PORT`, `SHETU_HOST`, `SHETU_APP_SCHEME`, and `SHETU_OAUTH_REDIRECT_URIS`. For one compatibility period, a non-empty Shetu setting takes precedence and the corresponding legacy setting remains a fallback: `TERMVAULT_PORT`, `TERMVAULT_HOST`, the existing generic `APP_SCHEME`, and `TERMVAULT_OAUTH_REDIRECT_URIS`, respectively. `DATABASE_URL` and other generic settings keep their current names. Document aliases and a removal policy in the server release notes.

Keep the `/api/v1` routes, payloads, token behavior, database schema/table names, and existing `DATABASE_URL` behavior compatible. In particular, keep the current server database filename as the default when no database URL is supplied; changing that default could silently make an existing installation appear empty. The current client uses a loopback OAuth callback and no custom deep-link plugin/handler was found in the active desktop source. Keep that loopback path unchanged. Rename the server's app-scheme fallback to Shetu and continue accepting the existing loopback callback URIs; do not add a new custom URI handler as part of this rename unless implementation discovery finds an existing consumer.

## Compatibility boundaries

Some `termvault` strings are data-format or cryptographic identifiers, not brand labels. They must remain stable unless a separately designed migration/dual-read protocol exists. This includes key-derivation/AAD domain strings, serialized payload format identifiers, local/server table names, and event names consumed across independently released components. Renaming these without a compatible reader would make existing ciphertext or persisted records unreadable. The implementation should inventory and explicitly preserve those values.

The repository's existing Linux server binary and GHCR package names can change for new releases. The previous image/package can remain published; do not delete or overwrite artifacts as part of the rename.

## Scope exclusions

- No migration of a TermVault desktop database, OS keychain entries, local settings, or app-support directory into Shetu.
- No changes to server API routes, user IDs, encrypted payload schema, password/recovery derivation, team key envelope formats, or sync protocol.
- No reset, conversion, or rename of an existing server database or existing user account.
- No removal of legacy `TERMVAULT_*` aliases in the first Shetu server release.
- No broad editing of dated audit/specification history.
- No change to public repository URLs, OAuth provider credentials, or the `nhridoy/shetu` monorepo identity.

## Failure handling and security

The new app must fail clearly if it cannot access its own local storage. OAuth login must continue to use the loopback callback used today and must not redirect to an unregistered scheme. Configuration precedence must be deterministic: the Shetu variable wins when set; otherwise the legacy variable is used; otherwise the documented default applies.

The Shetu updater key must be distinct from operating-system signing credentials. Never commit private signing keys. Release publication must fail if the Shetu updater artifacts/signatures or required platform manifest entries are missing. Keep release and registry permission scopes minimal.

## Verification

- Search active client/server/release source for remaining user-facing `TermVault` strings and classify each remaining `termvault` occurrence as a compatibility alias, persisted/wire/cryptographic format, historical document, external dependency, or missed rename.
- Build the desktop targets and server after package/module renames; run frontend, Rust, and Go checks affected by the change.
- On the same machine, install both applications and confirm TermVault and Shetu have distinct app identities, local database paths, keychain entries, protocol registration, and data. Confirm starting Shetu does not mutate the old TermVault database.
- Sign into the same existing server account from Shetu and verify login, OAuth loopback callback, E2E decryption, sync, and existing team/shared-vault access still work.
- Verify server config precedence for paired `SHETU_*`/`TERMVAULT_*` values, legacy-only values, and unset defaults; verify the existing database file is still opened by default.
- Build Shetu release artifacts and verify the Shetu-only update manifest, updater signatures, asset names, and GHCR `shetu-server` tags. Confirm a TermVault updater endpoint does not return Shetu's manifest.

## Implementation approach

Treat this as one identity migration with coordinated client and server repository changes, plus monorepo release/docs updates. Implement the compatibility-sensitive server identifiers and fallback behavior before switching Shetu clients and release artifacts. Preserve the user's existing uncommitted change in `client/src-tauri/tauri.conf.json`; do not overwrite or discard it.

## Open questions for review

1. Confirm `com.nhridoy.shetu` as the desktop bundle identifier, `github.com/nhridoy/shetu-server` as the Go module path, and `ghcr.io/nhridoy/shetu-server` as the public server image.
2. Confirm legacy server environment aliases remain for at least the first Shetu server release.
3. Decide whether current mobile/App Store identifiers are in scope. This spec assumes the requested Windows/Linux/macOS desktop product plus server/release identity; mobile store identity is excluded because changing a store app ID creates a separate installed application.

## Follow-up: complete active identifier rename

After the initial rename was committed, the user requested that remaining active Shetu-owned identifiers also be renamed. The current desktop emits `shetu:` window event names, uses `.shetu-diff-view`, writes the `shetu-revoked-team-edits-v1` export format, uses a Shetu namespace for session-retention row IDs, and seals new team-key envelopes with a Shetu cryptographic domain. Readers retain the prior key-envelope and preference-ID domains so encrypted team access and saved retention settings remain usable.

The server default SQLite filename is `shetu.db`. It opens the configured `DATABASE_URL` directly; it does not inspect, rename, or migrate another filename. Shetu-specific environment variables are the only accepted names.

The rename applies to active app/server/release identifiers and product branding. Stable encryption/AAD domains and stored preference IDs retain dual-read support where needed so Shetu can still read already-synced encrypted data; these are data-format identifiers, not product names. Dated audit/specification documents remain historical records. The separate mobile store ID remains out of scope because the mobile source and matching store identity are not present in this checkout.

Current product guides and the top-level project plan use Shetu. Dated audits and implementation records remain historical. The mobile store package ID remains excluded because this checkout does not include the mobile source or matching store identity configuration; changing only its workflow destination would make mobile publishing fail.
