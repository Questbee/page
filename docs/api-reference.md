# Questbee API Reference

Complete reference for the Questbee REST API.

## Base URL

```
http://localhost:8000/api/v1
```

(Update `host` and `port` as needed for your deployment)

## Authentication

### Bearer Token (for web dashboard users)

Include the JWT token in the `Authorization` header:

```bash
curl -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  http://localhost:8000/api/v1/forms
```

### API Key (for headless / IoT integrations)

Include the API key and secret in headers:

```bash
curl -H "X-API-Key: YOUR_API_KEY" \
     -H "X-API-Secret: YOUR_API_SECRET" \
  http://localhost:8000/api/v1/headless/submit
```

## Core Endpoints

### Forms

#### 1. Get All Forms
```
GET /forms
```

**Query Parameters:**
- `project_id` (required) — Filter by project
- `status` (optional) — "draft" | "published"
- `skip` (optional) — Pagination offset
- `limit` (optional) — Pagination limit

**Response:**
```json
{
  "forms": [
    {
      "id": "form-123",
      "name": "Field Inspection",
      "project_id": "proj-456",
      "schema": { "fields": [...] },
      "version": 1,
      "status": "published",
      "created_at": "2026-03-16T10:00:00Z",
      "updated_at": "2026-03-16T10:00:00Z"
    }
  ],
  "total": 10
}
```

#### 2. Get Single Form
```
GET /forms/{form_id}
```

**Response:**
```json
{
  "id": "form-123",
  "name": "Field Inspection",
  "schema": {
    "fields": [
      {
        "id": "field_1",
        "type": "text",
        "label": "Inspector Name",
        "required": true
      },
      {
        "id": "field_2",
        "type": "geopoint",
        "label": "Location",
        "required": false
      }
    ]
  },
  "version": 1
}
```

#### 3. Create Form
```
POST /forms
```

**Request Body:**
```json
{
  "project_id": "proj-456",
  "name": "Field Inspection",
  "schema": {
    "fields": [
      {
        "id": "field_1",
        "type": "text",
        "label": "Inspector Name",
        "required": true
      }
    ]
  }
}
```

**Response:** Returns the created form with ID.

#### 4. Update Form (Create New Version)
```
POST /forms/{form_id}/versions
```

**Request Body:** Same as create (schema changes)

**Response:** New version number returned.

---

### Submissions

#### 1. Get Submissions
```
GET /submissions
```

**Query Parameters:**
- `form_id` (required)
- `start_date` (optional) — ISO 8601 date
- `end_date` (optional) — ISO 8601 date
- `skip` (optional)
- `limit` (optional)

**Response:**
```json
{
  "submissions": [
    {
      "id": "sub-123",
      "form_id": "form-456",
      "form_version": 1,
      "data": {
        "field_1": "John Doe",
        "field_2": { "latitude": 0.123, "longitude": -78.456, "accuracy": 5 }
      },
      "submitted_by": "user-789",
      "submitted_at": "2026-03-16T10:15:00Z",
      "device_id": "mobile-device-uuid",
      "metadata": {
        "app_version": "1.0.0",
        "os": "android"
      }
    }
  ],
  "total": 42
}
```

#### 2. Submit Form (Mobile / Web)
```
POST /submissions
```

**Request Body:**
```json
{
  "form_id": "form-123",
  "form_version": 1,
  "data": {
    "field_1": "John Doe",
    "field_2": {
      "latitude": 0.123,
      "longitude": -78.456,
      "accuracy": 5
    }
  },
  "device_id": "mobile-device-uuid"
}
```

**Response:**
```json
{
  "id": "sub-789",
  "status": "accepted",
  "created_at": "2026-03-16T10:15:00Z"
}
```

---

### Headless API (IoT / Automated Data Ingestion)

#### Submit Data (Authenticated via API Key)
```
POST /headless/submit
```

**Headers:**
```
X-API-Key: YOUR_API_KEY
X-API-Secret: YOUR_API_SECRET
Content-Type: application/json
```

**Request Body:**
```json
{
  "form_id": "form-123",
  "data": {
    "temperature": 25.5,
    "humidity": 60,
    "timestamp": "2026-03-16T10:15:00Z"
  }
}
```

**Response:**
```json
{
  "submission_id": "sub-789",
  "status": "success",
  "message": "Data ingested successfully"
}
```

