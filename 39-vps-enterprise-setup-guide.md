# Enterprise-Grade VPS Setup Guide — Hostinger KVM 8 (Ubuntu 24.04 LTS)

**Target Deployment:**
- 4–5 Node.js / Next.js applications
- Relational + NoSQL databases
- Redis caching + BullMQ job queues
- Multiple message brokers (RabbitMQ, Kafka, NATS)
- WebSockets, SSE, long-polling
- Third-party integrations (Google Maps, Firebase, Analytics, Payments, Email, SMS)
- Full observability (Prometheus + Grafana + Loki + Sentry)
- GitHub Actions CI/CD with zero-downtime deploys

**Base OS:** Ubuntu 24.04 LTS (Noble Numbat) — supported until April 2029 (extended with Ubuntu Pro).

---

## Latest Versions Reference (as of September 2026)

| Component | Version | Source |
|---|---|---|
| Ubuntu | 24.04 LTS | ubuntu.com |
| Node.js | 22 LTS (Maintenance) / 24 LTS (Active) | nodejs.org |
| PostgreSQL | 18.x | postgresql.org (PGDG apt) |
| MongoDB | 8.x Community | mongodb.com |
| MySQL / MariaDB | 8.4 / 11.4 LTS | official repos |
| Redis | 8.x (Open Source) | packages.redis.io |
| RabbitMQ | 4.3.x | rabbitmq.com |
| Apache Kafka | 4.2.x (KRaft mode, no ZooKeeper) | kafka.apache.org |
| NATS Server | 2.11+ with JetStream | nats.io |
| Elasticsearch | 8.x / OpenSearch 2.x | elastic.co / opensearch.org |
| MeiliSearch | 1.11+ | meilisearch.com |
| MinIO | Latest RELEASE | min.io |
| Nginx | 1.24+ (Ubuntu stable) / 1.27 (mainline) | nginx.org |
| Caddy | 2.8+ | caddyserver.com |
| PM2 | Latest via npm | pm2.keymetrics.io |
| Docker CE | Latest | docs.docker.com |
| Docker Compose | v2 (plugin, `docker compose`) | docs.docker.com |
| Certbot | Latest via snap | certbot.eff.org |
| Prometheus | 2.55+ | prometheus.io |
| Grafana | 11.x | grafana.com |
| Loki | 3.x | grafana.com/oss/loki |
| Java (for Kafka) | OpenJDK 21 LTS | openjdk.org |

**Node.js recommendation:** Use **Node 24 LTS** for new projects. Node 22 for extra stability. Avoid Node 26 (currently Current, becomes LTS in October 2026).

---

## Table of Contents

**Part A — Foundation**
1. Initial Login & System Update
2. Hostname, Timezone, Swap, Kernel Tuning
3. Non-root Sudo User
4. SSH Hardening (Key-based Auth)
5. UFW Firewall
6. Fail2Ban / CrowdSec
7. Unattended Security Updates
8. ClamAV Malware Scanning (optional)

**Part B — Runtime Stack**
9. Node.js via NVM
10. PM2 Process Manager
11. Nginx Reverse Proxy
12. SSL with Certbot / Let's Encrypt
13. Docker & Docker Compose

**Part C — Data Layer**
14. PostgreSQL 18
15. MongoDB 8
16. MySQL / MariaDB (alternative)
17. Redis 8
18. MinIO (S3-compatible object storage)

**Part D — Search Engines**
19. MeiliSearch (lightweight, recommended for most apps)
20. Elasticsearch / OpenSearch

**Part E — Message Brokers & Queues**
21. RabbitMQ 4.x
22. Apache Kafka 4.x (KRaft mode)
23. NATS + JetStream
24. BullMQ (Redis-backed job queue for Node.js)
25. Broker Selection Guide (which one to use when)

**Part F — Application Deployment**
26. Multi-App Hosting Strategy
27. Node.js / Express App Deployment
28. Next.js App Deployment
29. WebSocket / SSE / Long-polling Nginx Config

**Part G — Third-Party Integrations**
30. Google Maps / Places / Geocoding APIs
31. Firebase (Auth, Cloud Messaging, Analytics, Firestore)
32. Google Analytics 4 (GA4)
33. PostHog / Mixpanel (product analytics)
34. Email Providers (Resend / SendGrid / Postmark / Mailgun)
35. SMS & WhatsApp (Twilio / MSG91 / Gupshup)
36. Payments (Stripe / Razorpay)
37. Cloudflare (CDN + DNS + WAF + Turnstile)
38. Sentry (error tracking + APM)
39. OAuth Providers (Google, Apple, GitHub)

**Part H — Observability**
40. Prometheus + Grafana + node_exporter
41. Loki (log aggregation)
42. Netdata (quick monitoring)
43. Uptime Kuma (self-hosted uptime monitoring)
44. OpenTelemetry (traces + metrics + logs)

**Part I — Ops & Automation**
45. Environment / Secrets Management
46. Backups (Databases + Files + Offsite)
47. Log Rotation
48. GitHub Actions CI/CD (SSH-based)
49. GitHub Actions CI/CD (Self-hosted Runner)
50. GitHub Actions CI/CD (Docker + Zero-Downtime)
51. Staging + Production Environments

**Part J — Security & Compliance**
52. API Key & Secrets Storage (Doppler / Vault)
53. Security Audit Checklist
54. Rate Limiting & DDoS Mitigation
55. Compliance Notes (GDPR / DPDP India)

**Part K — Reference**
56. Troubleshooting Common Issues
57. Command Cheat Sheet
58. Final Production Checklist

---

# PART A — FOUNDATION

## 1. Initial Login & System Update

From your local Windows terminal (Windows Terminal + built-in OpenSSH, WSL2, or PuTTY):

```bash
ssh root@YOUR_VPS_IP
```

Accept the fingerprint on first login. Then immediately:

```bash
apt update && apt upgrade -y
apt autoremove -y
apt install -y curl wget gnupg lsb-release ca-certificates \
               software-properties-common apt-transport-https \
               build-essential git htop net-tools ufw \
               unzip zip jq nano vim tmux
reboot
```

Wait 30 seconds, reconnect: `ssh root@YOUR_VPS_IP`.

---

## 2. Hostname, Timezone, Swap, Kernel Tuning

### 2.1 Hostname

```bash
hostnamectl set-hostname prod-server-01
```

Edit `/etc/hosts`:
```bash
nano /etc/hosts
```
Add: `127.0.1.1   prod-server-01`

### 2.2 Timezone

```bash
timedatectl set-timezone Asia/Kolkata     # or UTC — pick one and be consistent
timedatectl
```

### 2.3 Swap File

