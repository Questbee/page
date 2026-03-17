# Questbee Architecture

> ⚠️ **Pre-MVP Notice:** This document describes the intended system architecture. Questbee is in pre-MVP development — implementation details are subject to change.

This document provides a high-level overview of Questbee's system architecture.

## System Overview

```
┌─────────────────────────────────────────────────────────────────┐
│                        Field Workers (Offline)                  │
│                  Mobile App (React Native / Expo)               │
│                  Local DB: SQLite / WatermelonDB                │
└────────────────────────────┬────────────────────────────────────┘
                             │ Sync Queue (WiFi / Cellular)
                             ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Self-Hosted Server Cluster                  │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  ┌──────────────────┐  ┌─────────────────┐  ┌────────────────┐ │
│  │  Next.js Web     │  │   FastAPI       │  │  Postgres DB   │ │
│  │  Dashboard       │  │   REST API      │  │  (JSONB store) │ │
│  │                  │  │                 │  │                │ │
│  │  • Form Builder  │  │ • Form Schema   │  │ • Tenants      │ │
│  │  • User Mgmt     │  │ • Submissions   │  │ • Forms        │ │
│  │  • Reporting     │  │ • Sync Engine   │  │ • Submissions  │ │
│  │  • RBAC          │  │ • IoT Headless  │  │ • Audit Logs   │ │
│  └──────────────────┘  │ • MCP Server    │  │                │ │
│                        │                 │  │ • Settings     │ │
│                        └─────────────────┘  └────────────────┘ │
│                                                                 │
│  Docker Container Network (Internal)                           │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
         │                    │
         ├──────────────────┬─┴──────────────────┐
         ▼                  ▼                     ▼
    External              IoT Devices /       Admin Portal
    Web Forms             Headless API          (Browser)
```

## Component Details

### 1. Mobile App (React Native / Expo)

**Purpose:** Offline-first field data collection

**Key Features:**
- **Offline-First Storage:** SQLite / WatermelonDB stores form schemas and pending submissions locally
- **Sync Engine:** Background process that queues submissions and syncs when online
- **Auto-Capture:** GPS coordinate, timestamp, device ID recorded with each submission
- **Media Support:** Photo, audio, video captured and stored locally before sync
- **Server Pairing:** QR code–based pairing to connect to private server

**Tech Stack:**
- React Native / Expo
- SQLite / WatermelonDB (local DB)
- Axios / React Query (HTTP client)

### 2. Web Dashboard (Next.js + Tailwind CSS)

**Purpose:** Administrative interface for form design, user management, and data analysis

**Key Features:**
- **Form Builder:** Drag-and-drop UI to design JSONB form schemas
- **User Management:** Create users, assign roles (Admin, Manager, Field Worker)
- **Project Management:** Organize forms and submissions by project
- **Data Visualization:** View submissions, filter, search, export
- **API Key Management:** Generate/revoke API keys for headless integrations
- **Settings & White-Labeling:** Customize logos, colors, themes

**Tech Stack:**
- Next.js (React framework)
- TypeScript
- Tailwind CSS
- SWR / React Query

### 3. Backend API (FastAPI)

**Purpose:** Core business logic, data validation, and integration layer

**Key Endpoints:**
- `POST /api/v1/forms` — Create/update form schemas
- `POST /api/v1/submissions` — Submit form data (mobile)
- `GET /api/v1/submissions?form_id=X` — Query submissions
- `POST /api/v1/headless/submit` — Headless data ingestion (IoT)
- `GET /api/v1/health` — Health check

**Key Modules:**
- **Schema Validator:** Validates submissions against JSONB form schemas
- **Sync Queue:** Manages retry logic for failed submissions
- **MCP Server:** Exposes AI agent tools
- **Auth:** JWT-based authentication + API key system
- **RBAC:** Role-based access control enforcer

**Tech Stack:**
- FastAPI (Python web framework)
- Pydantic (data validation)
- SQLAlchemy (ORM)
- APScheduler (background jobs)

