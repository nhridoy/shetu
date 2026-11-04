# Release runbook

The client and the server release separately, each from its own repository. The monorepo does not release.

| Repo | Tag | Workflow builds | Publishes |
|------|-----|-----------------|-----------|
| `nhridoy/terra-client` | `v<client version>` | Signed desktop apps for Windows (NSIS), Linux (AppImage, `.deb`), and macOS, each for x64 and ARM64 | GitHub Release with `terra-latest.json` (the in-app update feed) |
| `nhridoy/terra-server` | `v<server version>` | `go vet` and tests, Linux server binaries for amd64 and arm64 | GitHub Release with both tarballs and `SHA256SUMS`, plus `ghcr.io/nhridoy/terra-server:<version>` for `linux/amd64` and `linux/arm64` |

A tag with a `-` (such as `v0.1.0-beta.1`) becomes a prerelease. Prereleases do not move the server image's `latest` tag, and installed apps never see them as updates, because the updater reads GitHub's `/releases/latest`, which skips prereleases.

## One-time setup

### Client (`nhridoy/terra-client`)

1. In Settings → Environments, create `Release`.
2. Add secret `TAURI_SIGNING_PRIVATE_KEY` with the updater private key file contents. Add `TAURI_SIGNING_PRIVATE_KEY_PASSWORD` only if the key has a password. The key's public half is embedded in `src-tauri/tauri.conf.json`; never replace it silently, because installed apps trust only that key.
3. Add variable `VITE_API_URL` (the default server URL compiled into the app).
4. Keep the repository public. Unauthenticated apps cannot read release assets from a private repository.

### Server (`nhridoy/terra-server`)

No secrets are needed; the workflow pushes to GHCR with its own `GITHUB_TOKEN`. If the `terra-server` package was first published from another repository, open the package's settings on GitHub → **Manage Actions access** and give `nhridoy/terra-server` the **Write** role, or the image push fails with `permission_denied`.

## Release the client

1. Bump the version in `package.json`, `src-tauri/tauri.conf.json`, `src-tauri/Cargo.toml`, and the `terra` entry in `src-tauri/Cargo.lock`. The workflow stops if the tag differs from the `tauri.conf.json` version.
2. Commit, push `main`, then tag and push:

   ```bash
   VERSION=0.0.3
   git commit -am "chore(release): set version $VERSION"
   git push origin main
   git tag -a "v$VERSION" -m "Terra $VERSION"
   git push origin "v$VERSION"
   gh run watch
   ```

3. Check the release has `terra-latest.json` and, for both x64 and ARM64 (Intel and Apple Silicon on macOS), named `Terra-<version>-<OS>-<CPU>`, the Windows `.exe`, the macOS `.app.tar.gz` and `.dmg`, and the Linux `.AppImage` and `.deb`, with a `.sig` for each updater bundle. The manifest has entries for `linux-x86_64`, `linux-x86_64-deb`, `linux-aarch64`, `linux-aarch64-deb`, `windows-x86_64`, `windows-aarch64`, `darwin-x86_64`, and `darwin-aarch64`.

   To test the builds without publishing, run the workflow by hand (`gh workflow run release.yml -R nhridoy/terra-client`); manual runs build every platform and skip the release.
4. Install the previous build and use Settings → Check for updates. Confirm download progress, Restart & install, and the new version after relaunch. On `.deb`, confirm the password prompt and `dpkg -l terra`.

### Bridge for apps installed from the monorepo (0.0.2 and older)

Builds up to 0.0.2 read the update feed from the monorepo's latest release (`nhridoy/terra` `v0.0.2`). After the **first** client-repo release, copy its manifest there once. Those apps then update to the new build, which reads the client repo's feed from then on:

```bash
VERSION=0.0.3
gh release download "v$VERSION" -R nhridoy/terra-client -p terra-latest.json -D /tmp/terra-bridge
gh release upload v0.0.2 -R nhridoy/terra /tmp/terra-bridge/terra-latest.json --clobber
```

## Release the server

The server has no version file; the tag is the version.

```bash
git tag -a v0.0.3 -m "Terra server 0.0.3"
git push origin v0.0.3
gh run watch
```

Check the release has `terra-server-linux-amd64.tar.gz`, `terra-server-linux-arm64.tar.gz`, and `SHA256SUMS`, and that `docker manifest inspect ghcr.io/nhridoy/terra-server:0.0.3` lists both `linux/amd64` and `linux/arm64`.

## Failed runs, rollback, and key loss

If a run fails before the release is published, delete the tag, push the fix, and tag again:

```bash
git tag -d "v$VERSION"
git push origin ":refs/tags/v$VERSION"
```

Never reuse a published version. The updater accepts only newer versions, so recover from a bad release by publishing a higher one. If the signing key is compromised, stop publishing and ship a new installer with a new public key through a trusted manual channel; installed builds cannot trust a key they never received.