For a KVM 8 VPS (32 GB RAM typical on Hostinger's KVM 8), 4–8 GB swap is enough:

```bash
fallocate -l 8G /swapfile
chmod 600 /swapfile
mkswap /swapfile
swapon /swapfile
echo '/swapfile none swap sw 0 0' | tee -a /etc/fstab
sysctl vm.swappiness=10
echo 'vm.swappiness=10' > /etc/sysctl.d/99-swappiness.conf
free -h
```

### 2.4 Kernel Tuning for High-Concurrency Node.js

Create `/etc/sysctl.d/99-network-tuning.conf`:

```bash
cat > /etc/sysctl.d/99-network-tuning.conf << 'EOF'
# Increase file descriptor limit
fs.file-max = 2097152

# Network performance
net.core.somaxconn = 65535
net.core.netdev_max_backlog = 5000
net.ipv4.tcp_max_syn_backlog = 8192
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_fin_timeout = 15
net.ipv4.tcp_keepalive_time = 300
net.ipv4.ip_local_port_range = 1024 65535

# Prevent SYN flood attacks
net.ipv4.tcp_syncookies = 1

# Increase TCP buffer sizes
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_rmem = 4096 87380 16777216
net.ipv4.tcp_wmem = 4096 65536 16777216
EOF

sysctl -p /etc/sysctl.d/99-network-tuning.conf
```

Increase per-process file descriptor limits — edit `/etc/security/limits.conf`:

```
*   soft   nofile   65535
*   hard   nofile   65535
root soft   nofile   65535
root hard   nofile   65535
```

Log out and back in for these to apply.

---

## 3. Non-root Sudo User

**Never operate as root.** Create a dedicated user:

```bash
adduser deploy
usermod -aG sudo deploy
```

Test in a **new terminal window** (keep old one open as backup):

```bash
ssh deploy@YOUR_VPS_IP
sudo whoami        # Should print "root"
```

---

## 4. SSH Hardening (Key-based Auth)

### 4.1 Generate a key pair on your local machine

```powershell
# On Windows PowerShell
ssh-keygen -t ed25519 -C "your_email@example.com"
```

### 4.2 Copy public key to VPS

```powershell
type $env:USERPROFILE\.ssh\id_ed25519.pub | ssh deploy@YOUR_VPS_IP "mkdir -p ~/.ssh && chmod 700 ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys"
```

Test: `ssh deploy@YOUR_VPS_IP` — no password prompt.

### 4.3 Harden SSH server config

```bash
sudo nano /etc/ssh/sshd_config
```

Set these values (edit or uncomment):

```
Port 22                                # Optionally change to a non-standard port
PermitRootLogin no
PasswordAuthentication no
PubkeyAuthentication yes
KbdInteractiveAuthentication no
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding no
ClientAliveInterval 300
ClientAliveCountMax 2
MaxAuthTries 3
MaxSessions 4
AllowUsers deploy
Protocol 2
```

Reload:

```bash
sudo sshd -t                # syntax check first
sudo systemctl reload ssh
```

Test in a **new window** before closing the current session.

---

## 5. UFW Firewall

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing

sudo ufw allow 22/tcp        # SSH (or your custom port)
sudo ufw allow 80/tcp        # HTTP
sudo ufw allow 443/tcp       # HTTPS

sudo ufw enable
sudo ufw status verbose
```

**Never expose database/broker ports publicly.** All internal services (5432, 6379, 5672, 9092, 4222, 27017) bind to `127.0.0.1` and stay behind UFW.

If a remote IP genuinely needs DB access:
```bash
sudo ufw allow from 203.0.113.5 to any port 5432 proto tcp
```

---

## 6. Fail2Ban / CrowdSec

### 6.1 Fail2Ban (traditional, well-documented)

```bash
sudo apt install fail2ban -y
sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
sudo nano /etc/fail2ban/jail.local
```

Add/edit `[sshd]`:

```ini
[sshd]
enabled = true
port    = 22
filter  = sshd
logpath = %(sshd_log)s
maxretry = 3
bantime  = 3600
findtime = 600
```

Also enable `[nginx-http-auth]` and `[nginx-limit-req]` if you use those Nginx features.

```bash
sudo systemctl enable --now fail2ban
sudo fail2ban-client status sshd
```

### 6.2 CrowdSec (modern alternative — collaborative threat intelligence)

```bash
curl -s https://install.crowdsec.net | sudo sh
sudo apt install crowdsec crowdsec-firewall-bouncer-iptables -y
sudo systemctl enable --now crowdsec

# Enroll (optional, gets you the community blocklists)
sudo cscli console enroll <your-enrollment-key>

# Add nginx collection
sudo cscli collections install crowdsecurity/nginx
sudo systemctl restart crowdsec
```

Pick one — Fail2Ban is simpler; CrowdSec is more powerful.

---

## 7. Unattended Security Updates

```bash
sudo apt install unattended-upgrades -y
sudo dpkg-reconfigure --priority=low unattended-upgrades
sudo nano /etc/apt/apt.conf.d/50unattended-upgrades
```

Enable:
```
Unattended-Upgrade::Remove-Unused-Dependencies "true";
Unattended-Upgrade::Automatic-Reboot "false";
Unattended-Upgrade::Automatic-Reboot-Time "04:00";
```

Verify:
```bash
sudo unattended-upgrade --dry-run --debug
```

---

## 8. ClamAV Malware Scanning (Optional but Recommended)

Useful when your app accepts file uploads.

```bash
sudo apt install clamav clamav-daemon -y
sudo systemctl stop clamav-freshclam
sudo freshclam        # Update signatures
sudo systemctl start clamav-freshclam
sudo systemctl enable --now clamav-daemon
```

Scan uploads on receipt using `clamdscan` from your Node.js app (via `clamscan` npm package).

---

# PART B — RUNTIME STACK

## 9. Node.js via NVM

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc

nvm install 24                # Active LTS (recommended)
nvm alias default 24
nvm install 22                # Optional: keep LTS Maintenance too

node -v
npm -v

# Global tools
npm install -g pm2 yarn pnpm typescript ts-node nodemon
```

---

## 10. PM2 Process Manager

### 10.1 Enable startup on boot

```bash
pm2 startup systemd
# It prints a sudo command — copy-paste and execute it exactly
```

### 10.2 Ecosystem file (single source of truth)

Create `~/apps/ecosystem.config.js`:

```javascript
module.exports = {
  apps: [
    {
      name: 'api-main',
      cwd: '/home/deploy/apps/api-main',
      script: 'dist/server.js',
      instances: 'max',              // Use all CPU cores (cluster mode)
      exec_mode: 'cluster',
      env: {
        NODE_ENV: 'production',
        PORT: 3001
      },
      max_memory_restart: '600M',
      error_file: '/home/deploy/logs/api-main-error.log',
      out_file: '/home/deploy/logs/api-main-out.log',
      time: true,
      wait_ready: true,              // App must call process.send('ready')
      listen_timeout: 10000,
      kill_timeout: 5000
    },
    {
      name: 'api-auth',
      cwd: '/home/deploy/apps/api-auth',
      script: 'dist/server.js',
      instances: 2,
      exec_mode: 'cluster',
      env: { NODE_ENV: 'production', PORT: 3002 }
    },
    {
      name: 'worker-queue',
      cwd: '/home/deploy/apps/worker-queue',
      script: 'dist/worker.js',
      instances: 1,
      exec_mode: 'fork',
      env: { NODE_ENV: 'production' }
    },
    {
      name: 'web-marketing',
      cwd: '/home/deploy/apps/web-marketing',
      script: 'npm',
      args: 'start',
      env: { NODE_ENV: 'production', PORT: 3010 }
    },
    {
      name: 'web-dashboard',
      cwd: '/home/deploy/apps/web-dashboard',
      script: 'npm',
      args: 'start',
      env: { NODE_ENV: 'production', PORT: 3011 }
    }
  ]
};
```

### 10.3 Operating PM2

```bash
mkdir -p ~/apps ~/logs

pm2 start ~/apps/ecosystem.config.js
pm2 save                           # persist for reboot restore
pm2 status
pm2 logs api-main --lines 200
pm2 monit                          # real-time TUI dashboard
pm2 reload api-main                # zero-downtime restart (cluster mode)
pm2 restart api-main               # hard restart
pm2 describe api-main
pm2 flush                          # clear all logs
```

### 10.4 PM2 log rotation

```bash
pm2 install pm2-logrotate
pm2 set pm2-logrotate:max_size 50M
pm2 set pm2-logrotate:retain 14
pm2 set pm2-logrotate:compress true
pm2 set pm2-logrotate:rotateInterval '0 0 * * *'
```

---

## 11. Nginx Reverse Proxy

```bash
sudo apt install nginx -y
sudo systemctl enable --now nginx
sudo rm /etc/nginx/sites-enabled/default
```

### 11.1 Global hardening

Edit `/etc/nginx/nginx.conf`, inside `http { }`:

```nginx
server_tokens off;
client_max_body_size 50M;
client_body_timeout 60s;
client_header_timeout 60s;
keepalive_timeout 65;
send_timeout 60s;

gzip on;
gzip_vary on;
gzip_min_length 1024;
gzip_types text/plain text/css application/json application/javascript
           text/xml application/xml application/xml+rss text/javascript
           application/wasm image/svg+xml;

# Basic rate limiting zone
limit_req_zone $binary_remote_addr zone=api_limit:10m rate=30r/s;
limit_conn_zone $binary_remote_addr zone=conn_limit:10m;

# Security headers (applied per-site too)
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header Referrer-Policy "strict-origin-when-cross-origin" always;
```

### 11.2 Per-site config (reverse proxy to Node app)

`/etc/nginx/sites-available/api.example.com`:

```nginx
upstream api_backend {
    server 127.0.0.1:3001;
    keepalive 32;
}

server {
    listen 80;
    server_name api.example.com;

    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }

    location / {
        limit_req zone=api_limit burst=50 nodelay;
        limit_conn conn_limit 20;

        proxy_pass http://api_backend;
        proxy_http_version 1.1;

        # Headers
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Forwarded-Host $host;

        # WebSocket support
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";

        # Timeouts
        proxy_connect_timeout 60s;
        proxy_read_timeout 300s;
        proxy_send_timeout 300s;

        # Buffering
        proxy_buffering on;
        proxy_buffer_size 8k;
        proxy_buffers 8 8k;
    }
}
```

Enable:
```bash
sudo ln -s /etc/nginx/sites-available/api.example.com /etc/nginx/sites-enabled/
sudo nginx -t && sudo systemctl reload nginx
```

---

## 12. SSL with Certbot / Let's Encrypt

```bash
sudo snap install core && sudo snap refresh core
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/bin/certbot
```

Ensure DNS A records point to your VPS IP before running:

```bash
sudo certbot --nginx -d api.example.com
sudo certbot --nginx -d www.example.com -d example.com
sudo certbot --nginx -d dashboard.example.com
```

Test auto-renewal:
```bash
sudo certbot renew --dry-run
sudo systemctl list-timers | grep certbot
```

### 12.1 Wildcard certificates (DNS-01 challenge)

For a wildcard cert (`*.example.com`), you need DNS provider access. Example with Cloudflare:

```bash
sudo snap install certbot-dns-cloudflare
sudo mkdir /root/.secrets
echo "dns_cloudflare_api_token = YOUR_TOKEN" | sudo tee /root/.secrets/cloudflare.ini
sudo chmod 600 /root/.secrets/cloudflare.ini

sudo certbot certonly \
  --dns-cloudflare \
  --dns-cloudflare-credentials /root/.secrets/cloudflare.ini \
  -d '*.example.com' -d 'example.com'
```

---

## 13. Docker & Docker Compose

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] \
  https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

sudo usermod -aG docker deploy
newgrp docker
docker --version && docker compose version
docker run hello-world
```

**Strategy decision:** You have two valid paths for infra services (Postgres, Redis, RabbitMQ, Kafka, etc.):

- **Native install** (via apt) — better for single-VPS setups, simpler backups, systemd-managed.
- **Docker Compose** — better isolation, easier version upgrades, portability.

This guide shows both. Pick one style and stick with it per service.

---

# PART C — DATA LAYER

## 14. PostgreSQL 18

### 14.1 Install via official PGDG repo

```bash
sudo install -d /usr/share/postgresql-common/pgdg
sudo curl -o /usr/share/postgresql-common/pgdg/apt.postgresql.org.asc \
  --fail https://www.postgresql.org/media/keys/ACCC4CF8.asc

sudo sh -c 'echo "deb [signed-by=/usr/share/postgresql-common/pgdg/apt.postgresql.org.asc] \
  https://apt.postgresql.org/pub/repos/apt $(lsb_release -cs)-pgdg main" \
  > /etc/apt/sources.list.d/pgdg.list'

sudo apt update
sudo apt install postgresql-18 postgresql-contrib-18 -y

sudo systemctl enable --now postgresql
psql --version
```

### 14.2 Create app users and databases

```bash
sudo -u postgres psql
```

```sql
CREATE USER app_main WITH PASSWORD 'strong_password_here';
CREATE DATABASE app_main_db OWNER app_main;
GRANT ALL PRIVILEGES ON DATABASE app_main_db TO app_main;

CREATE USER app_auth WITH PASSWORD 'another_strong_password';
CREATE DATABASE app_auth_db OWNER app_auth;
GRANT ALL PRIVILEGES ON DATABASE app_auth_db TO app_auth;

\q
```

### 14.3 Security config

`/etc/postgresql/18/main/postgresql.conf`:

```
listen_addresses = 'localhost'         # Never bind to 0.0.0.0 unless required
```

`/etc/postgresql/18/main/pg_hba.conf`:

```
local   all             all                                     peer
host    all             all             127.0.0.1/32            scram-sha-256
host    all             all             ::1/128                 scram-sha-256
```

Reload:
```bash
sudo systemctl reload postgresql
```

### 14.4 Performance tuning (for KVM 8 — assume 32 GB RAM, adjust proportionally)

```
shared_buffers = 8GB                   # ~25% of RAM
effective_cache_size = 24GB            # ~75% of RAM
work_mem = 32MB
maintenance_work_mem = 1GB
max_connections = 200
wal_buffers = 16MB
random_page_cost = 1.1                 # SSD
effective_io_concurrency = 200         # SSD
max_worker_processes = 8
max_parallel_workers_per_gather = 4
max_parallel_workers = 8
```

Use https://pgtune.leopard.in.ua/ for accurate values per your specs. Restart after changes:
```bash
sudo systemctl restart postgresql
```

### 14.5 Node connection

```
postgresql://app_main:PASSWORD@127.0.0.1:5432/app_main_db
```

Use `pg`, `postgres`, or an ORM (Prisma, Drizzle, TypeORM).

### 14.6 Useful extensions

```sql
-- In your database
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
CREATE EXTENSION IF NOT EXISTS "pgcrypto";
CREATE EXTENSION IF NOT EXISTS "pg_trgm";       -- fuzzy text search
CREATE EXTENSION IF NOT EXISTS "citext";        -- case-insensitive text
CREATE EXTENSION IF NOT EXISTS "postgis";       -- geographic queries (needs postgresql-18-postgis-3 apt package)
```

---

## 15. MongoDB 8

### 15.1 Install via official MongoDB repo

```bash
curl -fsSL https://www.mongodb.org/static/pgp/server-8.0.asc | \
  sudo gpg -o /usr/share/keyrings/mongodb-server-8.0.gpg --dearmor

