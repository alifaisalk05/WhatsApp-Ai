<div align="center">

# WA Agent

**A single-file WhatsApp AI agent with a neomorphic admin dashboard.**

Link your own number, plug in any OpenAI-compatible model, and let it reply like you: humanized, remembering, and always under your control.

![Node](https://img.shields.io/badge/node-%E2%89%A520-3c873a?logo=node.js&logoColor=white)
![Files](https://img.shields.io/badge/source-single%20file-0f8a66)
![UI](https://img.shields.io/badge/UI-neomorphic%20light%20%2B%20dark-6c7a96)
![License](https://img.shields.io/badge/license-MIT-blue)

</div>

> [!WARNING]
> WA Agent uses [Baileys](https://github.com/WhiskeySockets/Baileys), an **unofficial** WhatsApp Web client. This is against WhatsApp's Terms of Service, and the linked number **can be restricted or banned**. Keep volumes low, leave humanized delays on, and never cold-message strangers. Use at your own risk, preferably on a number you can afford to lose.

---

## Table of contents

- [Features](#features)
- [Quick start](#quick-start)
- [Configuration](#configuration)
- [Admin dashboard](#admin-dashboard)
- [AI providers](#ai-providers)
- [How replies work](#how-replies-work)
- [Agent tools](#agent-tools)
- [Owner control from WhatsApp](#owner-control-from-whatsapp)
- [Voice notes](#voice-notes)
- [Rules](#rules)
- [Data and backups](#data-and-backups)
- [Security](#security)
- [Running in production](#running-in-production)
- [Troubleshooting](#troubleshooting)
- [FAQ](#faq)
- [Contributing](#contributing)
- [License](#license)

---

## Features

**Messaging**
- Links your existing number as a *linked device* via **QR code or pairing code**. Your chats stay on your phone.
- Humanized replies: read receipts, typing indicator, randomized delays, reply splitting, quoting in groups.
- Message batching (debounce), so several quick messages get one coherent answer.
- Hourly and daily caps, quiet hours with an optional auto-message.

**Intelligence**
- **Multiple OpenAI-compatible providers** (OpenAI, OpenRouter, Groq, DeepSeek, Gemini, Ollama, LM Studio, or any custom endpoint) tried **top to bottom with automatic fallback**.
- **Voice-note transcription** and **image understanding** (vision).
- **Long-term memory**: per-contact facts plus automatic conversation summaries.
- **Tool calling**: remember facts, reminders, notify you, hand off to a human, search chat history, fetch web pages (optional).
- Optional **voice replies** using Gemini TTS, sent as WhatsApp voice notes.

**Control**
- **Modes**: allowlist, blocklist or everyone, with a per-chat override: *inherit / on / off / draft-approval*.
- **Groups**: off, only when mentioned or replied to, or all, plus trigger words.
- **Human takeover**: a chat pauses automatically when *you* reply manually.
- **Approval queue**: review, edit and approve drafts before they are sent.
- **Regex rules**: canned reply, ignore, hand off, or add an extra AI instruction.
- **Owner commands** from your own "Message yourself" chat (`/pause`, `/resume`, `/on`, `/off`, `/status`), or just talk to the control assistant.

**Dashboard**
- Neomorphic admin UI with a **light / dark theme toggle** (remembered per browser, follows your system by default).
- Overview, chats, drafts, agent, providers, rules, playground, live logs and settings.
- Live updates over Server-Sent Events, config export/import, responsive on mobile.

**Engineering**
- One file, three small dependencies, no build step.
- Config and state are plain JSON files on disk.
- Admin password hashed with scrypt; web fetching blocks private network addresses.

---

## Quick start

**Requirements:** Node.js **20+**. `ffmpeg` is only needed for voice replies.

```bash
git clone https://github.com/YOUR_USERNAME/wa-agent.git
cd wa-agent
npm install
npm start
```

Or from scratch with just the file:

```bash
mkdir wa-agent && cd wa-agent
cp /path/to/wa-agent.mjs .
npm init -y
npm i @whiskeysockets/baileys qrcode pino
node wa-agent.mjs
```

Then:

1. Open **http://127.0.0.1:3000**.
2. Sign in with the **admin password printed in the console** on first run (or set `ADMIN_PASSWORD` before the first start). Change it in **Settings**.
3. On **Overview**, link WhatsApp: *WhatsApp → Settings → Linked devices → Link a device*, then scan the QR code (or request a pairing code with your phone number).
4. Open **Providers**, pick a preset (for example OpenAI or Groq), paste your API key, and press **Test**, then **Save changes**.
5. Open **Agent**, choose who it may reply to (start with an **allowlist** containing only your own test contact), then turn the agent **on** from Overview.

> [!TIP]
> Use the **Playground** tab to test your prompt and providers. Nothing is sent to WhatsApp and nothing is saved there.

---

## Configuration

### Environment variables

| Variable | Default | Description |
| --- | --- | --- |
| `PORT` | `3000` | Port for the admin panel |
| `HOST` | `127.0.0.1` | Interface to bind. Keep it on localhost unless you add HTTPS and a reverse proxy |
| `WA_DATA` | `./data` | Directory for the WhatsApp session, `config.json` and `state.json` |
| `ADMIN_PASSWORD` | random | Initial admin password, used only on the very first start |
| `FFMPEG` | `ffmpeg` | Path to the ffmpeg binary (voice replies) |

```bash
PORT=8080 WA_DATA=/var/lib/wa-agent ADMIN_PASSWORD='change-me-please' node wa-agent.mjs
```

### Agent settings (Agent tab)

| Setting | Default | What it does |
| --- | --- | --- |
| `enabled` | off | Master switch for auto-replies |
| `mode` | `allowlist` | `allowlist`, `blocklist` or `all` |
| `allow` / `block` | empty | Numbers or JIDs for the mode above |
| `groups` | `mention` | `off`, `mention` (mentioned or replied-to) or `all` |
| `triggerWords` | empty | Extra words that wake the agent in groups |
| `ownerName` | `the owner` | Name the agent speaks as |
| `systemPrompt` | built-in | Persona and rules. `{owner}` is replaced with `ownerName` |
| `historyLimit` | `24` | Messages of context sent to the model |
| `debounceMs` | `3500` | Wait for more messages before replying |
| `ignoreOlderSec` | `180` | Ignore messages older than this (for example after downtime) |
| `approval` | off | Queue every reply as a draft instead of sending |
| `humanTakeoverMin` | `45` | Pause a chat this long after you reply manually |
| `readReceipts` / `typing` | on | Humanized behavior |
| `minDelayMs` / `maxDelayMs` | `1500` / `9000` | Random delay before replying |
| `typingCps` | `9` | Simulated typing speed (characters per second) |
| `splitReplies` | on | Split long replies into several messages |
| `quoteInGroups` | on | Quote the message being answered in groups |
| `maxPerHour` / `dailyCap` | `20` / `300` | Hard sending limits |
| `maxReplyChars` | `1200` | Maximum reply length |
| `quiet` | off | Quiet hours (`23:00` to `07:00`) with an optional auto-message |
| `transcribe` / `vision` | on | Voice-note transcription and image understanding |
| `tools` | on | Allow tool calling |
| `allowWeb` | off | Allow the `fetch_url` tool |
| `autoSummary` | on | Summarize long chats to keep memory compact |
| `notifyOwner` | on | Let the agent ping you privately |
| `ownerCommands` | on | Enable control from your own chat |

---

## Admin dashboard

| Tab | Purpose |
| --- | --- |
| **Overview** | Connection status, QR / pairing code, agent on/off, pause buttons, activity stats, reminders |
| **Chats** | Search chats, read history, set a per-chat mode, pause or clear a chat, manage remembered facts, send a message |
| **Drafts** | Edit and approve (or discard) replies waiting for approval |
| **Agent** | Mode, groups, persona, delays, limits, quiet hours, feature toggles |
| **Providers** | Add, order, test and remove AI endpoints |
| **Rules** | Regex rules that run before the AI |
| **Playground** | Dry-run your prompt against a blank chat or an existing history |
| **Logs** | Live log stream |
| **Settings** | Time zone, admin password, config export/import, reconnect, unlink device |

Use the **☾ Dark mode / ☀ Light mode** button at the bottom of the sidebar (or top-right on the login screen) to switch themes.

---

## AI providers

Any endpoint that speaks the OpenAI chat-completions API works. Providers are tried **in order**; if one fails, the next takes over. Presets are included for:

| Preset | Base URL |
| --- | --- |
| OpenAI | `https://api.openai.com/v1` |
| OpenRouter | `https://openrouter.ai/api/v1` |
| Groq | `https://api.groq.com/openai/v1` |
| DeepSeek | `https://api.deepseek.com/v1` |
| Gemini | `https://generativelanguage.googleapis.com/v1beta/openai` |
| Ollama | `http://localhost:11434/v1` |
| LM Studio | `http://localhost:1234/v1` |
| Custom | any URL ending in `/v1` |

Per provider you can set the chat model, an optional **vision model**, an optional **speech-to-text model** (for voice-note transcription, for example `whisper-1`), temperature, max tokens, custom headers, and whether the model supports tool calling.

---

## How replies work

```
incoming message
   │
   ├─ gate: agent on? mode / allow / block? group rules? paused? quiet hours? caps?
   ├─ debounce: wait for follow-up messages, batch them
   ├─ media: transcribe voice notes, describe images
   ├─ rules: regex rules (canned reply / ignore / handoff / extra instruction)
   ├─ AI: persona + facts + summary + recent history, with tools, across providers in order
   └─ deliver: send now, or queue as a draft for approval
        (read receipt → typing indicator → human-like delay → split → send)
```

If you reply manually in a chat, the agent steps back for `humanTakeoverMin` minutes.

---

## Agent tools

Available to the agent when `tools` is on:

| Tool | Description |
| --- | --- |
| `remember_fact` | Save a durable fact about the contact |
| `set_reminder` | Schedule a reminder for you or the chat |
| `notify_owner` | Send you a private note about the chat |
| `handoff_to_human` | Stop replying and alert you |
| `search_history` | Search earlier messages in the chat |
| `get_time` | Current date and time |
| `fetch_url` | Fetch a public web page (only when `allowWeb` is on) |

Extra tools for the **owner control assistant** only: `send_message` (to a group, contact name, number or JID), `list_chats`, and `set_agent`.

---

## Owner control from WhatsApp

Open your own "Message yourself" chat:

| Command | Effect |
| --- | --- |
| `/status` | Agent state, replies sent, daily usage, pending drafts |
| `/on` / `/off` | Enable or disable the agent |
| `/pause [min]` | Pause for N minutes (default 60) |
| `/resume` | Resume immediately |

Anything else is handled by a private control assistant, so you can say things like *"tell the Class 10th group the test is moved to Friday"* and it will use `send_message`.

---

## Voice notes

- **Incoming:** voice notes are transcribed through a provider that has an `sttModel`.
- **Outgoing:** enable **Voice replies (Gemini TTS)** in the Agent tab, add a Gemini API key, and install **ffmpeg** (or set `FFMPEG=/path/to/ffmpeg`). The agent only answers with audio when a contact clearly asks for it (for example "send a voice note").

---

## Rules

Rules are checked **in order, before the AI runs**. Patterns are regular expressions with flags (default `i`), scoped to all chats, and can:

- **reply** with a canned message,
- **ignore** the message,
- **hand off** to you,
- or add an **extra instruction** for the AI.

---

## Data and backups

Everything lives in `WA_DATA` (default `./data`):

```
data/
├── auth/          WhatsApp session credentials (treat like a password)
├── config.json    Settings, providers (with API keys), rules, password hash
└── state.json     Chats, history, facts, reminders, drafts, stats
```

Use **Settings → Export config** for a backup and **Import config** to restore. Exports **include API keys**, so keep the file private. `data/` is in `.gitignore`; never commit it.

---

## Security

- The panel binds to **127.0.0.1** by default. To reach it remotely, put it behind **HTTPS** (a reverse proxy such as Caddy or nginx) and keep a strong admin password; do not expose it directly to the internet.
- The admin password is stored as a salted **scrypt** hash. Change the first-run password immediately.
- `fetch_url` is **off by default** and blocks private and loopback addresses and redirects.
- API keys are stored in plain text in `config.json`. Restrict file permissions (`chmod 600`) and use keys with spending limits.
- Start with an **allowlist** and the **approval** queue until you trust the behavior.

---

## Running in production

**systemd**

```ini
[Unit]
Description=WA Agent
After=network-online.target

[Service]
WorkingDirectory=/opt/wa-agent
Environment=WA_DATA=/var/lib/wa-agent
ExecStart=/usr/bin/node wa-agent.mjs
Restart=always
User=waagent

[Install]
WantedBy=multi-user.target
```

**pm2**

```bash
npm i -g pm2
pm2 start wa-agent.mjs --name wa-agent
pm2 save
```

**Docker (example)**

```dockerfile
FROM node:20-slim
RUN apt-get update && apt-get install -y --no-install-recommends ffmpeg && rm -rf /var/lib/apt/lists/*
WORKDIR /app
COPY package.json wa-agent.mjs ./
RUN npm install --omit=dev
ENV HOST=0.0.0.0 WA_DATA=/data
VOLUME /data
EXPOSE 3000
CMD ["node", "wa-agent.mjs"]
```

```bash
docker build -t wa-agent .
docker run -d --name wa-agent -p 127.0.0.1:3000:3000 -v wa-data:/data wa-agent
```

---

## Troubleshooting

| Problem | Fix |
| --- | --- |
| Forgot the admin password | Stop the app, delete the `admin` block in `data/config.json`, and start with `ADMIN_PASSWORD=...` |
| No QR code appears | Wait a few seconds, press **Reconnect** in Settings; make sure the server has internet access |
| Logged out / session invalid | Settings → **Unlink device**, then scan a new QR code |
| "No AI provider yet" | Add one in **Providers**, test it, then **Save changes** |
| Agent never replies | Check the agent is **on**, the sender is allowed by the mode, the chat isn't paused, and you're within hourly/daily caps and outside quiet hours |
| Replies are skipped after downtime | Messages older than `ignoreOlderSec` are ignored on purpose |
| Voice replies fail | Install `ffmpeg` or set `FFMPEG`; add a Gemini key in the voice settings |
| Port already in use | Set `PORT=3001` |

---

## FAQ

**Does it use the official WhatsApp Business API?** No. It uses an unofficial web client, which is why the ban risk exists.

**Does it read my other chats?** It sees messages for the account it is linked to, but only replies according to your mode, allowlist and rules.

**Can I use local models?** Yes. Use the Ollama or LM Studio preset and keep everything on your machine.

**Is my data sent anywhere?** Message content is sent to the AI providers you configure, and nowhere else.

---

## Contributing

Issues and pull requests are welcome. The project is intentionally a **single file with minimal dependencies**, so please keep changes small and self-contained. Run `npm run check` before submitting.

---

## License

[MIT](LICENSE) © YOUR NAME

> WA Agent is an independent project and is not affiliated with, endorsed by, or connected to WhatsApp or Meta.
