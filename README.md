# 🐷 Pigbot — Personal AI Assistant

**Pigbot** is a *personal AI assistant* you run on your own devices.
It answers you on the channels you already use (WhatsApp, Telegram, Slack, Discord, Google Chat, Signal, iMessage, Microsoft Teams, etc.), plus extension channels like BlueBubbles, Matrix, Zalo, and Zalo Personal. It can speak and listen on macOS/iOS/Android, and can render a live Canvas you control. The Gateway is just the control plane — the product is the assistant.

![screenshot](https://raw.githubusercontent.com/mochiyaki/pigbot/master/pigbot.png)

# Pigbot project details (part 1)

If you want a personal, single-user assistant that feels local, fast, and always-on, this is it.

## Models (selection + auth)

- Models config + CLI
- Auth profile rotation (OAuth vs API keys) + fallbacks

## Install (recommended)

Runtime: **Node ≥22**.

```bash
npm install -g pigbot@latest
# or: pnpm add -g pigbot@latest

pigbot onboard --install-daemon
```

The wizard installs the Gateway daemon (launchd/systemd user service) so it stays running.

## Quick start (TL;DR)

Runtime: **Node ≥22**.

```bash
pigbot onboard --install-daemon

pigbot gateway --port 18789 --verbose

# Send a message
pigbot message send --to +1234567890 --message "Hello from Pigbot"

# Talk to the assistant (optionally deliver back to any connected channel: WhatsApp/Telegram/Slack/Discord/Google Chat/Signal/iMessage/BlueBubbles/Microsoft Teams/Matrix/Zalo/Zalo Personal)
pigbot agent --message "Ship checklist" --thinking high
```

## From source (development)

Prefer `pnpm` for builds from source. Bun is optional for running TypeScript directly.

```bash
git clone https://github.com/mochiyaki/pigbot.git
cd pigbot

pnpm install
pnpm ui:build # auto-installs UI deps on first run
pnpm build

pnpm pigbot onboard --install-daemon

# Dev loop (auto-reload on TS changes)
pnpm gateway:watch
```

## Technical Architecture Diagram

```
┌─────────────────────────────────────────────────────────┐
│                    User Devices                         │
│  (WhatsApp, Telegram, Slack, Discord, etc.)             │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│            Messaging Platform Adapters                  │
│  ┌──────────┬──────────┬──────────┬──────────┐          │
│  │ Telegram │  Slack   │ Discord  │  WhatsApp│  ...     │
│  └──────────┴──────────┴──────────┴──────────┘          │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│              Message Router & Queue                     │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│                 Core AI Engine                          │
│  ┌────────────────────────────────────────────┐         │
│  │  Context Manager  │  LLM Integration       │         │
│  │  Conversation DB  │  Model Router          │         │
│  └────────────────────────────────────────────┘         │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│              IDE Task Handlers                          │
│  ┌──────────┬──────────┬──────────┬──────────┐          │
│  │Code Gen  │ Debug    │ Refactor │  Test    │  ...     │
│  └──────────┴──────────┴──────────┴──────────┘          │
└──────────────────┬──────────────────────────────────────┘
                   │
                   ▼
┌─────────────────────────────────────────────────────────┐
│           Data Layer (PostgreSQL + Redis)               │
└─────────────────────────────────────────────────────────┘
```

## How it works

```
WhatsApp/Telegram/Slack/Discord/Signal/iMessage/GoogleChat/MicrosoftTeams...
               │
               ▼
┌───────────────────────────────┐
│            Gateway            │
│       (control plane)         │
│     ws://127.0.0.1:18789      │
└──────────────┬────────────────┘
               │
               ├─ Pi agent (RPC)
               ├─ CLI (pigbot …)
               ├─ WebChat UI
               ├─ macOS app
               └─ iOS / Android nodes
```

## Chat commands

Send these in WhatsApp/Telegram/Slack/Google Chat/Microsoft Teams/WebChat (group commands are owner-only):

- `/status` — compact session status (model + tokens, cost when available)
- `/new` or `/reset` — reset the session
- `/compact` — compact session context (summary)
- `/think <level>` — off|minimal|low|medium|high|xhigh (GPT-5.2 + Codex models only)
- `/verbose on|off`
- `/usage off|tokens|full` — per-response usage footer
- `/restart` — restart the gateway (owner-only in groups)
- `/activation mention|always` — group activation toggle (groups only)

## Apps (optional)

The Gateway alone delivers a great experience. All apps are optional and add extra features.

If you plan to build/run companion apps, follow the platform runbooks below.

### macOS (Pigbot.app) (optional)

- Menu bar control for the Gateway and health.
- Voice Wake + push-to-talk overlay.
- WebChat + debug tools.
- Remote gateway control over SSH.

Note: signed builds required for macOS permissions to stick across rebuilds (see `docs/mac/permissions.md`).

### iOS node (optional)

- Pairs as a node via the Bridge.
- Voice trigger forwarding + Canvas surface.
- Controlled via `pigbot nodes …`.

### Android node (optional)

- Pairs via the same Bridge + pairing flow as iOS.
- Exposes Canvas, Camera, and Screen capture commands.

## Configuration

Minimal `~/.pigbot/pigbot.json` (model + defaults):

```json5
{
  agent: {
    model: "anthropic/claude-opus-4-5"
  }
}
```

## Security model (important)

- **Default:** tools run on the host for the **main** session, so the agent has full access when it’s just you.
- **Group/channel safety:** set `agents.defaults.sandbox.mode: "non-main"` to run **non‑main sessions** (groups/channels) inside per‑session Docker sandboxes; bash then runs in Docker for those sessions.
- **Sandbox defaults:** allowlist `bash`, `process`, `read`, `write`, `edit`, `sessions_list`, `sessions_history`, `sessions_send`, `sessions_spawn`; denylist `browser`, `canvas`, `nodes`, `cron`, `discord`, `gateway`.

### WhatsApp

- Link the device: `pnpm pigbot channels login` (stores creds in `~/.pigbot/credentials`).
- Allowlist who can talk to the assistant via `channels.whatsapp.allowFrom`.
- If `channels.whatsapp.groups` is set, it becomes a group allowlist; include `"*"` to allow all.

### Telegram

- Set `TELEGRAM_BOT_TOKEN` or `channels.telegram.botToken` (env wins).
- Optional: set `channels.telegram.groups` (with `channels.telegram.groups."*".requireMention`); when set, it is a group allowlist (include `"*"` to allow all). Also `channels.telegram.allowFrom` or `channels.telegram.webhookUrl` as needed.

```json5
{
  channels: {
    telegram: {
      botToken: "123456:ABCDEF"
    }
  }
}
```

### Slack

- Set `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` (or `channels.slack.botToken` + `channels.slack.appToken`).

### Discord

- Set `DISCORD_BOT_TOKEN` or `channels.discord.token` (env wins).
- Optional: set `commands.native`, `commands.text`, or `commands.useAccessGroups`, plus `channels.discord.dm.allowFrom`, `channels.discord.guilds`, or `channels.discord.mediaMaxMb` as needed.

```json5
{
  channels: {
    discord: {
      token: "1234abcd"
    }
  }
}
```

### Signal

- Requires `signal-cli` and a `channels.signal` config section.

### iMessage

- macOS only; Messages must be signed in.
- If `channels.imessage.groups` is set, it becomes a group allowlist; include `"*"` to allow all.

### Microsoft Teams

- Configure a Teams app + Bot Framework, then add a `msteams` config section.
- Allowlist who can talk via `msteams.allowFrom`; group access via `msteams.groupAllowFrom` or `msteams.groupPolicy: "open"`.

Browser control (optional):

```json5
{
  browser: {
    enabled: true,
    controlUrl: "http://127.0.0.1:18791",
    color: "#FF4500"
  }
}
```

# Pigbot IDE extension (part 2)

A simple VS Code extension that manages Pigbot connection status via a status bar item.

*please go to /plugins/vscode/ directory for part 2 codebase development

## Features

- **Status Bar Item**: Shows connection status with three states:
  - `$(plug) Pigbot - Not connected (click to connect)`
  - `$(sync~spin) Connecting...` - Connection in progress
  - `$(check) Pigbot - Connected to Pigbot`

- **OS Detection**: Automatically detects the operating system and uses:
  - `pigbot status` on Windows (wsl)
  - `pigbot status` on other platforms

- **Auto-Connect**: Optional setting to automatically connect on startup (disabled by default)

## Usage

1. Click the Pigbot status bar item (bottom right) to connect
2. The extension will open a terminal and execute the `pigbot status` command
3. To enable auto-connect, go to Settings and enable `Pigbot: Auto Connect`

## Development

1. Install dependencies: `npm install`
2. Compile: `npm run compile`
3. Press F5 to launch the Extension Development Host

## Acknowledgment and Codebases

- core part codebase was forked from Clawdbot/Moltbot https://github.com/moltbot/moltbot (MIT License)
- vscode extension plugin codebase was forked from https://github.com/mochiyaki/clawdbot (MIT License)

## License

This project is licensed under the MIT License
