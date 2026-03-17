# Deployment Guide

> ⚠️ **Pre-MVP Notice:** This deployment guide describes the intended production setup. Questbee is in pre-MVP development — configuration details and Docker image names are subject to change. Contact [hello@questbee.io](mailto:hello@questbee.io) if you are planning a production deployment and want early-access support.

Instructions for deploying Questbee in different environments.

## Table of Contents
1. [Local Development](#local-development)
2. [AWS Deployment](#aws-deployment)
3. [Azure Deployment](#azure-deployment)
4. [On-Premise / Private Server](#on-premise--private-server)
5. [Production Checklist](#production-checklist)

---

## Local Development

### Prerequisites
- Docker & Docker Compose installed
- Git
- 2GB RAM, 10GB disk minimum

### Setup

1. **Clone Repository**
```bash
git clone https://github.com/your-org/questbee.git
cd questbee
```

2. **Start Services**
```bash
docker-compose up --build
```

3. **Verify Services**
```bash
# Check running containers
docker-compose ps

# Expected output:
# questbee-api       Up (Port 8000)
# questbee-web       Up (Port 3000)
# questbee-db        Up (Port 5432)
```

4. **Access Dashboard**
- Web: http://localhost:3000
- API: http://localhost:8000/docs (Swagger UI)
- Database: localhost:5432 (postgres/postgres)

### Cleanup
```bash
# Stop services
docker-compose down

# Remove volumes (WARNING: deletes data)
docker-compose down -v
```

---

## AWS Deployment

### Architecture
- **Compute:** EC2 instance or Elastic Container Service (ECS)
- **Database:** RDS PostgreSQL
- **Storage:** S3 for media files (photos, videos)
- **DNS:** Route 53

### Option 1: EC2 + Docker Compose (Simplest)

#### 1. Launch EC2 Instance
- **OS:** Ubuntu 22.04 LTS
- **Instance Type:** t3.medium (2 vCPU, 4GB RAM minimum)
- **Disk:** 50GB gp3
- **Security Group:** Open ports 80, 443, 22 (SSH)

#### 2. Install Prerequisites
```bash
# SSH into instance
ssh -i your-key.pem ubuntu@your-instance-ip

# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker ubuntu

# Install Docker Compose
sudo apt install -y docker-compose

# Logout and login for group changes
exit
ssh -i your-key.pem ubuntu@your-instance-ip
```

#### 3. Deploy Questbee
```bash
# Clone repository
git clone https://github.com/your-org/questbee.git
cd questbee

# Create .env file for production
cat > .env << EOF
DATABASE_URL=postgresql://postgres:YOUR_STRONG_PASSWORD@db:5432/questbee
API_SECRET_KEY=YOUR_SECRET_KEY_HERE
ENVIRONMENT=production
DOMAIN=your-domain.com
EOF

# Start services
docker-compose -f docker-compose.prod.yml up -d
```

#### 4. Setup Domain & SSL

**Using AWS Route 53:**
1. Create Route 53 hosted zone for your domain
2. Update nameservers with your domain registrar
3. Create A record pointing to EC2 instance's Elastic IP

**Setup HTTPS (Let's Encrypt):**
```bash
# Install Certbot
sudo apt install -y certbot python3-certbot-nginx

# Get certificate
sudo certbot certonly --standalone -d your-domain.com

# Auto-renewal
sudo systemctl enable certbot.timer
```

**Nginx Reverse Proxy:**
```bash
# Install Nginx
sudo apt install -y nginx

# Configure Nginx
sudo tee /etc/nginx/sites-available/questbee > /dev/null << 'EOF'
upstream questbee_api {
    server localhost:8000;
}

upstream questbee_web {
    server localhost:3000;
}

server {
    listen 80;
    server_name your-domain.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name your-domain.com;

    ssl_certificate /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    location /api/ {
        proxy_pass http://questbee_api;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    location / {
        proxy_pass http://questbee_web;
        proxy_set_header Host $host;
    }
}
EOF

# Enable site
sudo ln -s /etc/nginx/sites-available/questbee /etc/nginx/sites-enabled/

# Test and reload
sudo nginx -t
sudo systemctl reload nginx
```

### Option 2: AWS RDS + ECS (More Scalable)

#### 1. Create RDS PostgreSQL Instance
```bash
aws rds create-db-instance \
  --db-instance-identifier questbee-db \
  --db-instance-class db.t3.small \
  --engine postgres \
  --master-username postgres \
  --master-user-password YOUR_STRONG_PASSWORD \
  --allocated-storage 100 \
  --storage-type gp3 \
  --backup-retention-period 7
```

#### 2. Create ECS Cluster
```bash
aws ecs create-cluster --cluster-name questbee

# Create task definition (Fargate)
aws ecs register-task-definition --cli-input-json file://task-definition.json
```

#### 3. Create ECS Service
```bash
aws ecs create-service \
  --cluster questbee \
  --service-name questbee-api \
  --task-definition questbee-api:1 \
  --desired-count 2 \
  --launch-type FARGATE
```

---

## Azure Deployment

### Using Azure Container Instances (ACI)

#### 1. Create Resource Group
```bash
az group create \
  --name questbee-rg \
  --location eastus
```

#### 2. Create Azure Database for PostgreSQL
```bash
az postgres server create \
  --resource-group questbee-rg \
  --name questbee-db \
  --location eastus \
  --admin-user postgres \
  --admin-password YOUR_STRONG_PASSWORD
```

#### 3. Deploy API Container
```bash
az container create \
  --resource-group questbee-rg \
  --name questbee-api \
  --image your-registry/questbee-api:latest \
  --ports 8000 \
  --cpu 1 --memory 2 \
  --environment-variables \
    DATABASE_URL='postgresql://localhost/questbee' \
    ENVIRONMENT='production'
```

#### 4. Create App Service (Web Dashboard)
```bash
# Create App Service Plan
az appservice plan create \
  --name questbee-plan \
  --resource-group questbee-rg \
  --sku B2

# Create Web App
az webapp create \
  --resource-group questbee-rg \
  --plan questbee-plan \
  --name questbee-web
```

---

## On-Premise / Private Server

### Requirements
- **OS:** Ubuntu 20.04 LTS or similar
- **CPU:** 2+ cores
- **RAM:** 4GB+ (8GB recommended for production)
- **Disk:** 50GB+ (SSD recommended)
- **Network:** Static IP, firewall configured

### Setup Steps

#### 1. Prepare Server
```bash
# SSH into server
ssh user@your-server-ip

# Update system
sudo apt update && sudo apt upgrade -y

# Install Docker & Docker Compose
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
sudo usermod -aG docker $USER
sudo apt install -y docker-compose

# Create app directory
sudo mkdir -p /opt/questbee
sudo chown $USER:$USER /opt/questbee
```

#### 2. Copy Deployment Files
```bash
# From your local machine
scp -r questbee/ user@your-server-ip:/opt/questbee/
```

#### 3. Create Production Config
```bash
cd /opt/questbee

# Create .env
cat > .env << 'EOF'
DATABASE_URL=postgresql://postgres:STRONG_PASSWORD_HERE@db:5432/questbee
API_SECRET_KEY=YOUR_SECRET_KEY_$(openssl rand -hex 32)
ENVIRONMENT=production
ALLOWED_HOSTS=your-server-ip,your-domain.com
DEBUG=False
EOF

# Create docker-compose.prod.yml (modify as needed)
# Use docker-compose.yml as template, but mount data directories
```

#### 4. Setup SSL Certificate
```bash
# Using self-signed cert (fast)
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -keyout /opt/questbee/ssl/key.pem \
  -out /opt/questbee/ssl/cert.pem

# Or use Let's Encrypt (recommended)
sudo apt install -y certbot
sudo certbot certonly --standalone -d your-domain.com
```

#### 5. Start Services
```bash
cd /opt/questbee

# Pull latest images
docker-compose -f docker-compose.prod.yml pull

# Start services (daemon mode)
docker-compose -f docker-compose.prod.yml up -d

# Verify
docker-compose -f docker-compose.prod.yml ps
```

#### 6. Setup Firewall
```bash
sudo ufw enable
sudo ufw allow 22/tcp   # SSH
sudo ufw allow 80/tcp   # HTTP
sudo ufw allow 443/tcp  # HTTPS
sudo ufw allow 5432/tcp # PostgreSQL (if external access needed)
```

#### 7. Backup Strategy
```bash
# Automated daily backup
cat > /usr/local/bin/questbee-backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/backups/questbee"
mkdir -p $BACKUP_DIR
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Database backup
docker exec questbee-db pg_dump -U postgres questbee | gzip > $BACKUP_DIR/db_$TIMESTAMP.sql.gz

# Retain last 30 days
find $BACKUP_DIR -name "*.gz" -mtime +30 -delete
EOF

chmod +x /usr/local/bin/questbee-backup.sh

# Add to crontab
(crontab -l 2>/dev/null; echo "0 2 * * * /usr/local/bin/questbee-backup.sh") | crontab -
```

---

## Production Checklist

- [ ] Database backups automated (daily)
- [ ] SSL/TLS certificates installed and auto-renewal configured
- [ ] Firewall configured (ports 80, 443 only exposed)
- [ ] Database password is strong and stored securely
- [ ] API secret keys rotated
- [ ] Environment variables set correctly (.env file)
- [ ] Logs are centralized and monitored
- [ ] Monitoring alerts configured (disk space, CPU, memory)
- [ ] Disaster recovery plan documented
- [ ] Performance tested under expected load

### Monitoring Setup (Optional)

**Using Prometheus + Grafana:**
```bash
# Add to docker-compose.yml
prometheus:
  image: prom/prometheus
  volumes: [prometheus.yml:/etc/prometheus/prometheus.yml]

grafana:
  image: grafana/grafana
  ports: ["3001:3000"]
  depends_on: [prometheus]
```

### Performance Optimization

```bash
# Database indexing
docker exec questbee-db psql -U postgres questbee << 'SQL'
CREATE INDEX idx_submissions_form_id ON submissions(form_id);
CREATE INDEX idx_submissions_created_at ON submissions(created_at);
SQL

# Enable PostgreSQL slow query logs
docker exec questbee-db psql -U postgres questbee << 'SQL'
ALTER SYSTEM SET log_min_duration_statement = 1000;
SELECT pg_reload_conf();
SQL
```

---

## Troubleshooting

### Services Won't Start
```bash
# Check logs
docker-compose logs -f api
docker-compose logs -f db

# Verify network
docker network ls
docker network inspect questbee_default
```

### Database Connection Error
```bash
# Test connection
docker exec questbee-db psql -U postgres -c "SELECT 1"

# Check environment variables
docker exec questbee-api printenv | grep DATABASE
```

### Disk Space Issues
```bash
# Check usage
df -h

# Clean up old containers/images  
docker system prune -a

# Check database size
docker exec questbee-db psql -U postgres -c "SELECT pg_size_pretty(pg_database_size('questbee'));"
```

---

For support, contact: **support@questbee.io**
