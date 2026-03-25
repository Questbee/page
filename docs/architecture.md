# Questbee Architecture Overview

---

## System Overview

Questbee has three client surfaces — a web dashboard, a mobile app, and a headless API — all backed by a single FastAPI service and a PostgreSQL database, packaged as a Docker Compose stack that runs entirely on your own server.

```
┌────────────────────────────────────────────────────────────────────┐
│                         CLIENT SURFACES                            │
├─────────────────┬──────────────────────┬───────────────────────────┤
│  Web Dashboard  │    Mobile App        │    Headless API / IoT     │
│  (Next.js)      │    (React Native)    │    (API key auth)         │
│  Browser        │    iOS + Android     │    External systems       │
└────────┬────────┴──────────┬───────────┴───────────┬───────────────┘
         │                   │                        │
         │         HTTPS / REST API (FastAPI)         │
         └───────────────────┼────────────────────────┘
                             │
         ┌───────────────────▼────────────────────────┐
         │              FastAPI Backend               │
         │  • Forms & versioned schemas               │
         │  • Offline sync engine                     │
         │  • Auth: JWT + API keys + device tokens    │
         │  • Role-based access control               │
         │  • Media file storage                      │
         │  • Data export (CSV, GeoJSON, GPX, ZIP)    │
         └───────────────────┬────────────────────────┘
                             │
         ┌───────────────────▼────────────────────────┐
         │              PostgreSQL                    │
         │  • Forms, submissions (JSONB)              │
         │  • Users, tenants, projects                │
         │  • API keys, media files, device tokens    │
         └────────────────────────────────────────────┘
```

All containers run inside a Docker Compose stack on your own infrastructure. No data is sent to Questbee servers.

---

## Components

### Web Dashboard (Next.js)

Browser-based interface for program managers and administrators.

- Build and publish forms with a visual form builder
- Manage users and assign roles (Admin, Manager, Field Worker)
- Browse, search, and review submissions with media preview
- Export data as CSV, GeoJSON, GPX, or media ZIP
- Generate pairing QR codes and manage connected devices
- Manage API keys for headless integrations

### Mobile App (React Native / Expo)

Offline-first data collection app for field workers.

- Pairs with the server via QR code — no separate login required
- Downloads assigned form schemas to the device
- Collects data with no internet connection — data saved locally in SQLite
- Automatically syncs pending submissions and media files when back online
- Supports GPS, photos, audio, signatures, barcodes, and more

**Distribution:** prebuilt Android APK and Expo Go for iOS. White-label compilation (custom branding) is available on paid plans.

### Backend API (FastAPI)

The central service. All three client surfaces communicate through it.

| Area | Description |
|---|---|
| Forms | CRUD for form schemas; immutable versioning on publish |
| Submissions | Accept from mobile (JWT) and headless API (API key); idempotent bulk sync |
| Media | File upload and authenticated retrieval; stored on the server filesystem |
| Sync | Bulk submission endpoint with device-UUID deduplication |
| Auth | JWT for dashboard users; API keys for IoT/headless; device tokens for mobile |
| RBAC | Role enforcement: Admin > Manager > Field Worker |
| Export | CSV (flat + repeat groups), GeoJSON, GPX, media ZIP, full package |
| Settings | Mobile pairing tokens, connected device management, server URL config |
| Webhooks | Per-form HTTP webhooks triggered on new submissions |

Base URL: `http://<your-server>:8000/api/v1`

### Database (PostgreSQL)

All persistent data lives here.

Core tables: `tenants`, `projects`, `forms`, `form_versions`, `submissions`, `users`, `api_keys`, `media_files`, `device_tokens`, `pairing_tokens`

Form schemas and submission data are stored as **JSONB** — flexible enough to support any combination of field types without a schema migration per form change.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | FastAPI (Python 3.11+) |
| Web dashboard | Next.js 14 + TypeScript + Tailwind CSS |
| Mobile app | React Native / Expo (TypeScript) |
| Mobile local DB | SQLite (via Expo SQLite) |
| Server DB | PostgreSQL 15 |
| Containers | Docker + Docker Compose v2 |

Minimum server requirements: **2 CPU cores, 4 GB RAM, 50 GB disk**, Ubuntu 22.04 or equivalent.

---

## Data Flow: Offline Mobile Sync

```
Field worker fills form — no internet
        │
        ▼
Saved to SQLite on device (status: pending)
Media files (photos, audio) stored on device
        │
Device reconnects
        │
        ▼
Sync engine: POST /api/v1/submissions/bulk
Each submission carries a device-generated UUID — retries never create duplicates
        │
        ├─ 200 OK  → submission marked as synced
        └─ error   → retried with exponential backoff

Media uploads: POST /api/v1/media/upload (per file, after submission sync)
```

---

## Device Pairing Flow

```
Admin generates pairing token in dashboard
        │
        ▼
QR code encodes: server URL + one-time token + user identity
        │
Field worker scans QR code with mobile app
        │
        ▼
POST /api/v1/mobile/pair
  - token validated and consumed
  - device_token issued and stored securely on device
  - forms downloaded automatically
        │
        ▼
Subsequent requests: Authorization: Bearer <device_token>
```

No separate login step is required on the device — the QR code embeds the user identity.

---

## Multi-Tenancy

A single Questbee instance serves multiple independent organizations. Each tenant has its own users, projects, forms, and submissions — isolated at the database query level by `tenant_id`.

---

## Security

- **JWT** for web dashboard sessions (short-lived access tokens + refresh tokens)
- **API keys** for headless/IoT integrations (hashed in the database, scoped to projects)
- **Device tokens** for mobile app pairing (per-device, revocable from the dashboard)
- **RBAC** enforced at the API layer on every request
- **Multi-tenant isolation** — all queries are scoped to `tenant_id`
- **No external dependencies** — no calls to third-party services for core functionality

---

## Deployment

Single command to start the full stack (from `questbee/community/`):

```bash
docker compose up --build
```

For production: add Nginx or Caddy as a reverse proxy for TLS termination. See the [Deployment Guide](deployment.md).

---

## See Also

- [Getting Started](getting-started.md) — Step-by-step setup
- [API Reference](api-reference.md) — Endpoint reference
- [Deployment Guide](deployment.md) — Production configuration
