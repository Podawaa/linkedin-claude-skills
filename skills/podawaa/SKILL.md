---
name: podawaa
description: |
  Use the Podawaa Public API to publish LinkedIn posts on behalf of the user,
  with optional first comment, lead magnet comments, PDF carousel, and scheduled
  publishing. Private Accelerate+ API.
  TRIGGER when: the user asks to "post on LinkedIn", "schedule a LinkedIn post",
  "publish a LinkedIn post via Podawaa", or to "draft and schedule" content.
---

# Podawaa Public API

You can publish LinkedIn posts through a user's Podawaa account via the REST API.

## Authentication

All requests require a Bearer token. The user generates a key on
https://app.podawaa.com/settings/api-keys and pastes it into the user's
environment as `PODAWAA_API_KEY`. The key is shown ONCE; never ask the user to
re-paste a key you already received in this session.

```http
Authorization: Bearer pwa_live_<32-hex>
```

The API is available on **Private Accelerate** and **Private Dominate** plans. If
the user gets `401` or `PLAN_NO_LONGER_ELIGIBLE`, stop and ask them to upgrade
or restore their subscription — do not loop retries.

Base URL: `https://api-podawaa.com/api/v1`

## Discover the user's channels

Before publishing, you need the `channelId` of the LinkedIn account you want
to post from.

```bash
curl -s https://api-podawaa.com/api/v1/channels \
  -H "Authorization: Bearer $PODAWAA_API_KEY"
```

Response:
```json
{
  "data": [
    {
      "id": "550e8400-...",
      "platform": "linkedin",
      "accountType": "personal",
      "name": "Jean Dupont",
      "publicIdentifier": "jean-dupont"
    }
  ],
  "total": 1
}
```

If `total === 0`, ask the user to connect a LinkedIn channel in their dashboard.

## Publish a post

```bash
curl -s -X POST https://api-podawaa.com/api/v1/posts \
  -H "Authorization: Bearer $PODAWAA_API_KEY" \
  -H "Content-Type: application/json" \
  -H "X-Idempotency-Key: $(uuidgen)" \
  -d '{
    "channelId": "550e8400-...",
    "content": "Your LinkedIn post text. Max 3000 characters.",
    "firstComment": "Optional first comment. Max 1250 characters.",
    "leadMagnet": {
      "enabled": true,
      "comment": "DM me DEALS and I'll send you the resource.",
      "copies": 20
    },
    "carouselPdf": {
      "url": "https://example.com/my-carousel.pdf",
      "title": "My carousel title"
    },
    "scheduledAt": "2026-06-01T10:00:00Z",
    "boost": {
      "boostLevel": 2,
      "targetReactions": 50,
      "targetComments": 10
    }
  }'
```

### Field rules - enforce client-side before calling

- `channelId` (required) - must belong to the user's organisation, else 403.
- `content` (required) - 1 to **3000** characters (LinkedIn limit).
- `firstComment` (optional) - 1 to **1250** characters (LinkedIn comment limit).
- `leadMagnet` (optional) - use when the user asks for Lead Magnet mode: the
  same boost comment repeated N times. This mirrors the dashboard Engagement
  Bar and sets `isLeadMagnet=true` server-side so duplicate detection is
  intentionally bypassed for those copies.
  - `enabled`: boolean.
  - `comment`: required when enabled, max **500** characters.
  - `copies`: optional, **2 to 50**, defaults to 5. If `boost.targetComments`
    is also sent, it must equal `copies`.
- `carouselPdf` (optional) - attach a PDF carousel/document to the LinkedIn
  post.
  - `url`: required HTTPS URL to a public PDF file, fetchable by Podawaa.
  - `title`: optional, max **120** characters.
  - PDF must be **10 MB or smaller**. LinkedIn renders it as a native document
    carousel.
  - The API does **not** accept binary upload, multipart forms, base64, local
    file paths, or attached Claude files. Send only a direct public HTTPS URL.
- `scheduledAt` (optional, ISO8601 UTC) - if provided, must be at least **120 s**
  in the future and at most **30 days** out. **If omitted, the server publishes
  as soon as possible** (defaults to `now + 60 s` so the publish cron can pick it
  up). Use this for "post now"; only set `scheduledAt` when the user wants a
  specific time.