echo "deb [ arch=amd64,arm64 signed-by=/usr/share/keyrings/mongodb-server-8.0.gpg ] \
  https://repo.mongodb.org/apt/ubuntu noble/mongodb-org/8.0 multiverse" | \
  sudo tee /etc/apt/sources.list.d/mongodb-org-8.0.list

sudo apt update
sudo apt install mongodb-org -y

sudo systemctl enable --now mongod
mongod --version
```

### 15.2 Enable authentication

Access shell:
```bash
mongosh
```

Create admin:
```javascript
use admin
db.createUser({
  user: "mongo_admin",
  pwd: "strong_admin_password",
  roles: [{ role: "userAdminAnyDatabase", db: "admin" },
          { role: "readWriteAnyDatabase", db: "admin" },
          { role: "dbAdminAnyDatabase", db: "admin" }]
})
exit
```

Enable auth in `/etc/mongod.conf`:
```yaml
security:
  authorization: enabled
net:
  bindIp: 127.0.0.1
  port: 27017
```

Restart:
```bash
sudo systemctl restart mongod
```

Create app-specific user:
```bash
mongosh -u mongo_admin -p --authenticationDatabase admin
```
```javascript
use myapp
db.createUser({
  user: "app_user",
  pwd: "strong_app_password",
  roles: [{ role: "readWrite", db: "myapp" }]
})
```

Node connection:
```
mongodb://app_user:PASSWORD@127.0.0.1:27017/myapp?authSource=myapp
```

Use `mongoose` (ODM) or the native `mongodb` driver.

---

## 16. MySQL / MariaDB (Alternative RDBMS)

If you prefer MySQL over PostgreSQL:

```bash
# MySQL 8.4 LTS
sudo apt install mysql-server -y
sudo systemctl enable --now mysql
sudo mysql_secure_installation
```

Or MariaDB 11.4 LTS:
```bash
sudo apt install mariadb-server -y
sudo systemctl enable --now mariadb
sudo mariadb-secure-installation
```

Create user + DB:
```bash
sudo mysql -u root -p
```
```sql
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'app_user'@'localhost' IDENTIFIED BY 'strong_password';
GRANT ALL PRIVILEGES ON myapp.* TO 'app_user'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

Node driver: `mysql2` (better performance than `mysql`).

---

## 17. Redis 8

```bash
curl -fsSL https://packages.redis.io/gpg | sudo gpg --dearmor -o /usr/share/keyrings/redis-archive-keyring.gpg
sudo chmod 644 /usr/share/keyrings/redis-archive-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/redis-archive-keyring.gpg] \
  https://packages.redis.io/deb $(lsb_release -cs) main" | \
  sudo tee /etc/apt/sources.list.d/redis.list

sudo apt update
sudo apt install redis -y

sudo systemctl enable --now redis-server
```

Edit `/etc/redis/redis.conf`:

```
bind 127.0.0.1 ::1
protected-mode yes
port 6379
requirepass STRONG_REDIS_PASSWORD

appendonly yes
appendfsync everysec

maxmemory 4gb
maxmemory-policy allkeys-lru

slowlog-log-slower-than 10000
slowlog-max-len 128

# For BullMQ / job queues, allow more clients
maxclients 10000
```

Restart:
```bash
sudo systemctl restart redis-server
```

Test:
```bash
redis-cli -a STRONG_REDIS_PASSWORD ping
# PONG
```

Node connection:
```
redis://:PASSWORD@127.0.0.1:6379
```

Use `ioredis` (best BullMQ support) or `redis` (official).

### 17.1 Multiple logical databases

Redis has 16 databases (0–15) by default. Use different DB numbers to separate concerns:
- DB 0 — cache
- DB 1 — sessions
- DB 2 — BullMQ queues
- DB 3 — pub/sub for Socket.io

Connection: `redis://:PASSWORD@127.0.0.1:6379/2`

---

## 18. MinIO (S3-Compatible Object Storage)

Self-hosted S3 for user uploads, backups, media — cheaper than AWS S3 and keeps data on your VPS.

```bash
sudo useradd -r minio-user -s /sbin/nologin

wget https://dl.min.io/server/minio/release/linux-amd64/minio_20260901000000.0.0_amd64.deb -O minio.deb
sudo dpkg -i minio.deb

sudo mkdir -p /mnt/minio-data
sudo chown minio-user:minio-user /mnt/minio-data
```

Config `/etc/default/minio`:
```
MINIO_VOLUMES="/mnt/minio-data"
MINIO_OPTS="--address :9000 --console-address :9001"
MINIO_ROOT_USER=minio_admin
MINIO_ROOT_PASSWORD=very_strong_password_min_8_chars
```

Start:
```bash
sudo systemctl enable --now minio
```

Expose via Nginx (subdomain `s3.example.com` for API, `s3-console.example.com` for UI) with SSL — same pattern as RabbitMQ management UI.

Node SDK: `@aws-sdk/client-s3` — works with MinIO by pointing endpoint to `https://s3.example.com`.

---

# PART D — SEARCH ENGINES

## 19. MeiliSearch (Lightweight, Recommended for Most Apps)

Fastest to set up, low resource use, great typo tolerance out of the box.

```bash
curl -L https://install.meilisearch.com | sh
sudo mv ./meilisearch /usr/local/bin/

# Create systemd service
sudo useradd -d /var/lib/meilisearch -b /bin/false -m -r meilisearch
sudo mkdir -p /var/lib/meilisearch /etc/meilisearch
sudo chown -R meilisearch:meilisearch /var/lib/meilisearch
```

Config `/etc/meilisearch/config.toml`:
```toml
env = "production"
db_path = "/var/lib/meilisearch/data"
dump_dir = "/var/lib/meilisearch/dumps"
http_addr = "127.0.0.1:7700"
master_key = "very_strong_master_key_here_minimum_16_chars"
```

Systemd `/etc/systemd/system/meilisearch.service`:
```ini
[Unit]
Description=Meilisearch
After=network.target

[Service]
Type=simple
User=meilisearch
Group=meilisearch
ExecStart=/usr/local/bin/meilisearch --config-file-path=/etc/meilisearch/config.toml
Restart=on-failure
RestartSec=1

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now meilisearch
```

Node client: `meilisearch` npm package.

---

## 20. Elasticsearch / OpenSearch

Heavy but powerful — use only if you need advanced aggregations, log analysis, or Kibana-style dashboards.

### 20.1 Elasticsearch 8

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elastic-keyring.gpg

echo "deb [signed-by=/usr/share/keyrings/elastic-keyring.gpg] \
  https://artifacts.elastic.co/packages/8.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
sudo apt install elasticsearch -y
sudo systemctl enable --now elasticsearch
```

Config `/etc/elasticsearch/elasticsearch.yml`:
```yaml
network.host: 127.0.0.1
http.port: 9200
xpack.security.enabled: true
```

**Memory:** Elasticsearch needs at least 4 GB heap. Edit `/etc/elasticsearch/jvm.options.d/heap.options`:
```
-Xms4g
-Xmx4g
```

Node client: `@elastic/elasticsearch`.

---

# PART E — MESSAGE BROKERS & QUEUES

## 21. RabbitMQ 4.x

Best for: reliable task queues, RPC patterns, complex routing (topic exchanges, headers exchanges), microservices communication.

### 21.1 Install via official repo

```bash
sudo apt install curl gnupg apt-transport-https -y

# Signing keys
curl -1sLf "https://keys.openpgp.org/vks/v1/by-fingerprint/0A9AF2115F4687BD29803A206B73A36E6026DFCA" | \
    sudo gpg --dearmor -o /usr/share/keyrings/com.rabbitmq.team.gpg

curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-erlang.E495BB49CC4BBE5B.key | \
    sudo gpg --dearmor -o /usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg

curl -1sLf https://github.com/rabbitmq/signing-keys/releases/download/3.0/cloudsmith.rabbitmq-server.9F4587F226208342.key | \
    sudo gpg --dearmor -o /usr/share/keyrings/rabbitmq.9F4587F226208342.gpg

