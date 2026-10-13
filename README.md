# Voicemail-build

A minimal, practical blueprint for building a **custom visual voicemail app** for a Galaxy S25 Ultra (One UI 9.0).

## Goal
Build a self-hosted visual voicemail solution with:
- voicemail inbox UI
- playback + delete + mark-heard controls
- optional voicemail transcription
- ownership of your voicemail data

## One UI 9.0 / Android constraints to account for
On modern Android, voicemail data access is permission-gated. Your app should be designed to run in one of these supported modes:

1. **Default dialer mode** (recommended for full device integration)
   - request dialer role via `RoleManager.ROLE_DIALER`
   - use `VoicemailContract` APIs where allowed
2. **Companion mode** (no dialer role)
   - maintain your own inbox from a backend feed
   - do not rely on privileged carrier voicemail provider writes

This repository starts with the implementation plan for mode #2 and a path to mode #1.

## MVP architecture

### 1) Android client (Kotlin + Jetpack Compose)
- Auth to your backend
- Inbox list (caller, timestamp, duration, heard/unheard)
- Voicemail detail screen with streaming playback
- Actions: mark heard, delete, download
- Background sync (WorkManager)

### 2) Backend API (self-hosted)
- REST endpoints:
  - `POST /api/v1/voicemails` (ingest metadata + audio)
  - `GET /api/v1/voicemails`
  - `GET /api/v1/voicemails/{id}`
  - `PATCH /api/v1/voicemails/{id}` (heard/starred)
  - `DELETE /api/v1/voicemails/{id}`
- Media storage (S3-compatible or local object store)
- SQLite/PostgreSQL for metadata
- Optional async transcription worker

### 3) Data model
Each voicemail record should contain:
- `id`
- `caller_number`
- `caller_name` (nullable)
- `received_at`
- `duration_seconds`
- `audio_url`
- `transcript` (nullable)
- `heard` (boolean)
- `deleted` (boolean)

## Security baseline
- Require authenticated API access (JWT or session token)
- Encrypt transport with HTTPS only
- Encrypt stored voicemail audio at rest
- Do not log voicemail audio URLs or tokens in plaintext

## Suggested milestone plan
1. Build backend CRUD + media upload
2. Build Android inbox + playback from backend
3. Add sync + offline cache
4. Add transcription pipeline
5. Add default dialer integration path for deeper device-level behavior

## Manual verification checklist (MVP)
- Create a voicemail via API and confirm it appears in app inbox
- Stream playback on device without crash
- Mark heard/unheard updates persist after refresh
- Delete action removes voicemail from active inbox
- Auth failure returns user to sign-in cleanly

## Current repository scope
This repository currently defines the concrete MVP and implementation path required to build the custom voicemail app. Next code additions should follow the architecture above.