- `boost` (optional) - engagement configuration fired server-side once the
  post is live. Fields:
  - `boostLevel`: `1`, `2`, or `3`. Required if `boost` is provided.
  - `targetReactions`: 1 to 250.
  - `targetComments`: 0 to 50. Comments are AI-generated
    server-side based on the post content.
  - `audienceCategories`: optional array of category IDs (`XX-YY`). If omitted,
    the audience is selected by AI.
- `X-Idempotency-Key` (recommended header) - any UUID. Same key + same body
  within 24 h replays the original response. Same key + different body returns
  `409 IDEMPOTENCY_KEY_REUSED` (Stripe-style contract).
- Never send `firstCommentDelaySeconds` - the server controls the
  30-90 s anti-bot jitter automatically and will reject the field with 400.

### PDF carousel workaround for Claude

Claude cannot upload a PDF file to Podawaa through this API. To publish a PDF
carousel, use this workflow:

1. If the user gives you an existing public PDF URL, validate that it is a
   direct `https://...pdf`-style URL that opens without authentication and then
   send it as `carouselPdf.url`.
2. If the user gives you a local file, Claude attachment, Google Drive preview
   link, Dropbox preview link, Notion attachment, or any private URL, explain
   that Podawaa needs a direct public HTTPS URL first. Ask the user to upload
   the PDF to a public file host/CDN/bucket or provide a direct-download URL.
3. Do not invent a URL, do not send the file content in JSON, and do not use
   multipart upload. Podawaa will fetch the URL server-side, validate the PDF,
   store it, and attach it to the LinkedIn post.

Good:
```json
{
  "carouselPdf": {
    "url": "https://cdn.example.com/carousels/deals-playbook.pdf",
    "title": "Deals playbook"
  }
}
```

Bad:
```json
{
  "carouselPdf": {
    "file": "/Users/me/Desktop/deals-playbook.pdf",
    "base64": "JVBERi0x..."
  }
}
```

### Successful response (201)

```json
{
  "id": "550e8400-...",
  "channelId": "550e8400-...",
  "status": "scheduled",
  "scheduledAt": "2026-06-01T10:00:00Z",
  "hasFirstComment": true,
  "boost": {
    "scheduled": true,
    "boostLevel": 2,
    "targetReactions": 50,
    "targetComments": 10
  },
  "createdAt": "2026-05-29T12:00:00Z"
}
```

`boost` is `null` when no boost was requested. Store the returned `id`; you'll
use it to poll for status.

### Errors you should handle

| Status | Code | Action |
|---|---|---|
| 400 | `CONTENT_TOO_LONG` / `FIRST_COMMENT_TOO_LONG` / `SCHEDULED_AT_TOO_SOON` / `SCHEDULED_AT_TOO_FAR` | Fix the input, retry once. |
| 400 | `LEAD_MAGNET_COMMENT_REQUIRED` / `LEAD_MAGNET_COMMENT_TOO_LONG` / `INVALID_LEAD_MAGNET_COPIES` / `LEAD_MAGNET_TARGET_COMMENTS_MISMATCH` | Fix the lead magnet config and retry once. |
| 400 | `CAROUSEL_PDF_URL_REQUIRED` / `CAROUSEL_PDF_TITLE_TOO_LONG` / `UNSAFE_CAROUSEL_PDF_URL` / `CAROUSEL_PDF_FETCH_FAILED` / `INVALID_CAROUSEL_PDF_TYPE` / `CAROUSEL_PDF_TOO_LARGE` | Fix the PDF URL/file and retry with a fresh idempotency key. |
| 400 | `PLATFORM_NOT_SUPPORTED` | The channel exists but isn't LinkedIn — v1 is LinkedIn-only. Pick another channel. |
| 401 | `INVALID_API_KEY` | Ask user for a fresh key. Do NOT retry. |
| 403 | `CHANNEL_NOT_OWNED` | The `channelId` is from another orga — re-fetch `/channels`. |
| 403 | `PLAN_NO_LONGER_ELIGIBLE` | The user has been downgraded. Stop. |
| 404 | `CHANNEL_NOT_FOUND` | The `channelId` is missing, unknown, or soft-deleted — re-fetch `/channels`. |
| 404 | `POST_NOT_FOUND` | The post id is unknown or belongs to another orga (on `GET /posts/:id`). |
| 409 | `IDEMPOTENCY_KEY_REUSED` | Same `X-Idempotency-Key` used with a different body within 24 h. Generate a fresh UUID. |
| 412 | `CHANNEL_AUTH_EXPIRED` | The LinkedIn session expired; tell user to reconnect. |
| 429 | `RATE_LIMIT_EXCEEDED` | Honour `retryAfterSeconds`. Do NOT spin. |

