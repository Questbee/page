# Deployment Guide

---

## Contents

1. [Local Development](#local-development)
2. [Production — On-Premise / Private Server](#production--on-premise--private-server)
3. [Production — Cloud](#production--cloud)
4. [Environment Variables](#environment-variables)
5. [Backups](#backups)
6. [Production Checklist](#production-checklist)
7. [Troubleshooting](#troubleshooting)

---

## Local Development

**Requirements:** Docker, Docker Compose v2, Git. 2 CPU cores, 4 GB RAM, 10 GB disk minimum.

```bash
git clone https://github.com/questbee-io/questbee.git
cd questbee/community
cp .env.example .env
# Edit .env — see Environment Variables section
docker compose up --build
```

| Service | URL |
|---|---|
| Web dashboard | http://localhost:3000 |
| API | http://localhost:8000 |
| API docs (Swagger) | http://localhost:8000/docs |

On first login you will be prompted to change the default credentials.

**Stop services:**
```bash
docker compose down
```

**Stop and delete all data:**
```bash
docker compose down -v   # WARNING: destroys database and media volumes
```

---

## Production — On-Premise / Private Server

The most common deployment for governments and NGOs who want full data control.

**Server requirements:**
- Ubuntu 22.04 LTS (or equivalent Linux)
- 2 CPU cores, 4 GB RAM, 50 GB SSD (minimum)
- Static IP address or domain name
- Ports 80 and 443 open to the internet (or your intranet)
- Port 22 open for SSH administration

### 1. Install Docker

```bash
sudo apt update && sudo apt upgrade -y
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
sudo apt install -y docker-compose-plugin
# Log out and back in for group changes to take effect
```

### 2. Deploy Questbee

```bash
# Create application directory
sudo mkdir -p /opt/questbee
sudo chown $USER:$USER /opt/questbee
cd /opt/questbee

# Clone the repository
git clone https://github.com/questbee-io/questbee.git .

# Configure environment
cd community
cp .env.example .env
nano .env   # edit values (see Environment Variables section below)

# Start services in the background
docker compose up -d --build

# Verify all containers are running
docker compose ps
```

### 3. Set Up a Reverse Proxy (TLS / HTTPS)

**Option A — Caddy (recommended, automatic TLS)**

```bash
sudo apt install -y caddy

sudo tee /etc/caddy/Caddyfile > /dev/null << 'EOF'
your-domain.com {
    reverse_proxy /api/* localhost:8000
    reverse_proxy /* localhost:3000
}
EOF

sudo systemctl reload caddy
```

Caddy automatically provisions and renews a Let's Encrypt certificate.

**Option B — Nginx + Certbot**

```bash
sudo apt install -y nginx certbot python3-certbot-nginx

sudo tee /etc/nginx/sites-available/questbee > /dev/null << 'EOF'
server {
    server_name your-domain.com;

    location /api/ {
        proxy_pass http://localhost:8000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        proxy_pass http://localhost:3000;
        proxy_set_header Host $host;
    }
}
EOF

sudo ln -s /etc/nginx/sites-available/questbee /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx

# Obtain TLS certificate
sudo certbot --nginx -d your-domain.com
```

### 4. Configure Firewall

```bash
sudo ufw enable
sudo ufw allow 22/tcp    # SSH
sudo ufw allow 80/tcp    # HTTP (redirects to HTTPS)
sudo ufw allow 443/tcp   # HTTPS
# Do NOT expose port 5432 (PostgreSQL), 8000, or 3000 directly
```

---

## Production — Cloud

Questbee runs on any Linux host that supports Docker. Cloud deployments follow the same steps as on-premise, just on a cloud VM.

**Recommended VM sizing:**

| Provider | Instance | Spec |
|---|---|---|
| AWS | t3.medium | 2 vCPU, 4 GB RAM |
| Azure | B2s | 2 vCPU, 4 GB RAM |
| GCP | e2-medium | 2 vCPU, 4 GB RAM |
| DigitalOcean | 4 GB Droplet | 2 vCPU, 4 GB RAM |

---

## Environment Variables

Create a `.env` file in `community/`. Never commit this file to version control.

```env
# Database
DB_PASSWORD=a_very_strong_password

# Security
SECRET_KEY=replace_with_64_random_hex_chars   # openssl rand -hex 32
ADMIN_EMAIL=admin@your-domain.com
ADMIN_PASSWORD=ChangeMe123!

# CORS and API URL
ALLOWED_ORIGINS=https://your-domain.com
NEXT_PUBLIC_API_URL=https://your-domain.com/api/v1

# Optional tuning
MAX_UPLOAD_MB=20
SYNC_BATCH_SIZE=100
```

| Variable | Required | Default | Description |
|---|:---:|---|---|
| `DB_PASSWORD` | Yes | — | PostgreSQL password |
| `SECRET_KEY` | Yes | — | JWT signing secret (32+ random hex chars) |
| `ADMIN_EMAIL` | Yes | — | Auto-created admin account email |
| `ADMIN_PASSWORD` | Yes | — | Initial admin password (forced reset on first login) |
| `ALLOWED_ORIGINS` | Yes | — | Comma-separated CORS origins |
| `NEXT_PUBLIC_API_URL` | Yes | — | API URL as seen from the browser |
| `MAX_UPLOAD_MB` | No | `20` | Max media file upload size in MB |
| `SYNC_BATCH_SIZE` | No | `100` | Max submissions per bulk sync request |

---

## Backups

Questbee does not manage backups on self-hosted deployments — this is the customer's responsibility.

**Recommended: daily automated database backup**

```bash
sudo tee /usr/local/bin/questbee-backup.sh > /dev/null << 'EOF'
#!/bin/bash
BACKUP_DIR="/backups/questbee"
mkdir -p "$BACKUP_DIR"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Database dump
docker exec questbee-community-db-1 \
  pg_dump -U questbee questbee | gzip > "$BACKUP_DIR/db_$TIMESTAMP.sql.gz"

# Keep last 30 days only
find "$BACKUP_DIR" -name "*.gz" -mtime +30 -delete

echo "Backup complete: db_$TIMESTAMP.sql.gz"
EOF

chmod +x /usr/local/bin/questbee-backup.sh

# Schedule daily at 02:00
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/questbee-backup.sh") | crontab -
```

**Restore from backup:**
```bash
gunzip < /backups/questbee/db_20260317_020000.sql.gz | \
  docker exec -i questbee-community-db-1 psql -U questbee questbee
```

**Media files** are stored in the `media-data` Docker volume. Back it up with:
```bash
docker run --rm -v questbee_media-data:/data -v /backups/questbee:/backup \
  alpine tar czf /backup/media_$(date +%Y%m%d).tar.gz -C /data .
```

---

## Production Checklist

- [ ] Default credentials changed on first login
- [ ] `SECRET_KEY` set to a long random value (not the example)
- [ ] Database password is strong and not the default
- [ ] `.env` file is not committed to version control
- [ ] TLS/HTTPS configured and certificate is valid
- [ ] `ALLOWED_ORIGINS` and `NEXT_PUBLIC_API_URL` set to your actual domain
- [ ] Firewall configured — only ports 80 and 443 are exposed
- [ ] Database port (5432) is not exposed externally
- [ ] Daily database backup is scheduled and tested
- [ ] Media volume is included in backup schedule
- [ ] `docker compose ps` shows all containers as `Up`
- [ ] Health check endpoint responds: `GET /api/v1/health`

---

## Day-to-Day Operations

All commands run from `questbee/community/`:

| Task | Command |
|---|---|
| Start (first time / after code changes) | `docker compose up --build -d` |
| Normal restart | `docker compose up -d` |
| Stop, keep data | `docker compose down` |
| Stop, wipe all data | `docker compose down -v` |
| View logs | `docker compose logs -f api` |
| Update (pull + rebuild) | `git pull && docker compose up --build -d` |

---

## Troubleshooting

**Containers not starting**
```bash
docker compose logs api
docker compose logs db
docker compose logs web
```

**Database connection error**
```bash
docker compose ps
docker exec questbee-community-db-1 psql -U questbee -c "SELECT 1"
docker exec questbee-community-api-1 printenv DATABASE_URL
```

**Dashboard not reachable**
```bash
docker compose ps
docker compose port web 3000
```

**Disk space issues**
```bash
df -h
docker system prune    # remove unused containers and images
```

---

## See Also

- [Getting Started](getting-started.md) — Local development setup
- [Architecture](architecture.md) — System design
- [API Reference](api-reference.md) — Endpoint reference
