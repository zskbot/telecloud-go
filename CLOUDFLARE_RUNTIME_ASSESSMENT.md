# Cloudflare and media-runtime assessment

## Decision

This repository is **not deployable as a Cloudflare Worker**. It is a long-running Go/Gin application with Telegram client integration, local filesystem/session state, upload and download pipelines, FFmpeg/yt-dlp integration, and SQLite/MySQL/PostgreSQL options. A Worker is not a replacement for this process runtime.

Recommended split:
- Persistent Go container/runtime: Telecloud application, Telegram client, media jobs, FFmpeg/yt-dlp, session and durable file storage.
- Cloudflare Worker: optional HTTP edge gateway only, with bounded read-only routes until the backend security review is complete.
- External persistent database and object storage: configured independently for an isolated Preview; never bind Production data to Preview.

## Security gates before a public Preview

1. Supply Telegram API ID/hash and bot tokens only through the runtime secret manager; never commit them or paste them into source files.
2. Inspect `utils.LoadMasterKey()` and all session/cookie handling before selecting a storage volume or backup strategy.
3. Keep `.env`, `session.json`, database files, cookies, temporary media, and generated artifacts out of Git. The current `.gitignore` already excludes `.env`, `session.json`, and local database files.
4. Add authentication, authorization, upload limits, MIME/content validation, rate limits, and safe file-serving headers before exposing upload or media endpoints.
5. Verify rights and provenance for every Telegram channel, media source, uploaded file, and redistributed item. The GPL-3.0 application license does not grant rights to third-party media.
6. Test Go build, unit tests, container startup, health checks, upload/download bounds, storage persistence, and shutdown behavior on a non-production Preview.

## Cloudflare Worker gateway boundary

A Worker may later forward only explicitly allowlisted API routes to a separately deployed Preview origin. It must not:
- run the Go server or FFmpeg/yt-dlp;
- hold Telegram session state or master keys;
- store media or database files in ephemeral Worker memory;
- expose arbitrary URL proxying or user-controlled upstream destinations;
- activate third-party media providers before rights review.

## Current state

- This PR adds documentation only.
- No runtime, database, object storage, Worker binding, DNS, production service, or media source is configured.
- No deployment is claimed. The next safe step is a persistent-host Preview with isolated secrets and storage, followed by security and licensing checks.
