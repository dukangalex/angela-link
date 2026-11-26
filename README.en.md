# AngelaLink — a local bridge for any AI (Chinese alias: 安渡 / Andu, "safe crossing")

Let an AI work with files on your Windows PC inside folders you authorize,
and optionally run project commands inside a write-restricted sandbox with
local approval dialogs.

AngelaLink is agent-neutral: the desktop app owns authorization, sandboxing,
approvals and audit logs; any AI or agent script that can make HTTP requests
and keep a bearer token can connect through the `/agent/v1` API.
See [docs/AGENT-BRIEF.md](docs/AGENT-BRIEF.md) for the connection brief and
[docs/ROADMAP.md](docs/ROADMAP.md) for the generalization roadmap.

Highlights:

- Per-folder or per-drive grants, with search/read/write authorized separately.
  Credential and system paths are protected by default.
- Optional command execution (off by default) in a Windows write-restricted
  sandbox (restricted token + per-workspace SID ACLs + Job Object limits),
  with three-tier local approvals and a 120-second fail-closed timeout.
- Reachable over Tailscale Funnel or a Cloudflare named tunnel, using only
  the tunnels already installed on your machine — no third-party relay.
- Per-client tokens that can be rotated or revoked; rotation immediately
  aborts all running commands. Logs redact tokens automatically.
- Auto-update is off unless a deployment explicitly configures an update
  endpoint (HTTPS + signature verification).

Build from source (Windows, WebView2, Node 22.12+, pnpm, Rust MSVC):

```
pnpm -C angela-link install --frozen-lockfile
pnpm -C angela-link tauri build --bundles nsis
```

License: MIT (see LICENSE and THIRD_PARTY_NOTICES.md).
Self-owned fork of [Aenvo/MuseLink](https://github.com/Aenvo/MuseLink) (MIT);
see CREDITS.md. Maintained independently, not tracking upstream.
