# Questbee API Reference

---

## Base URL

```
http://<your-server>:8000/api/v1
```

Replace `<your-server>` with your server's IP or domain. In local development use `localhost`.

Interactive documentation (Swagger UI) is available at `http://<your-server>:8000/docs`.

---

## Authentication

### JWT (Web Dashboard Users)

Obtain a token via `POST /auth/login`. Pass it in all subsequent requests:

```
Authorization: Bearer <access_token>
```

Access tokens expire after 15 minutes. Use `POST /auth/refresh` with the refresh token to get a new one.

### API Keys (Headless / IoT)

API keys are created in the dashboard under **Settings → API Keys**. Pass the key in every request:

```
X-API-Key: <your_key>
```

> The key is shown only once at creation time. Store it securely.

### Device Token (Mobile App)

The mobile app receives a device token during QR-code pairing. It is used as a Bearer token and managed automatically by the app.

---

## Auth Endpoints

### Login
```
POST /auth/login
```
**Request:**
```json
{ "email": "admin@example.com", "password": "your_password" }
```
**Response:**
```json
{
  "access_token": "eyJ...",
  "refresh_token": "eyJ...",
  "token_type": "bearer"
}
```

---

### Refresh Token
```
POST /auth/refresh
```
**Request:** `{ "refresh_token": "eyJ..." }`
**Response:** New `access_token`.

---

### Logout
```
POST /auth/logout
```
Invalidates the current session. No request body needed.

---

### Change Password
```
POST /auth/change-password
```
**Request:**
```json
{ "current_password": "old", "new_password": "NewPass123!" }
```

---

### Get Current User
```
GET /auth/me
```
Returns the authenticated user's profile.

---

## Projects

### List Projects
```
GET /projects
```

### Get Project
```
GET /projects/<project_id>
```

### Create Project
```
POST /projects
```
```json
{ "name": "2026 Census" }
```

### Delete Project
```
DELETE /projects/<project_id>
```

---

## Forms

### List Forms
```
GET /forms
```

| Parameter | Required | Description |
|---|---|---|
| `project_id` | No | Filter by project |
| `status` | No | `"draft"` or `"published"` |

**Response:**
```json
[
  {
    "id": "uuid",
    "name": "Household Survey",
    "project_id": "uuid",
    "published": true,
    "created_at": "2026-03-17T10:00:00Z"
  }
]
```

---

### Get Form
```
GET /forms/<form_id>
```
Returns the form with its current schema.

---

### Create Form
```
POST /forms
```
**Request:**
```json
{
  "project_id": "uuid",
  "name": "Household Survey",
  "schema": {
    "title": "Household Survey",
    "fields": [
      { "id": "full_name", "type": "text", "label": "Full Name", "required": true },
      { "id": "gps", "type": "geopoint", "label": "GPS Location" }
    ]
  }
}
```

---

### Update Form (Draft)
```
PUT /forms/<form_id>
```
Updates the form name or schema while it is in draft state.

---

### Publish Form
```
POST /forms/<form_id>/publish
```
Freezes the current draft as an immutable published version. Mobile apps download the new version on next sync.

---

### List Form Versions
```
GET /forms/<form_id>/versions
```
Returns all published versions for the form, newest first.

---

### Webhooks

#### Create Webhook
```
POST /forms/<form_id>/webhooks
```
**Request:**
```json
{ "url": "https://your-system.com/hook", "secret": "optional_signing_secret" }
```
The webhook fires with a POST to the given URL on every new submission. The body is the full submission JSON.

#### List Webhooks
```
GET /forms/<form_id>/webhooks
```

#### Delete Webhook
```
DELETE /forms/<form_id>/webhooks/<webhook_id>
```

---

## Submissions

### List Submissions
```
GET /submissions
```

| Parameter | Required | Description |
|---|---|---|
| `form_id` | No | Filter by form |
| `start_date` | No | ISO 8601 — inclusive lower bound on `collected_at` |
| `end_date` | No | ISO 8601 — inclusive upper bound |
| `skip` | No | Pagination offset |
| `limit` | No | Page size (default: 50) |

---

### Get Submission
```
GET /submissions/<submission_id>
```
Returns full submission data including the list of attached media files.

---