sudo tee /etc/apt/sources.list.d/rabbitmq.list <<EOF
deb [arch=amd64 signed-by=/usr/share/keyrings/rabbitmq.E495BB49CC4BBE5B.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-erlang/deb/ubuntu noble main
deb [arch=amd64 signed-by=/usr/share/keyrings/rabbitmq.9F4587F226208342.gpg] https://ppa1.novemberain.com/rabbitmq/rabbitmq-server/deb/ubuntu noble main
EOF

sudo apt update
sudo apt install -y erlang-base erlang-asn1 erlang-crypto erlang-eldap erlang-ftp \
                    erlang-inets erlang-mnesia erlang-os-mon erlang-parsetools \
                    erlang-public-key erlang-runtime-tools erlang-snmp erlang-ssl \
                    erlang-syntax-tools erlang-tftp erlang-tools erlang-xmerl

sudo apt install rabbitmq-server -y --fix-missing
sudo systemctl enable --now rabbitmq-server
```

### 21.2 Secure and set up users

```bash
sudo rabbitmq-plugins enable rabbitmq_management

sudo rabbitmqctl add_user admin STRONG_ADMIN_PASSWORD
sudo rabbitmqctl set_user_tags admin administrator
sudo rabbitmqctl set_permissions -p / admin ".*" ".*" ".*"
sudo rabbitmqctl delete_user guest

# App-specific vhost + user
sudo rabbitmqctl add_vhost app_vhost
sudo rabbitmqctl add_user app_mq STRONG_APP_PASSWORD
sudo rabbitmqctl set_permissions -p app_vhost app_mq ".*" ".*" ".*"
```

Bind to localhost — edit `/etc/rabbitmq/rabbitmq.conf`:
```
listeners.tcp.default = 127.0.0.1:5672
management.tcp.ip = 127.0.0.1
management.tcp.port = 15672
```

Restart:
```bash
sudo systemctl restart rabbitmq-server
```

Node client: `amqplib` or higher-level `rascal`.

---

## 22. Apache Kafka 4.x (KRaft Mode)

Best for: high-throughput event streaming, event sourcing, CDC pipelines, log aggregation, replayable event logs.

Kafka 4.x uses **KRaft** (no ZooKeeper needed) — much simpler.

### 22.1 Install Java 21

```bash
sudo apt install openjdk-21-jdk-headless -y
java -version

echo 'export JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64' | sudo tee /etc/profile.d/java.sh
source /etc/profile.d/java.sh
```

### 22.2 Install Kafka

```bash
sudo useradd -r -s /usr/sbin/nologin kafka

cd /tmp
KAFKA_VERSION=4.2.0    # check https://kafka.apache.org/downloads for latest
wget https://downloads.apache.org/kafka/${KAFKA_VERSION}/kafka_2.13-${KAFKA_VERSION}.tgz

sudo tar -xzf kafka_2.13-${KAFKA_VERSION}.tgz -C /opt/
sudo mv /opt/kafka_2.13-${KAFKA_VERSION} /opt/kafka
sudo mkdir -p /var/lib/kafka/data
sudo chown -R kafka:kafka /opt/kafka /var/lib/kafka
```

### 22.3 Configure KRaft mode

Edit `/opt/kafka/config/server.properties`:
```properties
# KRaft process roles (combined broker+controller for single-node)
process.roles=broker,controller
node.id=1
controller.quorum.voters=1@localhost:9093

# Listeners
listeners=PLAINTEXT://127.0.0.1:9092,CONTROLLER://127.0.0.1:9093
inter.broker.listener.name=PLAINTEXT
advertised.listeners=PLAINTEXT://127.0.0.1:9092
controller.listener.names=CONTROLLER
listener.security.protocol.map=CONTROLLER:PLAINTEXT,PLAINTEXT:PLAINTEXT,SSL:SSL,SASL_PLAINTEXT:SASL_PLAINTEXT,SASL_SSL:SASL_SSL

# Storage
log.dirs=/var/lib/kafka/data

# Defaults for single-node
num.partitions=3
default.replication.factor=1
min.insync.replicas=1
offsets.topic.replication.factor=1
transaction.state.log.replication.factor=1
transaction.state.log.min.isr=1

# Retention
log.retention.hours=168        # 7 days
log.segment.bytes=1073741824
log.retention.check.interval.ms=300000

# Performance
num.network.threads=3
num.io.threads=8
socket.send.buffer.bytes=102400
socket.receive.buffer.bytes=102400
socket.request.max.bytes=104857600
```

### 22.4 Initialize storage

```bash
sudo -u kafka /opt/kafka/bin/kafka-storage.sh random-uuid
# copy the UUID output, e.g. 5L6g3nShT-eMCtK--X86sw

sudo -u kafka /opt/kafka/bin/kafka-storage.sh format \
  -t <PASTE-UUID-HERE> \
  -c /opt/kafka/config/server.properties
```

### 22.5 Systemd service

`/etc/systemd/system/kafka.service`:

```ini
[Unit]
Description=Apache Kafka Server (KRaft Mode)
Documentation=https://kafka.apache.org/documentation/
After=network.target

[Service]
Type=simple
User=kafka
Group=kafka
Environment="JAVA_HOME=/usr/lib/jvm/java-21-openjdk-amd64"
Environment="KAFKA_HEAP_OPTS=-Xmx2G -Xms2G"
ExecStart=/opt/kafka/bin/kafka-server-start.sh /opt/kafka/config/server.properties
ExecStop=/opt/kafka/bin/kafka-server-stop.sh
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now kafka
sudo systemctl status kafka
```

### 22.6 Test

```bash
# Create topic
/opt/kafka/bin/kafka-topics.sh --create --topic test-events \
  --partitions 3 --replication-factor 1 \
  --bootstrap-server localhost:9092

# List
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092

# Producer (in one terminal)
/opt/kafka/bin/kafka-console-producer.sh --topic test-events \
  --bootstrap-server localhost:9092

# Consumer (in another terminal)
/opt/kafka/bin/kafka-console-consumer.sh --topic test-events --from-beginning \
  --bootstrap-server localhost:9092
```

### 22.7 Node client

Use `kafkajs`:
```javascript
import { Kafka } from 'kafkajs';

const kafka = new Kafka({
  clientId: 'my-app',
  brokers: ['127.0.0.1:9092']
});

const producer = kafka.producer();
await producer.connect();
await producer.send({
  topic: 'test-events',
  messages: [{ value: JSON.stringify({ hello: 'world' }) }]
});
```

### 22.8 Enable SASL authentication (production)

For production, enable SASL/SCRAM authentication on Kafka. Follow official docs: https://kafka.apache.org/documentation/#security_sasl

---

## 23. NATS + JetStream

Lightweight, super fast alternative to Kafka/RabbitMQ. Great for microservices and real-time messaging. **Single binary, ~20MB memory footprint.**

### 23.1 Install

```bash
cd /tmp
wget https://github.com/nats-io/nats-server/releases/latest/download/nats-server-v2.11.0-linux-amd64.tar.gz
tar -xzf nats-server-v2.11.0-linux-amd64.tar.gz
sudo mv nats-server-v2.11.0-linux-amd64/nats-server /usr/local/bin/
nats-server --version
```

### 23.2 Config `/etc/nats/nats.conf`

```
port: 4222
http_port: 8222        # monitoring UI (bind to localhost only via listen)
listen: 127.0.0.1

jetstream {
  store_dir: /var/lib/nats/jetstream
  max_memory_store: 1G
  max_file_store: 20G
}

authorization {
  users = [
    { user: "app", password: "STRONG_PASSWORD" }
  ]
}
```

### 23.3 Systemd service

```bash
sudo useradd -r -s /sbin/nologin nats
sudo mkdir -p /var/lib/nats/jetstream /etc/nats
sudo chown -R nats:nats /var/lib/nats
```

`/etc/systemd/system/nats.service`:
```ini
[Unit]
Description=NATS Server
After=network.target

[Service]
Type=simple
User=nats
ExecStart=/usr/local/bin/nats-server -c /etc/nats/nats.conf
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload
sudo systemctl enable --now nats
```

Node client: `nats` npm package.

---

## 24. BullMQ (Redis-backed Job Queue)

Best for: Node-native background jobs, delayed jobs, cron-like recurring jobs, priorities, rate limiting. **No separate service — uses your existing Redis.**

Install in your Node app:
```bash
npm install bullmq
```

Producer:
```javascript
import { Queue } from 'bullmq';

const emailQueue = new Queue('emails', {
  connection: { host: '127.0.0.1', port: 6379, password: 'REDIS_PASS', db: 2 }
});

await emailQueue.add('send-welcome', {
  to: 'user@example.com',
  template: 'welcome'
}, {
  attempts: 3,
  backoff: { type: 'exponential', delay: 1000 }
});
```

Worker:
```javascript
import { Worker } from 'bullmq';

const worker = new Worker('emails', async (job) => {
  await sendEmail(job.data);
}, {
  connection: { host: '127.0.0.1', port: 6379, password: 'REDIS_PASS', db: 2 },
  concurrency: 10
});
```

Run the worker as a separate PM2 process (in ecosystem.config.js).

Optional dashboard: `@bull-board/api` + `@bull-board/express` — mount at `/admin/queues` behind auth.

---

## 25. Broker Selection Guide (Which One to Use When)

| Use Case | Recommended | Why |
|---|---|---|
| Background jobs, emails, delayed tasks (Node-only) | **BullMQ** | Zero extra infra, uses Redis |
| Reliable task queue, complex routing, RPC | **RabbitMQ** | Mature, versatile, great management UI |
| Event streaming, high throughput (>100k msg/s) | **Apache Kafka** | Persistent log, replay, partitioning |
| Microservices pub/sub, request/reply, low latency | **NATS** | Fastest, lightest, simplest |
| Real-time WebSocket fanout across cluster | **Redis Pub/Sub** | Simple, built into Redis |
| Event sourcing / CDC | **Kafka** | Immutable log is the primary store |
| Cross-datacenter replication | **Kafka** or **NATS** | Both handle this well |
| Serverless / short-lived workers | **SQS / Cloud queues** | Not in scope for VPS |

**Realistic enterprise stack on one VPS:**
- **BullMQ** for internal Node jobs (email, notifications, image processing)
- **RabbitMQ** for reliable inter-service communication (auth service ↔ billing service)
- **Kafka** for high-volume event streams (analytics events, activity logs, audit trails)
- **Redis Pub/Sub** for Socket.io scaling across PM2 cluster

You don't need all four — pick what fits your actual data flows.

---

# PART F — APPLICATION DEPLOYMENT

## 26. Multi-App Hosting Strategy

### 26.1 Port allocation plan (example)

| App | Type | Internal Port | Subdomain |
|---|---|---|---|
| api-main | Node/Express | 3001 | api.example.com |
| api-auth | Node/Fastify | 3002 | auth.example.com |
| api-payments | Node/Express | 3003 | pay.example.com |
| worker-queue | Node worker | (no port) | — |
| web-marketing | Next.js | 3010 | www.example.com |
| web-dashboard | Next.js | 3011 | app.example.com |
| ws-realtime | Socket.io | 3020 | ws.example.com |
| rabbitmq-mgmt | Internal | 15672 | mq.example.com |
| grafana | Monitoring | 3000 | grafana.example.com |
| prometheus | Monitoring | 9090 | (internal only) |
| minio-console | Storage UI | 9001 | s3.example.com |
| meilisearch | Search | 7700 | (internal only) |

### 26.2 Folder structure

```
/home/deploy/
├── apps/
│   ├── api-main/
│   ├── api-auth/
│   ├── api-payments/
│   ├── worker-queue/
│   ├── web-marketing/
│   ├── web-dashboard/
│   └── ecosystem.config.js
├── logs/
├── backups/
├── scripts/
│   ├── pg_backup.sh
│   ├── mongo_backup.sh
│   └── offsite_upload.sh
└── infra/
    └── docker-compose.yml
```

---

## 27. Node.js / Express App Deployment

```bash
cd ~/apps
git clone git@github.com:yourname/api-main.git
cd api-main

nvm use                        # if .nvmrc exists
npm ci
npm run build                  # if TypeScript
npm prune --production
```

Create `.env`:
```
NODE_ENV=production
PORT=3001
DATABASE_URL=postgresql://app_main:PASS@127.0.0.1:5432/app_main_db
REDIS_URL=redis://:PASS@127.0.0.1:6379/0
RABBITMQ_URL=amqp://app_mq:PASS@127.0.0.1:5672/app_vhost
KAFKA_BROKERS=127.0.0.1:9092
JWT_SECRET=very_long_random_secret_min_32_chars
SENTRY_DSN=https://xxx@xxx.ingest.sentry.io/xxx
```

```bash
chmod 600 .env
pm2 start ~/apps/ecosystem.config.js --only api-main
pm2 save
```

---

## 28. Next.js App Deployment

```bash
cd ~/apps
git clone git@github.com:yourname/web-marketing.git
cd web-marketing

nvm use
npm ci
npm run build
```

For **standalone build** (recommended — much smaller), add to `next.config.js`:
```javascript
module.exports = {
  output: 'standalone',
};
```

After `npm run build`, copy static assets:
```bash
cp -r public .next/standalone/
cp -r .next/static .next/standalone/.next/
```

PM2 ecosystem entry:
```javascript
{
  name: 'web-marketing',
  cwd: '/home/deploy/apps/web-marketing',
  script: '.next/standalone/server.js',
  env: {
    NODE_ENV: 'production',
    PORT: 3010,
    HOSTNAME: '127.0.0.1'
  }
}
```

Nginx config with static caching:
```nginx
server {
    listen 80;
    server_name www.example.com example.com;

    location /_next/static/ {
        proxy_pass http://127.0.0.1:3010;
        proxy_cache_valid 200 60m;
        add_header Cache-Control "public, max-age=31536000, immutable";
    }

    location / {
        proxy_pass http://127.0.0.1:3010;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
    }
}
```

---

## 29. WebSocket / SSE / Long-polling Nginx Config

Three essentials for WebSocket, SSE, and long-polling:

```nginx
location /socket.io/ {
    proxy_pass http://127.0.0.1:3020;
    proxy_http_version 1.1;

    proxy_set_header Upgrade $http_upgrade;
    proxy_set_header Connection "upgrade";
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    proxy_read_timeout 86400;
    proxy_send_timeout 86400;

    # Critical for SSE and streaming responses
    proxy_buffering off;
    proxy_cache off;
}
```

### 29.1 Socket.io + Redis adapter (mandatory in PM2 cluster mode)

```javascript
import { Server } from 'socket.io';
import { createAdapter } from '@socket.io/redis-adapter';
import { createClient } from 'redis';

const io = new Server(httpServer, {
  cors: { origin: '*' }
});

const pubClient = createClient({
  url: 'redis://:PASSWORD@127.0.0.1:6379/3'
});
const subClient = pubClient.duplicate();

await Promise.all([pubClient.connect(), subClient.connect()]);
io.adapter(createAdapter(pubClient, subClient));
```

Without the adapter, connections in one PM2 worker cannot emit to sockets connected to another worker.

---

# PART G — THIRD-PARTY INTEGRATIONS

**General principle for API keys:**
1. Never commit keys to Git
2. Store in `.env` with `chmod 600`
3. For production, use a secrets manager (Doppler / HashiCorp Vault / AWS Secrets Manager)
4. Restrict each key to only the required scopes/IPs

---

## 30. Google Maps / Places / Geocoding APIs

### 30.1 Create keys in Google Cloud Console

1. Go to https://console.cloud.google.com/
2. Create a new project
3. **APIs & Services → Library** → enable:
   - Maps JavaScript API
   - Places API (New)
   - Geocoding API
   - Directions API (if needed)
   - Distance Matrix API (if needed)
4. **APIs & Services → Credentials → Create Credentials → API Key**
5. **IMPORTANT — restrict the key immediately:**
   - **Application restrictions:** HTTP referrers → add `https://www.example.com/*`, `https://app.example.com/*`
   - **API restrictions:** select only the APIs enabled above
6. Create a **separate server-side key** for backend calls (Geocoding, Places from Node.js):
   - Application restrictions: **IP addresses** → your VPS IP
   - API restrictions: Geocoding, Places, etc.

### 30.2 Frontend usage (Next.js)

```javascript
// .env.local (Next.js)
NEXT_PUBLIC_GOOGLE_MAPS_KEY=AIzaSyXXXX...
```

```jsx
// Load Maps script with libraries
<script
  async
  src={`https://maps.googleapis.com/maps/api/js?key=${process.env.NEXT_PUBLIC_GOOGLE_MAPS_KEY}&libraries=places&loading=async`}
/>
```

Better: use `@react-google-maps/api` or `@vis.gl/react-google-maps`.

### 30.3 Backend usage (Node.js)

```bash
npm install @googlemaps/google-maps-services-js
```

```javascript
import { Client } from '@googlemaps/google-maps-services-js';

const client = new Client({});

const result = await client.geocode({
  params: {
    address: '1600 Amphitheatre Parkway, Mountain View, CA',
    key: process.env.GOOGLE_MAPS_SERVER_KEY
  }
});
```

### 30.4 Cost control

- Set daily quotas in Cloud Console (**APIs & Services → Quotas**)
- Enable billing alerts
- Cache geocoding results in Redis (24h+ TTL) — same address rarely changes

---

## 31. Firebase (Auth, Cloud Messaging, Analytics, Firestore)

### 31.1 Setup

1. Create a project at https://console.firebase.google.com/
2. Enable the products you need:
   - **Authentication** — enable providers (Email/Password, Google, Apple, Phone)
   - **Cloud Messaging (FCM)** — for push notifications
   - **Analytics** — auto-enabled with GA4 integration
   - **Firestore** — if you want a Firebase-native NoSQL (alternative to your MongoDB)
3. **Project Settings → Service Accounts → Generate new private key** — downloads a JSON file
4. **Project Settings → General → Your apps** — add a Web app, copy the config

### 31.2 Backend (Node.js Admin SDK)

```bash
npm install firebase-admin
```

Store the service account JSON securely on the VPS:
```bash
mkdir -p ~/apps/api-main/secrets
# Upload firebase-service-account.json here via scp
chmod 600 ~/apps/api-main/secrets/firebase-service-account.json
```

`.env`:
```
FIREBASE_SERVICE_ACCOUNT_PATH=/home/deploy/apps/api-main/secrets/firebase-service-account.json
FIREBASE_PROJECT_ID=your-project-id
```

Code:
```javascript
import admin from 'firebase-admin';

admin.initializeApp({
  credential: admin.credential.cert(
    require(process.env.FIREBASE_SERVICE_ACCOUNT_PATH)
  )
});

// Verify ID token from client
const decoded = await admin.auth().verifyIdToken(idToken);

// Send FCM push notification
await admin.messaging().send({
  token: deviceToken,
  notification: { title: 'Hello', body: 'World' },
  data: { orderId: '12345' }
});

// Firestore
const db = admin.firestore();
await db.collection('users').doc(uid).set({ ... });
```

### 31.3 Frontend (Next.js)

```bash
npm install firebase
```

```javascript
// firebase-client.js
import { initializeApp } from 'firebase/app';
import { getAuth } from 'firebase/auth';
import { getAnalytics } from 'firebase/analytics';
import { getMessaging } from 'firebase/messaging';

const firebaseConfig = {
  apiKey: process.env.NEXT_PUBLIC_FIREBASE_API_KEY,
  authDomain: process.env.NEXT_PUBLIC_FIREBASE_AUTH_DOMAIN,
  projectId: process.env.NEXT_PUBLIC_FIREBASE_PROJECT_ID,
  storageBucket: process.env.NEXT_PUBLIC_FIREBASE_STORAGE_BUCKET,
  messagingSenderId: process.env.NEXT_PUBLIC_FIREBASE_MSG_SENDER_ID,
  appId: process.env.NEXT_PUBLIC_FIREBASE_APP_ID,
  measurementId: process.env.NEXT_PUBLIC_FIREBASE_MEASUREMENT_ID
};

const app = initializeApp(firebaseConfig);
export const auth = getAuth(app);
export const analytics = typeof window !== 'undefined' ? getAnalytics(app) : null;
```

### 31.4 FCM Web Push setup

1. Firebase Console → **Cloud Messaging → Web Push certificates → Generate key pair** (VAPID key)
2. Add to Next.js:
```javascript
NEXT_PUBLIC_FIREBASE_VAPID_KEY=BKagOny...
```
3. Create `public/firebase-messaging-sw.js` (service worker) per Firebase docs
4. Request permission and get token:
```javascript
import { getToken, onMessage } from 'firebase/messaging';

const token = await getToken(messaging, {
  vapidKey: process.env.NEXT_PUBLIC_FIREBASE_VAPID_KEY
});
// Send token to your backend, store against user
```

---

## 32. Google Analytics 4 (GA4)

### 32.1 Setup

1. https://analytics.google.com/ → create Property → Web data stream
2. Copy your **Measurement ID** (starts with `G-`)

### 32.2 Next.js integration (App Router)

```jsx
// app/layout.tsx
import Script from 'next/script';

export default function RootLayout({ children }) {
  return (
    <html>
      <head>
        <Script
          strategy="afterInteractive"
          src={`https://www.googletagmanager.com/gtag/js?id=${process.env.NEXT_PUBLIC_GA_ID}`}
        />
        <Script id="ga4" strategy="afterInteractive">
          {`
            window.dataLayer = window.dataLayer || [];
            function gtag(){dataLayer.push(arguments);}
            gtag('js', new Date());
            gtag('config', '${process.env.NEXT_PUBLIC_GA_ID}', {
              page_path: window.location.pathname,
            });
          `}
        </Script>
      </head>
      <body>{children}</body>
    </html>
  );
}
```

### 32.3 Custom events

```javascript
window.gtag('event', 'purchase', {
  transaction_id: order.id,
  value: order.total,
  currency: 'INR',
  items: order.items
});
```

### 32.4 Consent mode (required for GDPR / DPDP)

Implement Google Consent Mode v2 — default consent to denied, upgrade after user opts in via cookie banner. Use `cookieyes`, `iubenda`, or roll your own.

---

## 33. PostHog / Mixpanel (Product Analytics)

**PostHog** — self-hostable, all-in-one (product analytics + session replay + feature flags + A/B testing).

Cloud version: sign up at https://posthog.com/

Frontend:
```bash
npm install posthog-js
```

```javascript
import posthog from 'posthog-js';

if (typeof window !== 'undefined') {
  posthog.init(process.env.NEXT_PUBLIC_POSTHOG_KEY, {
    api_host: 'https://app.posthog.com',
    person_profiles: 'identified_only'
  });
}

posthog.capture('button_clicked', { button_name: 'signup' });
posthog.identify(userId, { email, plan });
```

Backend (Node.js):
```bash
npm install posthog-node
```

```javascript
import { PostHog } from 'posthog-node';

const client = new PostHog(process.env.POSTHOG_API_KEY, {
  host: 'https://app.posthog.com'
});

client.capture({
  distinctId: userId,
  event: 'order_completed',
  properties: { amount, currency }
});
```

---

## 34. Email Providers

### 34.1 Resend (recommended for modern Node/React apps)

```bash
npm install resend
```

```javascript
import { Resend } from 'resend';

const resend = new Resend(process.env.RESEND_API_KEY);

await resend.emails.send({
  from: 'hello@example.com',
  to: 'user@example.com',
  subject: 'Welcome',
  react: WelcomeEmail({ name: 'John' })    // React Email templates
});
```

**Setup:**
1. Sign up at https://resend.com
2. Add and verify your domain (SPF + DKIM + DMARC DNS records)
3. Create API key

### 34.2 SendGrid (enterprise-scale, more features)

```bash
npm install @sendgrid/mail
```

```javascript
import sgMail from '@sendgrid/mail';
sgMail.setApiKey(process.env.SENDGRID_API_KEY);

await sgMail.send({
  to: 'user@example.com',
  from: 'noreply@example.com',
  subject: 'Hello',
  templateId: 'd-xxxx',
  dynamicTemplateData: { name: 'John', orderId: '123' }
});
```

### 34.3 Postmark (best for transactional deliverability)

```bash
npm install postmark
```

### 34.4 Mailgun (mass mail, good for marketing)

```bash
npm install mailgun.js
```

**Critical DNS records (any provider):**
- **SPF** (TXT): `v=spf1 include:_spf.resend.com ~all`
- **DKIM** (TXT): provider gives you the value
- **DMARC** (TXT): `v=DMARC1; p=quarantine; rua=mailto:dmarc@example.com`

Without these, emails hit spam.

---

## 35. SMS & WhatsApp

### 35.1 Twilio (global, WhatsApp API included)

```bash
npm install twilio
```

```javascript
import twilio from 'twilio';

const client = twilio(
  process.env.TWILIO_ACCOUNT_SID,
  process.env.TWILIO_AUTH_TOKEN
);

// SMS
await client.messages.create({
  body: 'Your OTP is 123456',
  from: '+1234567890',
  to: '+919876543210'
});

// WhatsApp
await client.messages.create({
  body: 'Hello via WhatsApp',
  from: 'whatsapp:+14155238886',
  to: 'whatsapp:+919876543210'
});
```

### 35.2 MSG91 (India-focused, cheaper for domestic SMS)

```bash
npm install msg91
```

MSG91 offers OTP APIs, transactional SMS, WhatsApp Business, and voice OTPs at ₹0.15–0.25/SMS domestically.

### 35.3 Gupshup (India, good for WhatsApp)

Enterprise WhatsApp Business API — https://www.gupshup.io

---

## 36. Payments

### 36.1 Stripe (global)

```bash
npm install stripe
```

```javascript
import Stripe from 'stripe';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY);

