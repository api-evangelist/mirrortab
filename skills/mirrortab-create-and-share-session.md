---
name: Create and share a MirrorTab session
description: >-
  Create an isolated MirrorTab browser session for a protected web app, hand the
  join code to a user, and clean the session up afterward.
api: openapi/mirrortab-api-openapi.yml
operations:
- createSession
- listSessions
- removeSession
---

# Create and share a MirrorTab session

Use this skill to spin up an isolated MirrorTab browser session, give a user a
way to join it, and remove it when done. All calls are `POST` to
`https://api.mirrortab.com` with a JSON body that always includes your
`api_key` (issued at https://mirrortab.com/API — free accounts have no API
access).

## Steps

1. **Create the session** — call `createSession` (`POST /new_session`) with
   `api_key` and, optionally, `session_name`, `permissions`
   (`open` | `gocode` | `account`, default `gocode`), `duration_min`
   (default 45), and `urls` to pre-open. The `201` response returns
   `session_id`, `go_url`, `gocode`, and `kill_time_UTC`.
2. **Share access** — give the user the `go_url` (and the `gocode` when
   `permissions=gocode`). With `permissions=open` anyone with the URL can join;
   with `account` every joiner must have a MirrorTab account.
3. **Track active sessions** — call `listSessions` (`POST /list_sessions`) with
   `api_key` to get the map of active sessions keyed by `session_id`. A `204`
   means the key is valid but has no active sessions.
4. **Remove the session** — when finished (or before `kill_time_UTC`), call
   `removeSession` (`POST /remove_session`) with `api_key` and `session_id`.
   A `200` `{ "removed": "true" }` confirms teardown.

## Rules

- **Auth:** send `api_key` in the body of every request. A missing key returns
  `400`; an invalid key returns `401` (see `errors/mirrortab-problem-types.yml`).
- **No idempotency:** each `createSession` call makes a new session — do not
  retry blindly; on an ambiguous failure, call `listSessions` to reconcile
  before recreating (see `conventions/mirrortab-conventions.yml`).
- **Session limits:** concurrent session count and `duration_min` are capped by
  account tier; sessions auto-remove at `kill_time_UTC`.
