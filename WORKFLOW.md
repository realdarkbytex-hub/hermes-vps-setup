# Noya Owns Workflow — for Hermes Agent on the VPS
Give this file to Hermes Agent. It must follow it exactly.

## Channel
- Channel: Noya Owns — family-betrayal drama Shorts, USA audience
- Daily: 2 Shorts, scheduled 8 PM and 10 PM IST, every day
- Format: dialogue only, characters talk to each other, NO narrator voiceover
- NO captions at all. Normal speed 1.0x, never speed up. 1080p vertical.

## Topic rule — MOST IMPORTANT
- Do NOT invent topics or stories yourself.
- Topics/scripts come ONLY from: the user, the pinned Gemini "Noya Owns" chat, or Claude web ("NOYA OWNS Shorts strategy setup" chat, say "next").
- Niche: betrayal stories broadly, not just Evelyn. New protagonists allowed.
- FLOW S formula: serialized 2-3 parts, pure dialogue, short lines 5-10 words, insult in first line (hook in first 3 sec), escalation, status-reveal twist in very last line.
- Check channel catalogue + workspace for duplicates before using a topic.

## Script first
- Show the script first and explain it in Hinglish before generating any video.
- Same character, ONE consistent appearance across all scenes in a story — identical detailed description in every prompt.

## Video generation — Gemini app (Flow is OUT)
- Scripts from Claude web, video from Gemini app.
- 1 account = 3 clips (one Short = 3 scenes from one account). Switch account for second Short.
- Triage: naitikkashyap007 first, noyaowns second, darkbyte last.
- Gemini "continue" prompts are cumulative — the Scene 3 clip alone is the full ~30 sec story. Build from Scene 3 alone, do NOT join scenes.
- Do NOT name real public figures in visual prompts — use generic descriptors.
- Never retry a refused prompt. State quota/credit cost BEFORE any re-generation.
- ALWAYS check clip AUDIO (transcribe) + frames before assembling, not just frames. Clips can contain baked-in dialogue — layering TTS on top = double voice.

## Assembly + clean
- Scene duration = dialogue duration. 1080p, upscale 720p to 1080p with ffmpeg lanczos if needed.
- Remove sparkle watermark with geminiwatermarkremover.io/video — FREE local flow ONLY ("Start local cleanup", no account, never paid "Remove with AI"). ffmpeg delogo is fallback only.
- Thumbnail: protagonist's emotional face at betrayal moment, text like "MY TROPHIES?!" / "MY TRUCK?!" — keep verified past style.
- Playlist: Money Betrayal / Humiliated in Public / They Took Her House — pick by story.
- Duplicate-check before upload. Never mark Public when Scheduled was said. Report video ID + schedule time + privacy + thumbnail after upload, verified in Studio.

## Limits on this VPS
- VPS: Oracle Always Free A1 Flex, 1 OCPU / 6 GB RAM, Ubuntu 24.04 aarch64. No GPU. Local heavy work is slow — keep it to control, files, and light tasks.
- This Hermes Agent runs gemini-3.1-flash-lite (primary) + NVIDIA NIM fallback — fast and light, good for control / small tasks. For heavy script/code, use a Pro/Claude model in Antigravity instead.
- The VPS runs 24/7 — the laptop does NOT need to be on. But Gemini clip generation still needs the user's own Google sign-in on the laptop; Hermes cannot sign in to Gemini/Claude and cannot generate Gemini clips.
- Never share Immich URL/API key, YouTube tokens, or passwords. Never retry a rejected saved Google password, never use recovery paths — user signs in himself.
