# s1x — Installation & Configuration Guide

`s1x` is a headless, **unattended-access** build of RustDesk for machines you
own or administer remotely — servers, VMs, and workstations in your own fleet
where nobody is sitting at the keyboard to click through dialogs. The
remote-desktop engine is unchanged from upstream RustDesk; the only difference
is that this build runs as a background service with no local UI (no main
window, tray icon, connection-manager popup, or setup wizard) and is
configured entirely through its config file and CLI flags. This is the same
model as "unattended access" in TeamViewer / AnyDesk.

> **Intended use.** Deploy this only on endpoints you own or are authorized to
> administer. Because it authenticates unattended (by a fixed password, with no
> on-screen approval step), it is meant for your own infrastructure, not for
> machines where another person would need to consent to each session.

## Contents of this repo

| File | Purpose |
|---|---|
| `s1x.exe` | Main binary (client, server, service — one executable) |
| `sciter.dll` | UI runtime dependency. Must sit next to `s1x.exe` even though the local UI is disabled — the binary still links it and won't start without it. |
| `S1X_INSTALL.md` | This guide |

## 1. Install

Copy `s1x.exe` and `sciter.dll` into the same folder — run it portable from
there, or install it as a Windows service (see below).

### Option A — Unattended install as a Windows service (recommended)

Run from an elevated (Administrator) command prompt:

```
s1x.exe --silent-install
```

This installs the app under Program Files and registers the background
service without an interactive wizard — a toast notification reports whether
it succeeded or failed. The non-interactive flow is what lets you roll it out
by script across several of your own machines.

To remove it later:

```
s1x.exe --uninstall
```

### Option B — Portable / manual service registration

```
s1x.exe --install-service
```

```
s1x.exe --uninstall-service
```

### Running it directly

Running `s1x.exe` with no arguments starts the background server and parks.
No window opens, because this build has no host-side UI (upstream RustDesk
would show the main window here). Use the CLI and config file to manage it.

## 2. Configure

All configuration is done through the config file or CLI — this build has no
settings window.

### Config file location

```
%APPDATA%\s1x\config\s1x.toml      (main config: id, key, servers)
%APPDATA%\s1x\config\s1x2.toml     (options: password, approve-mode, etc.)
```

Hand-edit these while the service is stopped, or use the CLI (below) while
it's running — the CLI talks to the running instance over IPC and requires
the app to be **installed** and the shell to be **elevated/root**.

### Set an access password (do this first)

```
s1x.exe --password "your-strong-password"
```

### Required setting: password-based (unattended) approval

Because this is an unattended build, there is no local operator to click
"Accept" on an incoming session, so the connection-manager popup (`--cm`) that
upstream RustDesk shows is disabled. Approval is therefore handled
automatically by the access password you set above:

```
s1x.exe --option approve-mode password
```

Leave `approve-mode` at `password`. Don't use `click` or `both` on this
build — those modes wait for a local click that can't happen on a headless
machine, so incoming sessions would just hang.

### Other useful options

```
s1x.exe --get-id                          Print this machine's ID
s1x.exe --config <server-config-string>   Apply a custom server config (self-hosted rendezvous/relay)
s1x.exe --option <key> <value>            Set any option key (see libs/hbb_common/src/config.rs for the full list)
```

### Using it as an outgoing viewer

The host side has no UI, but you can still use this machine to connect out to
another one — the viewer window is kept:

```
s1x.exe --connect <remote-id>
```

## 3. Notes / limitations of this fork

- No tray icon, even if the config requests one.
- No interactive install wizard — use `--silent-install` (or
  `--silent-install debug` for verbose output).
- The "show my cursor" whiteboard overlay is disabled host-side.
- `approve-mode` must be `password`: unattended endpoints authenticate by
  password rather than by an on-screen prompt.
