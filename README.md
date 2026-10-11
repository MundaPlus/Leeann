# Leeann

**A private assistant for one person: an AI agent on a home server, a desktop tray app and an Android app that share one conversation.** Leeann runs on a self-hosted agent with local language models. She reads mail, task boards, finances, release activity and the state of the homelab, writes a morning brief, and answers by voice. Nothing leaves the home network: the phone reaches her over Tailscale, and speech recognition and speech synthesis run on the phone itself.

![Phone: the home screen with Leeann's presence orb, the last question and answer, and the prompt](screenshots/phone-home.jpg)

> Screenshots show a demo gateway with invented tasks, mail, hosts, money and conversations.

| Chat | Dashboard | Homelab | Host |
|---|---|---|---|
| ![Shared chat with markdown answers](screenshots/phone-chat.jpg) | ![Dashboard cards](screenshots/phone-dashboard.jpg) | ![Homelab hosts by group](screenshots/phone-homelab.jpg) | ![Host sheet with failed units and services](screenshots/phone-host.jpg) |

![Desktop: dashboard and chat](screenshots/desktop-dashboard.jpg)

| Desktop homelab | Desktop chat |
|---|---|
| ![Host cards with a detail panel](screenshots/desktop-homelab.jpg) | ![Chat with markdown answers](screenshots/desktop-chat.jpg) |

## Features

- **One conversation, every device.** The phone and the desktop share a seven-day chat history. A question asked on the phone shows on the desktop within a fraction of a second, and the reverse. Each device speaks only the answers to its own questions.
- **Hands-free on the phone.** Tap the orb, press the play button on a Bluetooth headset, use the Quick Settings tile or the home screen shortcut, or say "Hey Leeann". A beep confirms, silence ends the turn, and the answer is spoken. The headset microphone is used when one is connected.
- **On-device voice.** Kokoro text-to-speech and streaming speech recognition run on the phone with sherpa-onnx; the wake word uses keyword spotting. Long answers are spoken sentence by sentence while the next part is generated. A server voice is the fallback.
- **Commands without a round trip.** "Brief", "next task", "homelab status", "mail", "stop", "repeat", "louder" and "softer" are answered on the phone from live data; everything else goes to Leeann.
- **Dashboard.** The day with weather and deadlines, the morning brief, the active task and what comes next, mail, company task boards and releases, open features and bugs of a client platform, net worth and a music channel, the same on both apps.
- **Homelab view.** Hosts grouped like a self-hosted homelab dashboard, with load, disk, uptime and failed systemd units, plus the services on each host with a live up/down check.
- **Notifications that keep secrets.** New briefs, focus pokes, mail and answers arrive as notifications with generic text only ("New brief from Leeann"); the content stays in the app.
- **Day and night volume.** Separate volumes for day and night, on both devices.
- **Offline behaviour.** The desktop falls back to direct access when the gateway is down; the phone shows a clear offline banner and keeps the last data.

## Tech stack

Hermes Agent (Nous Research) · local LLMs · Python · FastAPI · SQLite · Server-Sent Events · PyQt6 · Flutter · sherpa-onnx (Kokoro TTS, streaming Zipformer ASR, keyword spotting) · audio_service · WorkManager · Docker · Tailscale

## How it works

```
Android app (Flutter) ──┐                          ┌── Hermes agent (chat, local LLMs)
  on-device TTS + ASR   ├── Tailscale / LAN ── Gateway ──┼── status collector (mail, boards, finance, …)
Desktop tray (PyQt6) ───┘    device tokens    (FastAPI)  ├── homelab layout + live host checks
                                                         ├── cron output (morning brief, focus pokes)
                                                         └── SQLite: shared chat history
```

A small gateway sits between the apps and the agent. It holds the agent's API key, so the apps only ever hold a per-device token. It caches the dashboard data, merges the homelab layout with live host checks, streams answers as Server-Sent Events and pushes new messages, briefs, pokes and mail counts to every open app.

## Design principles

- **The home network is the cloud.** Language models, data and history stay on the home server; speech stays on the phone.
- **Least privilege.** The apps never see the agent's key; device tokens are stored as hashes and can be revoked one by one. The apps cannot change anything on the servers.
- **Quiet logs.** The gateway logs paths, status codes and timings, never message text or tokens.
- **One microphone user at a time.** The wake word, dictation and talk turns hand the microphone to each other, so the headset button stays a media button.

## Availability

Copyright (c) 2026 MUNDA PLUS d.o.o. All rights reserved. See [LICENSE](LICENSE).

Licensing enquiries: https://www.munda.si/#contact