### 4. Database (PostgreSQL)

**Purpose:** Persistent storage for forms, submissions, users, configurations

**Key Tables:**
- `tenants` — Organizations / customers
- `projects` — Grouping within tenants
- `forms` — Form definitions (JSON schemas stored in JSONB column)
- `form_versions` — Historical versions
- `submissions` — Form responses (JSONB data column)
- `users` — User accounts + roles
- `api_keys` — Headless integration keys
- `audit_logs` — User actions, API usage

**Database Features:**
- **JSONB Columns:** Form schemas and submission data stored as flexible JSONB
- **Full-Text Search:** Search submission data efficiently
- **Indexes:** Optimized for common queries (submissions by form, date range)
- **Backups:** Regular automated backups (customer responsible for on-premises setups)

## Data Flow

### 1. Field Data Submission (Mobile → Server)

```
User fills form offline
        ↓
Data stored in local SQLite
        ↓
Submission queued (pending_submissions)
        ↓
Device reconnects to internet
        ↓
Sync engine: POST /api/v1/submissions
        ↓
Server validates against form schema
        ↓
Data stored in PostgreSQL (submissions table)
        ↓
Sync complete; local queue cleared
```

### 2. Headless IoT Data Ingestion

```
IoT Device / External System
        ↓
POST /api/v1/headless/submit with API credentials
        ↓
Server validates API key
        ↓
Server validates payload against form schema
        ↓
Data stored in PostgreSQL
        ↓
Response: {"status": "success", "submission_id": "..."}
```

### 3. Form Generation (AI / MCP)

```
External AI Agent
        ↓
Call MCP tool: generate_form_schema(prompt)
        ↓
Questbee backend generates JSONB schema
        ↓
Schema stored in forms table
        ↓
Form now available in mobile app + dashboard
        ↓
AI can then inject test data or fetch results
```

## Deployment Architecture

### Container Stack (docker-compose)

```yaml
services:
  db:
    image: postgres:15
    volumes: [postgres-data]
  
  api:
    image: questbee-api:latest
    depends_on: [db]
  
  web:
    image: questbee-web:latest
    depends_on: [api]
```

### Multi-Instance (High Availability)

For production deployments:
- **Load Balancer:** Nginx/HAProxy in front of multiple API instances
- **Database Replication:** PostgreSQL primary + replicas (optional)
- **Object Storage:** S3-compatible storage for media files (photos, videos)

## Security Considerations

### Authentication & Authorization
- **JWT Tokens:** Short-lived tokens for web dashboard users
- **API Keys:** Long-lived keys for IoT/headless integrations (scoped to projects/forms)
- **RBAC:** Role-based access control at the API level

### Data Protection
- **TLS/HTTPS:** All communications encrypted in transit
- **Database Encryption:** At-rest encryption via OS-level (dm-crypt, XFS encryption) or PostgreSQL extensions
- **Audit Logs:** All API actions logged for compliance

### Isolation
- **Multi-Tenancy:** Data strictly isolated per tenant
- **Network Isolation:** Docker networks restrict inter-service communication

## Scaling Considerations

### Horizontal Scaling
- **Stateless API:** Can run multiple instances behind a load balancer
- **Sync Queue:** Use message broker (RabbitMQ, Redis) for distributed queue

### Vertical Scaling
- **Database:** PostgreSQL supports large datasets (terabytes)
- **Caching:** Redis for API caching and session storage

### Performance Tips
- Index frequently queried fields (form_id, user_id, created_at)
- Archive old submissions to optimize table size
- Enable API response caching for read-heavy workloads

## Monitoring & Observability

### Key Metrics
- **API Response Time:** /metrics endpoint
- **Database Query Performance:** Slow query logs
- **Sync Success Rate:** Successful submissions vs. failures
- **Error Rates:** 4xx / 5xx responses

### Logging
- Docker logs: `docker-compose logs -f api`
- Centralized logging (optional): ELK Stack, Splunk, CloudWatch

---

For deployment details, see [Deployment Guide](deployment.md).
