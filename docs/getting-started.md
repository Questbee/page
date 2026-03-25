# Getting Started with Questbee

---

## Prerequisites

- Docker and Docker Compose v2 installed
- Git (to clone the repository)
- At least 2 CPU cores, 4 GB RAM, 50 GB disk
- Basic command-line familiarity

---

## Quick Start

### 1. Clone the Repository

```bash
git clone https://github.com/questbee-io/questbee.git
cd questbee/community
```

### 2. Configure Environment

```bash
cp .env.example .env
```

Edit `.env` — the minimum required values:

```env
DB_PASSWORD=a_strong_password
SECRET_KEY=run_python3_c_import_secrets_print_secrets_token_hex_32
ADMIN_EMAIL=admin@yourorg.com
ADMIN_PASSWORD=ChangeMe123!
ALLOWED_ORIGINS=http://localhost:3000
NEXT_PUBLIC_API_URL=http://localhost:8000/api/v1
```

Generate a proper `SECRET_KEY`:
```bash
python3 -c "import secrets; print(secrets.token_hex(32))"
```

> **Note on email:** use a real-looking domain like `yourorg.com`. Pydantic rejects reserved TLDs like `.local`.

### 3. Start the Stack

```bash
docker compose up --build
```

| Service | URL | Purpose |
|---|---|---|
| Web dashboard | http://localhost:3000 | Next.js admin interface |
| API | http://localhost:8000 | FastAPI backend |
| API docs (Swagger) | http://localhost:8000/docs | Interactive endpoint explorer |

### 4. First Login

1. Open **http://localhost:3000**
2. Log in with `ADMIN_EMAIL` and `ADMIN_PASSWORD` from your `.env`
3. You will be redirected to a **Change Password** screen — this is mandatory on first login
4. After changing your password, you land on the dashboard

---

## First Steps in the Dashboard

### Create a Project

Forms are organized inside Projects. A project maps to one program or initiative.

1. Go to **Forms** → click **New Project**
2. Enter a name (e.g. "2026 Household Survey")
3. Click **Create**

### Create and Build a Form

1. Inside the project, click **New Form**
2. Give it a name and optional description
3. Click **Create** — you land in the form builder
4. Add fields from the toolbar
5. Configure each field: label, required, hint, conditional logic
6. Click **Publish** when the form is ready

**Available field types:**

| Category | Fields |
|---|---|
| Text | Short text, Long text, Number, Email, Phone |
| Date / Time | Date, Time, Date + Time |
| Choice | Single choice, Multiple choice, Single choice with write-in |
| Location | GPS point (geopoint), GPS trace (geotrace) |
| Media | Photo, Audio recording, Signature, File attachment |
| Scan | Barcode / QR code |
| Logic | Calculated value, Hidden field |
| Layout | Note, Divider, Field group, Repeating group |

> **Versioning:** Once published, a form version is locked. Any edit creates a new draft version. Existing submissions always reference the version they were collected against.

---

## Pairing the Mobile App

The mobile app connects to your server via a QR code generated in the dashboard. The QR code embeds both the server URL and a short-lived pairing token linked to your user account — no separate login is required on the device.

### Generate a Pairing Code

1. In the dashboard go to **Settings**
2. Under **Mobile Pairing**, click **Generate QR Code**

### Connect the App

1. Open the Questbee app on your device
2. Tap **Scan QR Code**
3. Point the camera at the QR code shown in the dashboard

The app connects, stores your credentials securely, and downloads your assigned forms automatically. You are ready to collect data.

### Manage Connected Devices

Go to **Settings → Connected Devices** to see all paired devices. You can revoke access for any device from there.

---

## Collecting Data

### Via Mobile App (Offline-First)

1. Open the app → select a form
2. Fill in all fields — no internet required
3. Tap **Submit** — data is saved locally if offline
4. When the device reconnects, pending submissions are uploaded automatically

### Via Web (Online Only)

Web submission from the dashboard is available for managers testing forms or entering data manually.

---

## Viewing Submissions

1. Go to **Submissions** in the sidebar
2. Filter by form using the dropdown
3. Click any row to see full submission details, including media files

---

## Exporting Data

Click **Export** on the Submissions page to download your data. The export modal lets you pick:

| Format | Contents |
|---|---|
| **CSV** | One row per submission; repeat groups in separate sheets |
| **GeoJSON** | GPS points, traces, and routes as a FeatureCollection |
| **GPX** | Route/trace fields as a GPX track file (opens in QGIS, OsmAnd, etc.) |
| **Media files** | ZIP archive of all uploaded files with an index CSV |
| **Full package** | ZIP with all of the above combined |

You can also filter by date range before exporting.

---

## Environment Variables

| Variable | Required | Description |
|---|:---:|---|
| `DB_PASSWORD` | Yes | PostgreSQL password for the `questbee` user |
| `SECRET_KEY` | Yes | JWT signing secret (32+ random hex chars) |
| `ADMIN_EMAIL` | Yes | Email for the auto-created admin account |
| `ADMIN_PASSWORD` | Yes | Initial password (must be changed on first login) |
| `ALLOWED_ORIGINS` | Yes | Comma-separated CORS origins for the web dashboard |
| `NEXT_PUBLIC_API_URL` | Yes | API URL as seen from the browser |
| `MAX_UPLOAD_MB` | No | Max media file upload size in MB (default: 20) |
| `SYNC_BATCH_SIZE` | No | Max submissions per bulk sync request (default: 100) |

---

## Troubleshooting

**Dashboard not loading**
```bash
docker compose logs web
```

**API errors**
```bash
docker compose logs api
```

**Database issues**
```bash
docker compose ps           # check all containers are Up
docker compose restart db   # restart database if needed
```

**Mobile app not syncing**
- Confirm the server URL is reachable from the device network
- Check: `docker compose logs api` for 4xx/5xx errors

**Full reset (delete all data)**
```bash
docker compose down -v
```

---

## Next Steps

- [Architecture](architecture.md) — System design overview
- [API Reference](api-reference.md) — Headless API and IoT ingestion
- [Deployment Guide](deployment.md) — Production setup
- [Mobile App Guide](mobile-app.md) — Full field worker reference