// Create payment intent
const paymentIntent = await stripe.paymentIntents.create({
  amount: 2000,       // in cents / paise
  currency: 'inr',
  automatic_payment_methods: { enabled: true }
});

// Webhook handler (verify signature!)
app.post('/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  (req, res) => {
    const sig = req.headers['stripe-signature'];
    const event = stripe.webhooks.constructEvent(
      req.body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET
    );
    // Handle event.type
  }
);
```

### 36.2 Razorpay (India — recommended for INR)

```bash
npm install razorpay
```

```javascript
import Razorpay from 'razorpay';

const razorpay = new Razorpay({
  key_id: process.env.RAZORPAY_KEY_ID,
  key_secret: process.env.RAZORPAY_KEY_SECRET
});

const order = await razorpay.orders.create({
  amount: 50000,        // paise (₹500)
  currency: 'INR',
  receipt: 'order_rcpt_11'
});

// Webhook signature verification
import crypto from 'crypto';
const expected = crypto
  .createHmac('sha256', process.env.RAZORPAY_WEBHOOK_SECRET)
  .update(req.rawBody)
  .digest('hex');
if (expected !== req.headers['x-razorpay-signature']) return res.status(400).send();
```

**Always verify webhook signatures.** Never trust webhook bodies without verification — that's how fake "payment success" fraud happens.

---

## 37. Cloudflare (CDN + DNS + WAF + Turnstile)

**Highly recommended** — free tier gives you global CDN, DDoS protection, WAF, DNS, and bot protection.

### 37.1 Setup

1. Sign up at https://cloudflare.com
2. Add your domain — Cloudflare will scan existing DNS records
3. Change your registrar's nameservers to Cloudflare's (given during setup)
4. Wait for propagation (~24h max)
5. In Cloudflare dashboard:
   - **SSL/TLS → Overview → Full (strict)** — requires valid cert on your origin (Let's Encrypt gives you that)
   - **SSL/TLS → Edge Certificates → Always Use HTTPS: ON**
   - **Security → WAF → Managed Rules: ON**
   - **Speed → Optimization → Auto Minify (JS, CSS, HTML): ON**
   - **Caching → Configuration → Cache Level: Standard**

### 37.2 Restore real client IP in Nginx

When Cloudflare proxies, `$remote_addr` becomes a Cloudflare IP. Restore the real one:

```nginx
# /etc/nginx/conf.d/cloudflare.conf
# Cloudflare IPv4 ranges (update from https://www.cloudflare.com/ips-v4)
set_real_ip_from 173.245.48.0/20;
set_real_ip_from 103.21.244.0/22;
set_real_ip_from 103.22.200.0/22;
# ... (all ranges)