## Check post status

```bash
curl -s https://api-podawaa.com/api/v1/posts/<id> \
  -H "Authorization: Bearer $PODAWAA_API_KEY"
```

Status progression: `draft` → `scheduled` → `publishing` → `published` (or `failed`).

### When status is `failed`

The response includes a `publishResults` array describing what went wrong on
each channel. The most common cause is an expired LinkedIn session:

```json
{
  "id": "550e8400-...",
  "status": "failed",
  "publishResults": [
    {
      "platform": "linkedin",
      "channelId": "550e8400-...",
      "success": false,
      "error": "LINKEDIN_AUTH_REQUIRED",
      "requiresAuth": true
    }
  ]
}
```

What happened: the user's `li_at` cookie (the session Podawaa uses to publish
on their behalf) is expired, revoked, or was never set. The post was created
and the boost was queued, but LinkedIn refused the publication call.

**What to do:**
1. Surface a clear message to the user: *"Your LinkedIn session expired.
   Reconnect LinkedIn in your Podawaa dashboard, then I'll publish again."*
2. **Do not retry with the same `X-Idempotency-Key`.** The cache holds the
   original `201 scheduled` response (publish failures happen later, in the
   async cron, after the 201 was already returned), so reusing the key replays
   that `scheduled` 201 and does NOT trigger a new publish attempt. Generate a
   fresh UUID for the next attempt.
3. The original draft stays in `failed`; the queued boost will not match any
   members because the post never went live. Both are safe to leave — the user
   can clean them from the dashboard.

Any other `error` value in `publishResults` is an upstream LinkedIn message
passed through verbatim — treat it as `PUBLISH_FAILED` and surface the raw
string to the user. Do not assume a specific code (e.g. there is no
`LINKEDIN_RATE_LIMITED` enum today; you'd see the raw LinkedIn HTTP error).

## Identity

```bash
curl -s https://api-podawaa.com/api/v1/me \
  -H "Authorization: Bearer $PODAWAA_API_KEY"
```

Returns the plan key and your current rate limits.

## Limits

- **100 req/min** per key
- **5000 req/day** per key
- PDF carousels: 10 MB max, HTTPS public URL only.
- The post itself is subject to anti-bot rate-limiting; do not chain calls
  faster than the user would type. If you schedule N posts in a row, space
  the `scheduledAt` values out (5-10 min apart minimum).

## Out of scope for v1

There is **no** standalone boost / engagement endpoint, **no** webhook,
**no** delete-post, **no** comments fetch. Do not invent endpoints. If the user
asks for those, say they are dashboard-only for now.

## Workflow you should follow

1. Check for `PODAWAA_API_KEY` in env. If missing, prompt the user for it once.
2. Call `/channels` to list available accounts (cache for the session).
3. If the user did not specify the channel and has more than one, ask.
4. Validate `content` length, `scheduledAt` rules, lead magnet fields, and PDF
   carousel URL before sending. If the user only has a local/attached PDF, ask
   for a direct public HTTPS URL first.
5. POST `/posts`, surface the returned `id` and `status` to the user.
6. If they ask "is my post out?", poll `/posts/:id` — no faster than once every
   30 s, give up after 5 attempts.

## Example prompts the user might give you

- "Post on LinkedIn now: 'Big launch tomorrow.'"
- "Schedule a LinkedIn post for next Monday 9am with a first comment that
  links to my product page."
- "Post this carousel PDF tomorrow at 9am and add a lead magnet asking people
  to comment DEALS."
- "Take this Notion DB row and turn it into a LinkedIn post scheduled for
  tomorrow morning."
- "Push my last 3 GitHub releases to LinkedIn — one per day at 10am."
- "Draft a 'we are hiring' post for my Acme channel and schedule it for Tuesday."

In every case: list channels, draft content under 3 000 chars, pick a valid
`scheduledAt`, call `POST /posts`, report the returned `id` and status. Never
fabricate `channelId` or pretend a post is published when the API returned
`status: draft`.
