# s1x — Installation & Configuration Guide

`s1x` is a private, headless build of RustDesk. The remote-desktop engine is
unchanged; the difference is that this build shows **no interface at all** —
no main window, no tray icon, no connection-manager popup, no install
wizard. It is meant to run silently as a background service and be
configured entirely through its config file and CLI flags.

## Contents of this repo

| File | Purpose |
|---|---|
| `s1x.exe` | Main binary (client, server, service — one executable) |
| `sciter.dll` | Required UI runtime dependency. Must sit next to `s1x.exe` even though the local UI is disabled — the binary still links it and will fail to start without it. |
| `S1X_INSTALL.md` | This guide |

## 1. Install

Copy `s1x.exe` and `sciter.dll` into the same folder (either run it portable
from there, or install it as a Windows service — see below).

### Option A — Silent install as a Windows service (recommended)

Run from an elevated (Administrator) command prompt:

```
s1x.exe --silent-install
```

This installs the app under Program Files, registers the background
service, and does **not** show any wizard or dialog (a toast notification
reporting success/failure is the only visible artifact).

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

### Just running it directly

Double-clicking `s1x.exe` or running it with no arguments does **not** open
any window — it silently starts the background server and parks. This is
intentional in this fork (upstream RustDesk would normally show the main
window here).

## 2. Configure

All configuration is done via the config file or CLI — there is no settings
UI to open.

### Config file location

```
%APPDATA%\s1x\config\s1x.toml      (main config: id, key, servers)
%APPDATA%\s1x\config\s1x2.toml     (options: password, approve-mode, etc.)
```

You can hand-edit these while the service is stopped, or use the CLI
(below) while it's running — the CLI talks to the running instance over
IPC and requires the app to be **installed** and the shell to be
**elevated/root**.

### Set a permanent password (do this first)

```
s1x.exe --password "your-strong-password"
```

### ⚠️ Required setting: password-only approval

This build hard-disables the connection-manager popup (`--cm`), which is
what upstream RustDesk normally pops up on the host to let a human
click "Accept" for an incoming connection. In this fork **that window can
never appear**, so if approval mode is left at anything that expects a
click, incoming connections will simply hang forever with nothing to
click.

You must set approval mode to permanent-password-only:

```
s1x.exe --option approve-mode password
```

Do not set `approve-mode` to `click` or `both` on this build.

### Other useful options

```
s1x.exe --get-id                          Print this machine's ID
s1x.exe --config <server-config-string>   Apply a custom server config (self-hosted rendezvous/relay)
s1x.exe --option <key> <value>            Set any option key (see libs/hbb_common/src/config.rs for the full list)
```

### Using it as an outgoing viewer

Even though the host side has no UI, you can still use this machine to
*look at* another machine — that viewer window is intentionally kept:

```
s1x.exe --connect <remote-id>
```

## 3. Notes / limitations of this fork

- No tray icon will ever appear, even if config asks for one.
- No install wizard — only `--silent-install` / `--silent-install debug`.
- The "show my cursor" whiteboard overlay feature is disabled host-side.
- `approve-mode` **must** be `password`. There is no way to manually
  approve a connection on this machine once installed.