### Submit (Single)
```
POST /submissions
```
**Request:**
```json
{
  "form_id": "uuid",
  "form_version_id": "uuid",
  "local_uuid": "device-generated-uuid",
  "data": { "full_name": "Maria Lopez", "gps": { "latitude": -0.2295, "longitude": -78.5243 } },
  "device_id": "uuid",
  "collected_at": "2026-03-17T14:35:00Z"
}
```
`local_uuid` ensures idempotency — resubmitting the same UUID is silently ignored.

---

### Bulk Sync (Mobile)
```
POST /submissions/bulk
```
Accepts an array of submissions in one request. Each is processed independently; duplicates are skipped. Used by the mobile sync engine.

---

## Export

All export endpoints accept the following query parameters:

| Parameter | Required | Description |
|---|---|---|
| `form_id` | Yes | Form to export |
| `from` | No | ISO 8601 date — start of range |
| `to` | No | ISO 8601 date — end of range |

### Export CSV
```
GET /submissions/export/csv
```
Returns a ZIP file containing:
- `<form_name>.csv` — one row per submission; media fields show the `media_file_id`
- `<form_name>_<repeat_field>.csv` — one file per repeat group field

Column headers use field labels. Duplicate labels get the field ID appended in parentheses.

`select_multiple` values are pipe-separated (`A|B|C`). `geopoint` flattens to four columns (`_lat`, `_lng`, `_alt`, `_accuracy`). Submissions where a media file was not uploaded yet show `[upload_missing]`.

### Export GeoJSON
```
GET /submissions/export/geojson
```
Returns a GeoJSON `FeatureCollection`. Each GPS field type produces features:
- `geopoint` → `Point`
- `geotrace` → `LineString`
- Route fields → `LineString` with telemetry in `properties`

All non-spatial submission fields are included as feature properties.

### Export GPX
```
GET /submissions/export/gpx
```
Returns a GPX file. Route fields become `<trk>` elements with `<trkseg>` breaks at pause boundaries.

### Export Media Files
```
GET /submissions/export/media
```
Returns a ZIP archive containing all uploaded media files organized as `media/<submission_id>/<field_id>/<filename>`, plus a `media/index.csv` mapping each file to its submission and field.

### Export Full Package
```
GET /submissions/export/package
```
Returns a single ZIP combining all of the above: CSV files, GeoJSON, GPX, and the full media archive.

---

## Media

### Upload Media File
```
POST /media/upload
```
`Content-Type: multipart/form-data`

| Field | Description |
|---|---|
| `file` | The file to upload |
| `submission_local_uuid` | The local UUID of the associated submission |
| `field_id` | The form field ID this file belongs to |

**Response:**
```json
{ "media_file_id": "uuid", "filename": "photo.jpg", "size_bytes": 204800 }
```

### Get Media File
```
GET /media/<media_id>
```
Returns the file with appropriate `Content-Type`. Requires authentication.

---

## Headless API (IoT / Automation / AI)

The headless API is the external integration surface for automated data collection. It uses API key authentication and is intentionally scoped to **read structure + submit data** — form and project management is only available through the web dashboard.

All four endpoints accept the same header:
```
X-API-Key: <your_key>
```

---

### List Projects
```
GET /headless/projects
```
Returns all projects visible to the API key's tenant.

**Response:**
```json
[
  { "id": "uuid", "name": "Rural Census 2026", "description": "..." }
]
```

---

### List Forms
```
GET /headless/forms
```

| Parameter | Required | Description |
|---|---|---|
| `project_id` | No | Filter by project |

Returns only **published** forms.

**Response:**
```json
[
  {
    "id": "uuid",
    "name": "Household Survey",
    "project_id": "uuid",
    "version_num": 3
  }
]
```

---

### Get Form Schema
```
GET /headless/forms/<form_id>/schema
```
Returns the full published schema for the form — field types, labels, options, validation rules, and conditional logic. Use this to understand what data a `submit` call should contain.

**Response:**
```json
{
  "id": "uuid",
  "name": "Household Survey",
  "version_num": 3,
  "schema": {
    "version": 1,
    "title": "Household Survey",
    "fields": [
      { "id": "full_name", "type": "text", "label": "Full Name", "required": true },
      { "id": "has_water", "type": "select_one", "label": "Running water?",
        "options": [{"value": "yes", "label": "Yes"}, {"value": "no", "label": "No"}] }
    ]
  }
}
```

---

### Submit Data
```
POST /headless/submit
```

