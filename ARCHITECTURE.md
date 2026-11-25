# Architecture — Hermes on Oracle Always Free VPS

## The setup

One Oracle Always Free VM runs two things, 24/7, with the laptop off:

```
                        Telegram
                           |
              +------------+------------+
              |                         |
     @noyastick_bot            @videoo_waleagyebot
     (Hermes — you only)       (Video bot — public)
              |                         |
              v                         v
   Hermes Gateway (systemd)    videobot service (systemd)
              |                         |
              v                         v
   Hermes Agent v0.23.4       bot.py + yt-dlp + ffmpeg
   Ubuntu 24.04 aarch64       /opt/videobot
   1 OCPU / 6 GB RAM
   Oracle A1 Flex, Mumbai
```

## Model routing

Hermes on the VPS does NOT use OmniRoute (that only exists on the laptop). It uses direct providers:

```
User message (Telegram)
        |
        v
Hermes Agent
        |
        +--> Gemini gemini-3.1-flash-lite  (primary, direct API)
        |
        +--> NVIDIA NIM nemotron-3.5-super-vl-120b-a12b  (fallback)
             reasoning effort: high  (never medium — NVIDIA rejects it, 400)
```

Why direct Gemini on the VPS: OmniRoute is a local router (`localhost:20128`) that isn't installed on the VM, and the VM doesn't need it — two providers, one primary + one fallback, is enough. Copying the laptop's OmniRoute config verbatim would point the VPS at a `localhost:20128` that doesn't exist.

## Why the laptop is still (sometimes) needed

| Task | Where | Why |
|---|---|---|
| Hermes control, files, Noya workflow | VPS | 24/7, no laptop needed |
| Video-downloader bot | VPS | 24/7, public |
| Google / Gemini / Claude sign-in | Laptop | Only the user signs in; agents never get passwords |
| Gemini clip generation (video) | Laptop | Needs the user's own signed-in Gemini app |
| Screen work (VNC, GRUB, GUI) | Laptop | Interactive console |

## Failure modes we've hit (and what they mean)

### 1. Polling conflict — two gateways, one token
Telegram delivers each update to exactly one long-poller. If the laptop gateway and the VPS gateway both poll the same bot token, updates get split between them. Symptom: the bot replies sometimes, or replies on one account and stays silent on another. Fix: stop the laptop gateway before starting the VPS one. Verify by asking Hermes which machine it's on.

### 2. Private build still polling — the silent second ID
The old private bot only answers its allowed ID. If it's still running (e.g. laptop auto-start after a reboot), it steals the poll and the public VM bot never sees the message. Symptom: main ID works, second ID gets nothing — even after the VM is fixed to public mode. Fix: stop/disable the laptop bot service (or turn the laptop off), then the VM bot receives everything.

### 3. 401 Unauthorized loop — corrupted token
If a script rewrites `.env` and mangles the `BOT_TOKEN` line, the bot can't authenticate at all and loops on `HTTP Error 401: Unauthorized`. Symptom: no reply to ANY ID, logs full of 401s. Fix: put the correct token back (from BotFather → API Token), restart the service. Check the token first after any `.env` edit — don't debug the network.

### 4. Out of host capacity — Always Free Ampere
`VM.Standard.A1.Flex` in `ap-mumbai-1` regularly returns "Out of host capacity" (HTTP 500). It's not your config — it's scarcity. Fix: retry the same CLI launch every 15 minutes. Never terminate a running VM to retry; you may not get one back.

### 5. No SSH key on a launched instance — the GRUB recovery
Oracle blocks adding `ssh_authorized_keys` AND `user_data` to a launched instance, and the Run Command agent (`ocarun`, uid 999) has no sudo and can't write to `/home/ubuntu`. If you launched without a key, the only recovery is GRUB `init=/bin/bash` over VNC + serial console (see `SETUP.md` Part 5). The lesson is one line: always launch with `--ssh-authorized-keys-file`.

## What runs where (final state)

| Service | Host | How it starts | Needs laptop? |
|---|---|---|---|
| Hermes gateway | VPS | systemd | No |
| videobot | VPS | systemd (`videobot.service`, Restart=always) | No |
| Noya Owns pipeline | VPS (control) + laptop (clips) | Hermes skill + user sign-in | Clips only |
| Laptop Hermes gateway | Stopped | — | — |