real_ip_header CF-Connecting-IP;
```

Reload nginx. Now UFW-level IP checks and Fail2Ban see real client IPs.

### 37.3 Cloudflare Turnstile (CAPTCHA replacement)

Better than reCAPTCHA — no puzzle for users, free unlimited.

1. Cloudflare Dashboard → **Turnstile → Add Site** → get site key + secret
2. Frontend:
```jsx
<script src="https://challenges.cloudflare.com/turnstile/v0/api.js" async defer />
<div className="cf-turnstile" data-sitekey="0xXXXX" />
```
3. Backend verification:
```javascript
const res = await fetch('https://challenges.cloudflare.com/turnstile/v0/siteverify', {
  method: 'POST',
  headers: { 'Content-Type': 'application/x-www-form-urlencoded' },
  body: `secret=${process.env.TURNSTILE_SECRET}&response=${token}`
});
const data = await res.json();
if (!data.success) throw new Error('Bot detected');
```

---

## 38. Sentry (Error Tracking + APM)

### 38.1 Node/Express

```bash
npm install @sentry/node @sentry/profiling-node
```

```javascript
import * as Sentry from '@sentry/node';
import { nodeProfilingIntegration } from '@sentry/profiling-node';

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  environment: process.env.NODE_ENV,
  integrations: [nodeProfilingIntegration()],
  tracesSampleRate: 0.1,        // 10% of transactions
  profilesSampleRate: 0.1
});

// Express: add BEFORE routes
app.use(Sentry.Handlers.requestHandler());
app.use(Sentry.Handlers.tracingHandler());

// ...your routes...

// Add AFTER routes
app.use(Sentry.Handlers.errorHandler());
```

### 38.2 Next.js

```bash
npx @sentry/wizard@latest -i nextjs
```

Follow the wizard — it creates `sentry.client.config.ts`, `sentry.server.config.ts`, `sentry.edge.config.ts`.

Add source maps upload in your CI so stack traces show original code.

---

## 39. OAuth Providers

### 39.1 NextAuth.js / Auth.js (all-in-one)

```bash
npm install next-auth
```

```javascript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GoogleProvider from 'next-auth/providers/google';
import GitHubProvider from 'next-auth/providers/github';
import AppleProvider from 'next-auth/providers/apple';
import CredentialsProvider from 'next-auth/providers/credentials';

export const authOptions = {
  providers: [
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET
    }),
    GitHubProvider({
      clientId: process.env.GITHUB_CLIENT_ID,
      clientSecret: process.env.GITHUB_CLIENT_SECRET
    }),
    AppleProvider({
      clientId: process.env.APPLE_CLIENT_ID,
      clientSecret: process.env.APPLE_CLIENT_SECRET     // generated JWT
    }),
    CredentialsProvider({
      async authorize(credentials) {
        // your DB lookup
      }
    })
  ],
  session: { strategy: 'jwt' },
  secret: process.env.NEXTAUTH_SECRET
};

export default NextAuth(authOptions);
```

**Provider setup:**
- **Google** — https://console.cloud.google.com → OAuth consent screen → Credentials → OAuth 2.0 Client ID
- **GitHub** — Settings → Developer Settings → OAuth Apps
- **Apple** — Apple Developer → Certificates → Services ID + Sign in with Apple key

Callback URL format: `https://your-domain.com/api/auth/callback/{provider}`

---

# PART H — OBSERVABILITY

## 40. Prometheus + Grafana + node_exporter

### 40.1 node_exporter (system metrics)

```bash
sudo useradd -r -s /sbin/nologin node_exporter
cd /tmp
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-amd64.tar.gz
tar -xzf node_exporter-1.8.2.linux-amd64.tar.gz
sudo mv node_exporter-1.8.2.linux-amd64/node_exporter /usr/local/bin/
```

Systemd `/etc/systemd/system/node_exporter.service`:
```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter --web.listen-address=127.0.0.1:9100

[Install]
WantedBy=multi-user.target
```

```bash
sudo systemctl daemon-reload && sudo systemctl enable --now node_exporter
```

### 40.2 Prometheus

```bash
sudo useradd -r -s /sbin/nologin prometheus
sudo mkdir -p /etc/prometheus /var/lib/prometheus
cd /tmp
wget https://github.com/prometheus/prometheus/releases/download/v2.55.0/prometheus-2.55.0.linux-amd64.tar.gz
tar -xzf prometheus-2.55.0.linux-amd64.tar.gz
cd prometheus-2.55.0.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
sudo mv consoles console_libraries /etc/prometheus/
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

Config `/etc/prometheus/prometheus.yml`:
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['127.0.0.1:9090']

  - job_name: 'node'
    static_configs:
      - targets: ['127.0.0.1:9100']

  - job_name: 'nginx'
    static_configs:
      - targets: ['127.0.0.1:9113']   # requires nginx-prometheus-exporter

  - job_name: 'postgres'
    static_configs:
      - targets: ['127.0.0.1:9187']   # requires postgres_exporter

  - job_name: 'redis'
    static_configs:
      - targets: ['127.0.0.1:9121']   # requires redis_exporter

  - job_name: 'nodejs-apps'
    static_configs:
      - targets:
          - '127.0.0.1:3001'          # your Node apps expose /metrics
          - '127.0.0.1:3002'
    metrics_path: '/metrics'
```

Systemd service — similar pattern. Enable and start.

### 40.3 Grafana

```bash
sudo apt install -y apt-transport-https software-properties-common wget
sudo mkdir -p /etc/apt/keyrings/
wget -q -O - https://apt.grafana.com/gpg.key | \
  gpg --dearmor | sudo tee /etc/apt/keyrings/grafana.gpg > /dev/null
echo "deb [signed-by=/etc/apt/keyrings/grafana.gpg] https://apt.grafana.com stable main" | \
  sudo tee /etc/apt/sources.list.d/grafana.list

sudo apt update
sudo apt install grafana -y
sudo systemctl enable --now grafana-server
```

Grafana runs on `:3000` — put behind Nginx + auth + SSL. First login: `admin` / `admin` (force change).

Import dashboards by ID:
- **1860** — Node Exporter Full
- **9628** — PostgreSQL
- **763** — Redis
- **12708** — Nginx
- **7362** — MongoDB

### 40.4 Instrumenting Node apps

```bash
npm install prom-client
```

```javascript
import express from 'express';
import client from 'prom-client';

const register = new client.Registry();
client.collectDefaultMetrics({ register });

const httpRequestDuration = new client.Histogram({
  name: 'http_request_duration_seconds',
  help: 'HTTP request duration in seconds',
  labelNames: ['method', 'route', 'status'],
  buckets: [0.005, 0.01, 0.025, 0.05, 0.1, 0.25, 0.5, 1, 2.5, 5]
});
register.registerMetric(httpRequestDuration);

app.use((req, res, next) => {
  const end = httpRequestDuration.startTimer();
  res.on('finish', () => {
    end({ method: req.method, route: req.route?.path || req.path, status: res.statusCode });
  });
  next();
});

app.get('/metrics', async (req, res) => {
  res.set('Content-Type', register.contentType);
  res.send(await register.metrics());
});
```

Protect `/metrics` — only Prometheus (localhost) should access:
```nginx
location /metrics {
    allow 127.0.0.1;
    deny all;
    proxy_pass http://127.0.0.1:3001;
}
```

---

## 41. Loki (Log Aggregation)

Ship all your logs (PM2 output, Nginx, systemd) to Loki, query in Grafana.

Install Loki:
```bash
sudo useradd -r -s /sbin/nologin loki
sudo mkdir -p /etc/loki /var/lib/loki
cd /tmp
wget https://github.com/grafana/loki/releases/latest/download/loki-linux-amd64.zip
unzip loki-linux-amd64.zip
sudo mv loki-linux-amd64 /usr/local/bin/loki
```