**Error Response (400):**
```json
{
  "error": "validation_error",
  "details": {
    "temperature": "Required field"
  }
}
```

---

### API Keys

#### Generate API Key
```
POST /api-keys
```

**Request Body:**
```json
{
  "name": "IoT Temperature Sensor",
  "project_id": "proj-456",
  "form_ids": ["form-123"],
  "expiry_days": 365
}
```

**Response:**
```json
{
  "id": "key-123",
  "key": "qb_key_abc123def456...",
  "secret": "qb_secret_xyz789abc...",
  "created_at": "2026-03-16T10:00:00Z"
}
```

⚠️ **Important:** Store the `secret` securely. It won't be retrievable again.

#### List API Keys
```
GET /api-keys?project_id=proj-456
```

#### Revoke API Key
```
DELETE /api-keys/{key_id}
```

---

### Projects

#### Get Projects
```
GET /projects
```

**Response:**
```json
{
  "projects": [
    {
      "id": "proj-456",
      "tenant_id": "tenant-123",
      "name": "2026 Census",
      "created_at": "2026-03-16T10:00:00Z"
    }
  ]
}
```

#### Create Project
```
POST /projects
```

**Request Body:**
```json
{
  "name": "2026 Census"
}
```

---

### Health Check

#### System Status
```
GET /health
```

**Response:**
```json
{
  "status": "ok",
  "database": "connected",
  "timestamp": "2026-03-16T10:15:00Z"
}
```

---

## Error Codes

| Code | Status | Meaning |
| :--- | :--- | :--- |
| 200 | OK | Request successful |
| 201 | Created | Resource created |
| 400 | Bad Request | Invalid input data |
| 401 | Unauthorized | Missing/invalid auth |
| 403 | Forbidden | Access denied |
| 404 | Not Found | Resource not found |
| 409 | Conflict | Data conflict (e.g., duplicate) |
| 500 | Server Error | Unexpected server error |

---

## Rate Limiting

**Default Limits:**
- 1000 requests per minute per API key
- 100 requests per minute per user session

**Headers:**
```
X-RateLimit-Limit: 1000
X-RateLimit-Remaining: 999
X-RateLimit-Reset: 1234567890
```

---

## Code Examples

### Python
```python
import requests

api_key = "qb_key_abc123..."
api_secret = "qb_secret_xyz789..."

headers = {
    "X-API-Key": api_key,
    "X-API-Secret": api_secret
}

payload = {
    "form_id": "form-123",
    "data": {
        "temperature": 25.5,
        "humidity": 60
    }
}

response = requests.post(
    "http://localhost:8000/api/v1/headless/submit",
    json=payload,
    headers=headers
)

print(response.json())
```

### JavaScript / Node.js
```javascript
const axios = require('axios');

const apiKey = 'qb_key_abc123...';
const apiSecret = 'qb_secret_xyz789...';

const payload = {
  form_id: 'form-123',
  data: {
    temperature: 25.5,
    humidity: 60
  }
};

axios.post('http://localhost:8000/api/v1/headless/submit', payload, {
  headers: {
    'X-API-Key': apiKey,
    'X-API-Secret': apiSecret
  }
})
.then(response => console.log(response.data))
.catch(error => console.error(error.response.data));
```

### cURL
```bash
curl -X POST http://localhost:8000/api/v1/headless/submit \
  -H "X-API-Key: qb_key_abc123..." \
  -H "X-API-Secret: qb_secret_xyz789..." \
  -H "Content-Type: application/json" \
  -d '{
    "form_id": "form-123",
    "data": {
      "temperature": 25.5,
      "humidity": 60
    }
  }'
```

---

## Webhooks (Optional)

When a submission is received, trigger a webhook:

**Configure Webhook:**
```
POST /forms/{form_id}/webhooks
```

**Request Body:**
```json
{
  "url": "https://your-system.com/questbee/submissions",
  "events": ["submission.created"]
}
```

**Webhook Payload:**
```json
{
  "event": "submission.created",
  "submission_id": "sub-789",
  "form_id": "form-123",
  "data": { ... },
  "timestamp": "2026-03-16T10:15:00Z"
}
```

---

For more examples and detailed documentation, visit the [GitHub Repository](https://github.com/your-org/questbee).
