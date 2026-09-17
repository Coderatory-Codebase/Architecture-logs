# Document 11 — Deployment

> Source: `Yango_Clone_Architecture.docx`

## 11.1 Infrastructure Overview

```
Internet
  ↓
Nginx (Reverse Proxy + SSL + WebSocket)
  ├── Backend API (Node.js — PM2 fork mode, single instance)
  ├── Socket.io (same process as API)
  └── Admin Panel (Next.js)

Databases (V1: same VPS / V2: dedicated):
  ├── MongoDB
  ├── PostgreSQL
  └── Redis

External:
  ├── AWS S3 (driver documents)
  ├── Google Maps API
  ├── Twilio (OTP)
  └── FCM (push notifications)
```

## 11.2 V1 Server Spec

| Component | Spec | Notes |
|---|---|---|
| VPS | 4 vCPU, 8GB RAM, 100GB SSD | Handles 500 concurrent rides + margin |
| OS | Ubuntu 22.04 LTS | LTS for stability |
| Node.js | v20 LTS | Active LTS |
| Process Manager | PM2 fork mode | NOT cluster — Socket.io compatibility |
| Web Server | Nginx | Reverse proxy + SSL termination |
| SSL | Let's Encrypt (Certbot) | Auto-renew |

> ⚠️ **PM2 Configuration — Fork Mode Required** — In `ecosystem.config.js`: set `instances: 1` and `exec_mode: 'fork'`. Do NOT set `exec_mode: 'cluster'` — this breaks Socket.io without Redis Adapter. Only switch to cluster mode after Redis Adapter is added.

## 11.3 Docker Compose

```yaml
version: "3.8"
services:
  api:
    build: .
    ports:
      - "3000:3000"
    env_file: .env
    restart: unless-stopped
    depends_on:
      - mongo
      - postgres
      - redis

  mongo:
    image: mongo:7
    restart: unless-stopped
    volumes:
      - mongo_data:/data/db

  postgres:
    image: postgres:16
    restart: unless-stopped
    volumes:
      - pg_data:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    restart: unless-stopped
    command: redis-server --requirepass ${REDIS_PASSWORD} --appendonly yes
    volumes:
      - redis_data:/data

volumes:
  mongo_data:
  pg_data:
  redis_data:
```

> 💡 **Redis Persistence — AOF Enabled** — Redis command includes `--appendonly yes` — enables AOF persistence. On crash/restart, Redis recovers OTPs and matching state from AOF log. Acceptable data loss window: seconds only.

> ⚠️ **Docker vs PM2 — Choose One** — Using both Docker containers AND PM2 creates redundant restart management. **Decision:** Use Docker with `restart: unless-stopped` for container management. Remove PM2 from inside the Docker container — Docker handles restarts. PM2 only if deploying directly on host without Docker.

## 11.4 Nginx Configuration

```nginx
server {
    listen 443 ssl;
    server_name api.yango-clone.com;

    ssl_certificate     /etc/letsencrypt/live/api.yango-clone.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yango-clone.com/privkey.pem;

    # REST API
    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }

    # WebSocket — CRITICAL: extended timeout
    location /socket.io/ {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_read_timeout 86400;    # 24 hours — prevents WS cut at 60s
        proxy_send_timeout 86400;
    }
}

# Redirect HTTP to HTTPS
server {
    listen 80;
    server_name api.yango-clone.com;
    return 301 https://$host$request_uri;
}
```

## 11.5 Environments

| Environment | Purpose | File |
|---|---|---|
| Development | Local machine | .env.development |
| Staging | Pre-production testing | .env.staging |
| Production | Live | .env.production |

## 11.6 CI/CD Pipeline

```yaml
# .github/workflows/deploy.yml

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 20

      - run: npm ci
      - run: npm run lint
      - run: npm run test
      - run: npm run build

      # Load test gate — 500 concurrent rides
      # - run: npx k6 run tests/load/500-rides.js

      - name: Deploy to VPS
        uses: appleboy/ssh-action@v0.1.10
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            cd /app/yango-clone
            git pull origin main
            npm ci --production
            npm run migrate:prod
            docker compose up -d --build api
```

## 11.7 Monitoring & Alerts

| Tool | Purpose | Alert Condition |
|---|---|---|
| UptimeRobot | HTTP uptime check | API down → immediate SMS + email |
| Sentry | Error tracking | Error rate > 5% → alert |
| MongoDB Atlas | DB monitoring | Slow queries, disk > 80% |
| Nginx logs | Request logs | Manual review / pipe to log aggregator |
| Docker stats | Container resources | Memory > 85% → alert |

## 11.8 Backup Strategy

| Data | Method | Frequency | Destination |
|---|---|---|---|
| MongoDB | mongodump → gzip | Daily 2AM | S3 / Backblaze |
| PostgreSQL | pg_dump → gzip | Daily 2AM | S3 / Backblaze |
| .env files | Encrypted | On every change | Secure vault (1Password / Bitwarden) |
| Redis | AOF file backup | Daily | S3 / Backblaze |

> ⚠️ **Backup Restore Drill Required** — A backup never tested is not a backup. Before launch: restore MongoDB dump to staging, verify data integrity, document the restore procedure. Schedule quarterly restore drills in production prep.

## 11.9 Scale-Up Plan

| Trigger | Action |
|---|---|
| Concurrent rides > 1000 | Upgrade VPS to 8 vCPU 16GB |
| Concurrent rides > 2000 | Add Redis Adapter, switch PM2 to cluster mode |
| Concurrent rides > 5000 | Separate DB server, dedicated Socket.io instance |
| Disk > 80% | Expand VPS storage or archive old ride data |
| Monthly ride data > 6 months old | Archive to cold storage (S3 Glacier) |

## 11.10 2-Year Data Retention

```
// Monthly archival job (cron)
// Archive rides older than 2 years to cold storage
// Run: 1st of every month at 3AM

0 3 1 * * node scripts/archive-old-rides.js

// Script:
// 1. Find rides where completedAt < NOW - 2 years
// 2. Export to JSON → upload to S3 Glacier
// 3. Delete from MongoDB
// 4. Log archive job result
```

---
◀ Previous: [10 — Base Server Mapping](10-base-server-mapping.md) | Back to [README](../README.md) | Next → [12 — Issues Fixed Review Summary](12-issues-fixed-review-summary.md)