**Request:**
```json
{
  "form_id": "uuid",
  "data_json": { "full_name": "Maria Lopez", "has_water": "yes" },
  "collected_at": "2026-03-22T14:35:00Z",
  "local_uuid": "device-generated-uuid"
}
```

`local_uuid` is optional but recommended — resubmitting the same UUID is silently ignored (idempotency).

The payload is validated against the form's published schema before being stored.

**Response (201):**
```json
{ "status": "accepted", "id": "uuid" }
```

**Python example (full flow):**
```python
import requests

BASE = "http://your-server:8000/api/v1/headless"
HEADERS = {"X-API-Key": "your_key_here"}

# 1. Discover forms
projects = requests.get(f"{BASE}/projects", headers=HEADERS).json()
forms = requests.get(f"{BASE}/forms", headers=HEADERS,
                     params={"project_id": projects[0]["id"]}).json()

# 2. Read the schema
schema = requests.get(f"{BASE}/forms/{forms[0]['id']}/schema", headers=HEADERS).json()

# 3. Submit
response = requests.post(f"{BASE}/submit", headers=HEADERS, json={
    "form_id": forms[0]["id"],
    "data_json": {"full_name": "Maria Lopez", "has_water": "yes"},
})
print(response.json())  # {"status": "accepted", "id": "..."}
```

---

## API Keys

### Create API Key
```
POST /api-keys
```
**Request:**
```json
{
  "name": "Sensor Lab 3"
}
```
**Response:**
```json
{ "id": "uuid", "key": "qb_...", "scopes": [], "created_at": "2026-03-22T10:00:00Z" }
```
> The key is shown only once. Store it now.

### List API Keys
```
GET /api-keys
```
Returns keys without secrets.

### Revoke API Key
```
DELETE /api-keys/<key_id>
```

---

## Settings

### Generate Mobile Pairing Token
```
POST /settings/mobile/pairing-token
```
Generates a short-lived token for QR-code pairing. The token embeds the caller's user identity so the mobile device is pre-authenticated.

**Response:**
```json
{
  "token": "abc123...",
  "expires_at": "2026-03-22T11:30:00Z",
  "user_id": "uuid",
  "user_email": "manager@org.com"
}
```

### List Connected Devices
```
GET /settings/mobile/devices
```
Returns all non-revoked paired devices. Admins and managers see all devices; field workers see only their own.

### Revoke Device
```
DELETE /settings/mobile/devices/<device_id>
```
Immediately invalidates the device token. The device will be unable to sync until re-paired.

### Get Server URL
```
GET /settings/mobile/server-url
```

### Set Server URL
```
PUT /settings/mobile/server-url
```
```json
{ "server_url": "https://your-domain.com" }
```

---

## Users

### List Users
```
GET /users
```

### Create User
```
POST /users
```
```json
{ "email": "worker@org.com", "password": "Pass123!", "role": "field_worker" }
```
Roles: `admin`, `manager`, `field_worker`.

### Delete User
```
DELETE /users/<user_id>
```

---

## Health Check

```
GET /health
```
```json
{ "status": "ok", "database": "connected", "timestamp": "2026-03-22T10:00:00Z" }
```

---

## Mobile Pairing

```
POST /mobile/pair
```
Called by the mobile app after scanning a QR code.

**Request:**
```json
{ "token": "abc123...", "device_label": "Samsung Galaxy A55" }
```
**Response:**
```json
{
  "device_token": "eyJ...",
  "user_id": "uuid",
  "user_email": "manager@org.com"
}
```

### Mobile: Download Forms
```
GET /mobile/forms
```
Returns all published forms assigned to the authenticated device's user. Used by the mobile app on initial pair and subsequent syncs.

---

## Error Codes

| Status | Meaning |
|---|---|
| 200 | OK |
| 201 | Created |
| 204 | No content (successful delete) |
| 400 | Bad request — invalid input |
| 401 | Unauthorized — missing or invalid credentials |
| 403 | Forbidden — authenticated but not allowed |
| 404 | Not found |
| 409 | Conflict — e.g. duplicate `local_uuid` (treated as success, not an error, in bulk sync) |
| 422 | Validation error — payload failed schema check |
| 500 | Server error |

---

## See Also

- [Architecture](architecture.md) — How the API fits into the system
- [Getting Started](getting-started.md) — First setup and dashboard walkthrough
- [Deployment Guide](deployment.md) — Production configuration
