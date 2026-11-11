<p align="center">
  <img src="https://raw.githubusercontent.com/nhridoy/terra-client/main/src-tauri/icons/icon.png" width="96" height="96" alt="Terra app icon">
</p>

<h1 align="center">Terra</h1>

<p align="center">
  The free, open-source SSH client for macOS, Windows and Linux, and a Termius alternative.<br>
  Split terminals, SFTP, port forwarding and an encrypted vault you can sync through your own server.
</p>

<p align="center">
  <a href="https://terra.iamnahid.com">Website</a> ·
  <a href="https://terra.iamnahid.com/download">Download</a> ·
  <a href="https://terra.iamnahid.com/docs">Docs</a> ·
  <a href="https://terra.iamnahid.com/changelog">Changelog</a> ·
  <a href="https://terra.iamnahid.com/termius-alternative">Terra vs Termius</a>
</p>

<p align="center">
  <img src="https://terra.iamnahid.com/og/index.png" width="720" alt="Terra: free open-source SSH client and Termius alternative">
</p>

This is the project home. The code lives in three repositories, included here as Git submodules:

| Repository | What it is |
|---|---|
| [terra-client](https://github.com/nhridoy/terra-client) | The desktop app: Tauri v2, Rust, React and TypeScript |
| [terra-server](https://github.com/nhridoy/terra-server) | The sync server you can host yourself: Go, Gin, GORM, PostgreSQL |
| [terra-web](https://github.com/nhridoy/terra-web) | The website and docs at [terra.iamnahid.com](https://terra.iamnahid.com): Astro |

## Features

- **Terminal** with tabs, split panes, focus mode and a local shell; 36 themes and 8 terminal fonts
- **Hosts** in nested groups with tags, colors and OS icons
- **SFTP** with local and remote panes, drag and drop, chmod, chown and search
- **Port forwarding:** local, remote and dynamic (SOCKS5)
- **Keys:** generate Ed25519, RSA 4096 or ECDSA P-256, or import OpenSSH and PEM keys
- **Editor** with about 20 languages, diffs, remote editing over SFTP and a Git panel
- **Workspaces** that save and restore tab and pane layouts
- **Encrypted sync and team vaults** through a Terra server, with owner, admin and member roles

Not there yet: mobile apps, Mosh, Telnet and serial, and importing from Termius or `~/.ssh/config`.

## Security

Your vault is encrypted on your device before it is synced: **Argon2id** derives a key from your account password, and records are sealed with **XChaCha20-Poly1305**. Team vaults have their own keys, sealed to each member with **X25519** and rotated when a member is removed. The server stores ciphertext and never decrypts it; it can see record names, tags and timestamps, while host addresses, usernames, passwords, private keys and commands stay encrypted. SSH connects directly from the app to your servers.

Details: [security reference](https://terra.iamnahid.com/docs/security). Report vulnerabilities privately through [GitHub Security Advisories](https://github.com/nhridoy/terra/security/advisories/new).

## Download

Get the desktop app from the [download page](https://terra.iamnahid.com/download) or [terra-client releases](https://github.com/nhridoy/terra-client/releases/latest), or install it with one command:

```bash
# macOS or Linux
curl -fsSL https://raw.githubusercontent.com/nhridoy/terra-client/main/install.sh | bash
```

```powershell
# Windows (PowerShell)
irm https://raw.githubusercontent.com/nhridoy/terra-client/main/install.ps1 | iex
```

Builds are not notarized by Apple or signed by Microsoft yet, so a file you download yourself shows a one-time warning. The [client README](https://github.com/nhridoy/terra-client#download) explains what to click.

## Self-host the sync server

Sign-in and sync go through a Terra server. Run your own with Docker and PostgreSQL: see the [terra-server README](https://github.com/nhridoy/terra-server) and the [self-hosting guide](https://terra.iamnahid.com/docs/server).

## Development

Each part has its own README with setup steps. To work on one, clone its repository directly:

```bash
git clone https://github.com/nhridoy/terra-client.git   # desktop app (bun install && bun tauri dev)
git clone https://github.com/nhridoy/terra-server.git   # server (go run ./cmd/terra-server)
git clone https://github.com/nhridoy/terra-web.git      # website (pnpm install && pnpm dev)
```

See [CONTRIBUTING.md](CONTRIBUTING.md) for prerequisites and conventions.

## Roadmap

What's planned, roughly in order. There are no dates, and priorities can change with feedback; [open an issue](https://github.com/nhridoy/terra/issues) to vote for something or suggest it.

**Next**

- **Trust and polish:** sign and notarize the macOS and Windows builds so they install without warnings; a working "Delete all data" option in settings; repair the key type shown for SSH keys imported by older versions.
- **Verification:** finish live two-device testing of sync, teams and shared vaults, session history, saved workspaces and the jump host on all three platforms.
- **Productivity:** run snippets straight from the command palette, customizable keyboard shortcuts, and global SSH options in settings.

**Later**

- **More ways to connect:** jump chains through more than one bastion, then Telnet, Mosh and serial connections, shaped by demand.
- **Smarter terminal:** context-aware command autocomplete.
- **Terra Cloud:** a hosted, managed sync option for people who'd rather not run a server.

**Exploring**

- **Mobile apps** for iOS and Android.

Done so far is in the [changelog](https://terra.iamnahid.com/changelog).

## Releases

The client and the server release separately, each from its own repository, by pushing a `v*` tag there:

- **Client** (`nhridoy/terra-client`): builds the desktop apps for Windows, Linux and macOS, each for x64 and ARM64, and publishes the in-app update feed. The tag must match the version in `src-tauri/tauri.conf.json`.
- **Server** (`nhridoy/terra-server`): runs the tests, publishes Linux binaries for amd64 and arm64, and pushes `ghcr.io/nhridoy/terra-server` for `linux/amd64` and `linux/arm64`.

Tags with a `-` (such as `v0.1.0-beta.1`) become prereleases, which installed apps do not offer as updates. See the [release runbook](docs/UPDATER_RELEASE_RUNBOOK.md) for one-time setup and release steps.

## Support

- Questions and bugs: [GitHub Issues](https://github.com/nhridoy/terra/issues)
- Desktop app bugs (with logs): [terra-client issues](https://github.com/nhridoy/terra-client/issues/new?template=bug_report.yml)

## License

[MIT](LICENSE)