Grab default config from https://grafana.com/docs/loki/latest/configure/, place at `/etc/loki/config.yaml`, systemd wrap it. Bind to `127.0.0.1:3100`.

Install **Promtail** or **Grafana Alloy** on the same VPS to tail log files and ship to Loki:

```bash
wget https://github.com/grafana/loki/releases/latest/download/promtail-linux-amd64.zip
unzip promtail-linux-amd64.zip
sudo mv promtail-linux-amd64 /usr/local/bin/promtail
```

Promtail config `/etc/promtail/config.yml`:
```yaml
server:
  http_listen_port: 9080

positions:
  filename: /tmp/positions.yaml

clients:
  - url: http://127.0.0.1:3100/loki/api/v1/push

scrape_configs:
  - job_name: nginx
    static_configs:
      - targets: [localhost]
        labels:
          job: nginx
          __path__: /var/log/nginx/*.log

  - job_name: pm2
    static_configs:
      - targets: [localhost]
        labels:
          job: pm2
          __path__: /home/deploy/logs/*.log

  - job_name: syslog
    journal:
      max_age: 12h
      labels:
        job: systemd-journal
```

In Grafana, add Loki as a data source (`http://127.0.0.1:3100`), then query logs with LogQL.

---

## 42. Netdata (One-command Monitoring)

Quickest way to see everything in real time — dashboards for CPU, RAM, disk, network, per-process, DB metrics.

```bash
wget -O /tmp/netdata-kickstart.sh https://get.netdata.cloud/kickstart.sh
sh /tmp/netdata-kickstart.sh
```

Expose via Nginx + auth at `netdata.example.com`.

---

## 43. Uptime Kuma (Self-hosted Uptime Monitoring)

Beautiful alternative to UptimeRobot — monitors HTTP, TCP, ping, DNS, keyword, cert expiry.

```bash
# Via Docker (easiest)
docker run -d --restart=always -p 127.0.0.1:3001:3001 \
  -v /home/deploy/uptime-kuma:/app/data \
  --name uptime-kuma louislam/uptime-kuma:1
```

Expose at `uptime.example.com` via Nginx + SSL. Alerts via Slack, Discord, Telegram, email, Gotify, etc.

---

## 44. OpenTelemetry (OTel)

Vendor-neutral standard for traces + metrics + logs. Instrument once, send to Sentry, Grafana Tempo, Jaeger, Datadog, etc.

```bash
npm install @opentelemetry/api @opentelemetry/auto-instrumentations-node @opentelemetry/exporter-trace-otlp-http
```

```javascript
// tracing.js — require this BEFORE anything else
const { NodeSDK } = require('@opentelemetry/sdk-node');
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node');
const { OTLPTraceExporter } = require('@opentelemetry/exporter-trace-otlp-http');

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'http://127.0.0.1:4318/v1/traces'      // local OTel collector
  }),
  instrumentations: [getNodeAutoInstrumentations()]
});

sdk.start();
```

Start with `node -r ./tracing.js dist/server.js`. Automatic tracing for Express, HTTP, PostgreSQL, Redis, Kafka, and more.

---

# PART I — OPS & AUTOMATION

## 45. Environment / Secrets Management

### Levels of maturity

**Level 1 (starting):** `.env` files, `chmod 600`, in `.gitignore`, `.env.example` committed.

**Level 2 (small team):** Doppler (https://doppler.com) — free tier, one CLI to push envs to all environments.
```bash
curl -Ls https://cli.doppler.com/install.sh | sh
doppler setup
doppler run -- node dist/server.js
```

**Level 3 (enterprise):** HashiCorp Vault, AWS Secrets Manager, or GCP Secret Manager. Rotate secrets automatically, audit access.

---

## 46. Backups (Databases + Files + Offsite)

### 46.1 PostgreSQL daily backup

`~/scripts/pg_backup.sh`:
```bash
#!/bin/bash
set -e
BACKUP_DIR=/home/deploy/backups/postgres
mkdir -p "$BACKUP_DIR"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

for DB in app_main_db app_auth_db; do
  PGPASSWORD='PASS' pg_dump -h 127.0.0.1 -U app_main -d $DB -F c \
    -f "$BACKUP_DIR/${DB}_${TIMESTAMP}.dump"
done

find "$BACKUP_DIR" -name "*.dump" -mtime +14 -delete
```

### 46.2 MongoDB daily backup

`~/scripts/mongo_backup.sh`:
```bash
#!/bin/bash
set -e
BACKUP_DIR=/home/deploy/backups/mongo
TIMESTAMP=$(date +%Y%m%d_%H%M%S)
mkdir -p "$BACKUP_DIR/$TIMESTAMP"

mongodump --uri="mongodb://mongo_admin:PASS@127.0.0.1:27017/?authSource=admin" \
          --out="$BACKUP_DIR/$TIMESTAMP"

# Compress
tar -czf "$BACKUP_DIR/${TIMESTAMP}.tar.gz" -C "$BACKUP_DIR" "$TIMESTAMP"
rm -rf "$BACKUP_DIR/$TIMESTAMP"

find "$BACKUP_DIR" -name "*.tar.gz" -mtime +14 -delete
```

### 46.3 Offsite backup with restic (encrypted)

```bash
sudo apt install restic -y

# Initialize repo (Backblaze B2 example)
export B2_ACCOUNT_ID=xxx
export B2_ACCOUNT_KEY=xxx
export RESTIC_PASSWORD=very_strong_password
restic -r b2:my-vps-backups:/ init

# Backup
restic -r b2:my-vps-backups:/ backup /home/deploy/backups /etc /home/deploy/apps
```

Cron `crontab -e`:
```
0 3 * * * /home/deploy/scripts/pg_backup.sh >> /home/deploy/logs/pg_backup.log 2>&1
15 3 * * * /home/deploy/scripts/mongo_backup.sh >> /home/deploy/logs/mongo_backup.log 2>&1
30 4 * * * /home/deploy/scripts/offsite_upload.sh >> /home/deploy/logs/offsite.log 2>&1
```

**Test restore quarterly.** An untested backup is not a backup.

---

## 47. Log Rotation

`/etc/logrotate.d/deploy-apps`:
```
/home/deploy/logs/*.log {
    daily
    rotate 14
    compress
    delaycompress
    missingok
    notifempty
    copytruncate
}
```

PM2 has its own rotation (Section 10.4). Nginx has default rotation via `/etc/logrotate.d/nginx`.

---

## 48. GitHub Actions CI/CD — SSH-Based (Simplest)

### 48.1 Create a dedicated deploy key on VPS

```bash
# On VPS as deploy user
ssh-keygen -t ed25519 -f ~/.ssh/github_deploy -N ""
cat ~/.ssh/github_deploy.pub >> ~/.ssh/authorized_keys
cat ~/.ssh/github_deploy         # copy this PRIVATE key
```

Add to GitHub repo:
- **Settings → Secrets and variables → Actions → New secret**
  - `VPS_HOST` — your VPS IP
  - `VPS_USER` — `deploy`
  - `VPS_SSH_KEY` — paste the private key content
  - `VPS_SSH_PORT` — `22` (or custom)

### 48.2 Workflow file

`.github/workflows/deploy.yml`:
```yaml
name: Deploy to VPS

on:
  push:
    branches: [main]
  workflow_dispatch:

concurrency:
  group: production-deploy
  cancel-in-progress: false

jobs:
  test:
    runs-on: ubuntu-24.04
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '24'
          cache: 'npm'
      - run: npm ci
      - run: npm run lint
      - run: npm test

  deploy:
    needs: test
    runs-on: ubuntu-24.04
    environment: production
    steps:
      - name: Deploy over SSH
        uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          port: ${{ secrets.VPS_SSH_PORT }}
          script: |
            set -e
            cd /home/deploy/apps/api-main
            git fetch origin
            git reset --hard origin/main
            source ~/.nvm/nvm.sh
            nvm use
            npm ci
            npm run build
            pm2 reload api-main --update-env
            pm2 save
            echo "Deployment complete."
```

---

## 49. GitHub Actions CI/CD — Self-Hosted Runner

Runs the CI directly on your VPS — no SSH needed, faster deploys, saves GitHub minutes.

### 49.1 Register runner on VPS

```bash
# On GitHub: repo → Settings → Actions → Runners → New self-hosted runner
# Follow the commands GitHub gives you. They look like:

mkdir ~/actions-runner && cd ~/actions-runner
curl -o actions-runner-linux-x64.tar.gz -L \
  https://github.com/actions/runner/releases/download/v2.320.0/actions-runner-linux-x64-2.320.0.tar.gz
tar xzf ./actions-runner-linux-x64.tar.gz
./config.sh --url https://github.com/YOUR_ORG/YOUR_REPO --token XXXX

# Install as systemd service
sudo ./svc.sh install deploy
sudo ./svc.sh start
```

### 49.2 Workflow using self-hosted runner

```yaml
name: Deploy (Self-Hosted)

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: self-hosted
    steps:
      - uses: actions/checkout@v4
      - name: Setup Node
        run: |
          source ~/.nvm/nvm.sh
          nvm use
      - name: Install & Build
        run: |
          source ~/.nvm/nvm.sh
          npm ci
          npm run build
      - name: Sync to app directory
        run: |
          rsync -av --delete --exclude='.git' --exclude='node_modules' \
            $GITHUB_WORKSPACE/ /home/deploy/apps/api-main/
      - name: Install prod deps & reload
        run: |
          source ~/.nvm/nvm.sh
          cd /home/deploy/apps/api-main
          npm ci --production
          pm2 reload api-main --update-env
```

**Security note:** Self-hosted runners have full VPS access. Never enable them for public forks / PRs — restrict to trusted commits only.

---

## 50. GitHub Actions CI/CD — Docker + Zero-Downtime

For containerized deploys — build image in CI, push to GitHub Container Registry (GHCR), pull on VPS.

`.github/workflows/deploy.yml`:
```yaml
name: Build and Deploy

on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-24.04
    permissions:
      contents: read
      packages: write
    outputs:
      image-tag: ${{ steps.meta.outputs.version }}
    steps:
      - uses: actions/checkout@v4
      - uses: docker/setup-buildx-action@v3
      - uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - id: meta
        uses: docker/metadata-action@v5
        with:
          images: ghcr.io/${{ github.repository }}
          tags: |
            type=sha,prefix=sha-
            type=ref,event=branch
      - uses: docker/build-push-action@v6
        with:
          push: true
          tags: ${{ steps.meta.outputs.tags }}
          cache-from: type=gha
          cache-to: type=gha,mode=max

  deploy:
    needs: build
    runs-on: ubuntu-24.04
    environment: production
    steps:
      - uses: appleboy/ssh-action@v1.0.3
        with:
          host: ${{ secrets.VPS_HOST }}
          username: ${{ secrets.VPS_USER }}
          key: ${{ secrets.VPS_SSH_KEY }}
          script: |
            set -e
            IMAGE="ghcr.io/${{ github.repository }}:sha-${GITHUB_SHA::7}"
            echo "${{ secrets.GITHUB_TOKEN }}" | docker login ghcr.io -u ${{ github.actor }} --password-stdin
            docker pull $IMAGE
            cd /home/deploy/infra
            IMAGE_TAG=$IMAGE docker compose up -d --no-deps --scale api=2 api
            sleep 15
            docker compose up -d --no-deps --scale api=1 api
            docker system prune -f
```

`Dockerfile` in the app repo:
```dockerfile
FROM node:24-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:24-alpine
WORKDIR /app
RUN addgroup -g 1001 -S nodejs && adduser -S nodejs -u 1001
COPY --from=builder --chown=nodejs:nodejs /app/dist ./dist
COPY --from=builder --chown=nodejs:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodejs:nodejs /app/package.json ./
USER nodejs
EXPOSE 3001
CMD ["node", "dist/server.js"]
```

---

## 51. Staging + Production Environments

Setup:
- **Same VPS**, different subdomains (`api.example.com` vs `staging-api.example.com`), different ports, different databases.
- Or **cheaper staging VPS** — Hostinger KVM 2 or similar for a separate box.

In GitHub Actions, use `environment: staging` and `environment: production` — with different secrets and manual approvers for prod:

```yaml
jobs:
  deploy-staging:
    if: github.ref == 'refs/heads/develop'
    environment: staging     # auto-deploy on push to develop

  deploy-production:
    if: github.ref == 'refs/heads/main'
    environment: production  # requires manual approval (set in repo settings)
```

---

# PART J — SECURITY & COMPLIANCE

## 52. API Key & Secrets Storage

- Never commit secrets to Git. Add pre-commit hook:
```bash
npm install -g @secretlint/secretlint @secretlint/secretlint-rule-preset-recommend
```
- Use **git-secrets** or **detect-secrets** in CI
- Rotate keys every 90 days
- Use short-lived tokens where possible

---

## 53. Security Audit Checklist

- [ ] Root SSH login disabled
- [ ] SSH password auth disabled, keys only
- [ ] UFW enabled, only 22/80/443 open
- [ ] Fail2Ban or CrowdSec running
- [ ] Unattended security upgrades enabled
- [ ] All databases bound to `127.0.0.1` only
- [ ] All internal services (Redis, RabbitMQ, Kafka, NATS) password-protected
- [ ] All external services (Nginx-exposed) use HTTPS (Let's Encrypt)
- [ ] `server_tokens off` in Nginx
- [ ] Security headers (HSTS, X-Frame-Options, CSP, X-Content-Type-Options)
- [ ] Rate limiting configured (`limit_req_zone`)
- [ ] Cloudflare WAF enabled with real-IP restoration
- [ ] `.env` files `chmod 600`, in `.gitignore`
- [ ] Webhook signatures verified (Stripe, Razorpay, etc.)
- [ ] All API keys restricted (HTTP referrer or IP)
- [ ] Regular backups with tested restore
- [ ] Offsite backup encrypted (restic / rclone)
- [ ] Sentry / error tracking active
- [ ] Log monitoring active (Loki + alerting rules)
- [ ] Uptime monitoring from external source
- [ ] SSL certificate auto-renewal verified
- [ ] Dependency scanning (`npm audit`, Dependabot enabled on GitHub)
- [ ] File upload validation (size, MIME, ClamAV scan)

---

## 54. Rate Limiting & DDoS Mitigation

Layer defense-in-depth:
1. **Cloudflare** — L3/L4 DDoS protection at edge (free)
2. **Nginx** — application-level rate limits (`limit_req_zone`)
3. **App-level** — per-user rate limits with `express-rate-limit` + Redis store:
```bash
npm install express-rate-limit rate-limit-redis
```
```javascript
import rateLimit from 'express-rate-limit';
import RedisStore from 'rate-limit-redis';

const limiter = rateLimit({
  store: new RedisStore({ sendCommand: (...args) => redisClient.sendCommand(args) }),
  windowMs: 60_000,
  limit: 100,
  standardHeaders: 'draft-7',
  legacyHeaders: false,
  keyGenerator: (req) => req.user?.id || req.ip
});

app.use('/api/', limiter);
```

---

## 55. Compliance Notes (GDPR / DPDP India)

- **Cookie consent banner** — required in EU (GDPR) and India (DPDP 2023). Use `cookieyes`, `iubenda`, or `osano`.
- **Privacy policy + Terms** — publish on your site.
- **Data export + deletion endpoints** — users can request their data or account deletion.
- **Data Processing Agreement (DPA)** — sign with each processor (Stripe, SendGrid, etc.).
- **Consent Mode v2** for Google Analytics.
- **PII encryption at rest** — for sensitive fields, use application-level encryption (e.g., `crypto.createCipheriv` with AES-256-GCM).
- **Audit logs** — record who accessed what data.
- **India-specific:** DPDP requires data localization for certain categories; check whether your VPS location (Hostinger) meets your compliance needs.

---

# PART K — REFERENCE

## 56. Troubleshooting Common Issues

### SSH lockout
Use Hostinger's VNC / recovery console (in hPanel). Reset `/etc/ssh/sshd_config` or disable UFW temporarily.

### Nginx 502 Bad Gateway
```bash
pm2 status
pm2 logs <app> --lines 100
curl -I http://127.0.0.1:3001    # backend responsive?
sudo tail -f /var/log/nginx/error.log
```

### Certbot fails (DNS not propagated)
```bash
dig +short api.example.com       # should show VPS IP
```

### Postgres/Redis "connection refused"
```bash
sudo systemctl status postgresql redis-server
sudo ss -tlnp | grep -E '5432|6379'
```

### Disk full
```bash
df -h
du -sh /var/log/* /home/deploy/logs/* 2>/dev/null | sort -h
pm2 flush
sudo journalctl --vacuum-time=7d
docker system prune -af          # if using Docker
```

### OOM / server slow
```bash
free -h
htop
pm2 status                       # check per-app memory
dmesg | tail -50                 # look for OOM kills
```

### Kafka fails to start
```bash
sudo journalctl -u kafka -n 200
# Common: JAVA_HOME not set, insufficient heap, storage not formatted
```

---

## 57. Command Cheat Sheet

```bash
# System
sudo systemctl status <service>
sudo journalctl -u <service> -f --lines 100
sudo ss -tlnp                              # ports listening
sudo ufw status
htop
df -h && free -h

# PM2
pm2 status | pm2 list
pm2 restart <app>
pm2 reload <app>                           # zero-downtime
pm2 logs <app> --lines 200
pm2 monit
pm2 save
pm2 startup

# Nginx
sudo nginx -t                              # syntax check
sudo systemctl reload nginx
sudo tail -f /var/log/nginx/{access,error}.log

# Postgres
sudo -u postgres psql
psql -h 127.0.0.1 -U user -d db
\l  \dt  \d+ table_name  \q
sudo systemctl reload postgresql

# MongoDB
mongosh -u user -p --authenticationDatabase admin
show dbs; use myapp; show collections; db.users.findOne();

# Redis
redis-cli -a PASSWORD
INFO memory
KEYS *                                     # avoid in prod — use SCAN
FLUSHDB

# RabbitMQ
sudo rabbitmqctl status
sudo rabbitmqctl list_queues name messages consumers
sudo rabbitmqctl list_users

# Kafka
/opt/kafka/bin/kafka-topics.sh --list --bootstrap-server localhost:9092
/opt/kafka/bin/kafka-console-consumer.sh --topic X --from-beginning --bootstrap-server localhost:9092

# NATS
nats server ping                           # via nats CLI tool
nats stream list

# Certbot
sudo certbot certificates
sudo certbot renew --dry-run

# Docker
docker ps
docker compose up -d
docker compose logs -f service
docker system prune -a
docker stats

# NVM / Node
nvm ls
nvm use 24
node -v && npm -v

# Git
git log --oneline -20
git reset --hard origin/main
git clean -fd
```

---

## 58. Final Production Checklist

### Foundation
- [ ] Non-root sudo user created
- [ ] SSH keys only, root login disabled
- [ ] UFW enabled (22/80/443)
- [ ] Fail2Ban or CrowdSec active
- [ ] Unattended security updates enabled
- [ ] Swap configured, kernel tuning applied
- [ ] File descriptor limits raised

### Runtime
- [ ] Node.js LTS via NVM
- [ ] PM2 startup registered, `pm2 save` done
- [ ] PM2 log rotate installed
- [ ] Nginx installed with hardened global config
- [ ] SSL on every domain (auto-renewal tested)

### Data
- [ ] PostgreSQL 18 with `scram-sha-256`, localhost-only
- [ ] MongoDB 8 with auth enabled, localhost-only
- [ ] Redis 8 with `requirepass`, persistence on
- [ ] MinIO for object storage (if needed)

### Brokers
- [ ] RabbitMQ 4.x — default `guest` deleted, per-app vhost
- [ ] Kafka 4.x running in KRaft mode
- [ ] NATS with JetStream and auth
- [ ] BullMQ workers running as PM2 processes

### Apps
- [ ] All apps in `/home/deploy/apps/`
- [ ] Ecosystem file managed
- [ ] `.env` files `chmod 600`, in `.gitignore`
- [ ] Health check endpoints exposed
- [ ] `/metrics` endpoints restricted to localhost

### Third-party
- [ ] Google Maps keys restricted (referrer + IP)
- [ ] Firebase service account JSON secured
- [ ] GA4 with consent mode v2
- [ ] Email provider DNS records (SPF, DKIM, DMARC)
- [ ] Payment webhooks verify signatures
- [ ] Cloudflare in front with real-IP restoration
- [ ] Sentry integrated on server + client

### Observability
- [ ] Prometheus scraping node, nginx, DBs, apps
- [ ] Grafana dashboards imported
- [ ] Loki ingesting logs from Nginx + PM2 + systemd
- [ ] Alerts configured (email/Slack)
- [ ] External uptime monitoring active

### Ops
- [ ] Daily DB backups scripted
- [ ] Offsite backup uploaded (restic/rclone)
- [ ] Backup restore tested
- [ ] CI/CD workflow deployed (GitHub Actions)
- [ ] Staging environment separate from prod

### Security
- [ ] Rate limiting at Nginx + app level
- [ ] Security headers (HSTS, CSP, etc.)
- [ ] Dependency scanning (Dependabot)
- [ ] Secrets rotated (initial rotation from setup)
- [ ] Cookie consent + privacy policy live
- [ ] Data export + deletion endpoints ready

---

## Notes for Long-term Maintenance

- **Weekly:** review Grafana dashboards, check Sentry issues, review Fail2Ban bans.
- **Monthly:** `sudo apt update && apt list --upgradable`, plan security-relevant reboots.
- **Quarterly:** test backup restore, rotate high-privilege secrets, review dependency vulnerabilities.
- **Semi-annually:** upgrade Node LTS if a new one is out, review Nginx / DB configs against latest hardening guides.
- **When upgrading anything:** always snapshot the VPS (Hostinger provides this) before major upgrades.

---

**End of Guide.** Bookmark this file — the sections are independent enough that you can jump straight to whichever piece you need on any given day.
