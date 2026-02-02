# StayBird Platform - Deployment Guide

## 🚀 Deployment Overview

This guide covers deployment strategies for the StayBird Platform across different environments and scales.

**Author:** Ekansh Jain ([@ekanshjain](https://github.com/ekanshjain))  
**Contact:** mailme@ekanshjain.com

---

## 📋 Pre-Deployment Checklist

### System Requirements

#### Minimum (Development/Testing)
- **CPU:** 2 cores
- **RAM:** 4 GB
- **Storage:** 20 GB SSD
- **OS:** Ubuntu 20.04+ / CentOS 8+ / macOS

#### Recommended (Production)
- **CPU:** 8+ cores
- **RAM:** 16+ GB
- **Storage:** 100+ GB SSD
- **OS:** Ubuntu 22.04 LTS
- **Database:** Dedicated MariaDB server
- **Cache:** Dedicated Redis server

### Required Software
- Python 3.10+
- Node.js 18+
- MariaDB 10.6+ or MySQL 8.0+
- Redis 7.0+
- Nginx 1.24+
- Supervisor or Systemd
- Git

---

## 🏗️ Production Deployment

### 1. Server Setup

```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install dependencies
sudo apt install -y python3-pip python3-dev python3-venv \
  mariadb-server mariadb-client redis-server \
  nginx supervisor git curl software-properties-common

# Install Node.js
curl -fsSL https://deb.nodesource.com/setup_18.x | sudo -E bash -
sudo apt install -y nodejs

# Create deployment user
sudo adduser staybird
sudo usermod -aG sudo staybird
su - staybird
```

### 2. Database Setup

```bash
# Secure MariaDB installation
sudo mysql_secure_installation

# Create database and user
sudo mysql -u root -p
```

```sql
CREATE DATABASE staybird_production;
CREATE USER 'staybird'@'localhost' IDENTIFIED BY 'strong_password_here';
GRANT ALL PRIVILEGES ON staybird_production.* TO 'staybird'@'localhost';
FLUSH PRIVILEGES;
EXIT;
```

### 3. Install Frappe Bench

```bash
# Install bench
pip3 install frappe-bench

# Initialize bench
bench init staybird-bench --frappe-branch version-15
cd staybird-bench
```

### 4. Get Applications

```bash
# Get ERPNext
bench get-app erpnext --branch version-15

# Get StayBird apps from ekanshjain's repos
bench get-app https://github.com/ekanshjain/staybird-core.git
bench get-app https://github.com/ekanshjain/staybird-pms.git
bench get-app https://github.com/ekanshjain/staybird-pos.git
bench get-app https://github.com/ekanshjain/staybird-inventory.git
bench get-app https://github.com/ekanshjain/staybird-travel.git
bench get-app https://github.com/ekanshjain/staybird-connectors.git
bench get-app https://github.com/ekanshjain/staybird-crm-portal.git
bench get-app https://github.com/ekanshjain/staybird-finance-bridge.git
bench get-app https://github.com/ekanshjain/staybird-banquets.git
```

### 5. Create Production Site

```bash
# Create site
bench new-site staybird.com \
  --db-root-password [mariadb_root_password] \
  --admin-password [admin_password] \
  --db-name staybird_production

# Install apps
bench --site staybird.com install-app erpnext
bench --site staybird.com install-app staybird_core
bench --site staybird.com install-app staybird_pms
# ... install other apps

# Set as default site
bench use staybird.com
```

### 6. Production Configuration

```bash
# Disable developer mode
bench --site staybird.com set-config developer_mode 0

# Set production configuration
bench --site staybird.com set-config host_name "https://staybird.com"
bench --site staybird.com set-config mail_server "smtp.gmail.com"
bench --site staybird.com set-config mail_port 587
bench --site staybird.com set-config mail_login "mailme@ekanshjain.com"

# Enable scheduler
bench --site staybird.com scheduler enable

# Build assets
bench build --production
```

### 7. Setup Nginx

```bash
# Generate nginx config
bench setup nginx

# Create symlink
sudo ln -s /home/staybird/staybird-bench/config/nginx.conf \
  /etc/nginx/sites-enabled/staybird.conf

# Test and reload nginx
sudo nginx -t
sudo systemctl reload nginx
```

### 8. Setup Supervisor

```bash
# Generate supervisor config
bench setup supervisor

# Create symlink
sudo ln -s /home/staybird/staybird-bench/config/supervisor.conf \
  /etc/supervisor/conf.d/staybird.conf

# Update and start supervisor
sudo supervisorctl reread
sudo supervisorctl update
sudo supervisorctl start all
```

### 9. SSL Certificate (Let's Encrypt)

```bash
# Install certbot
sudo apt install certbot python3-certbot-nginx

# Get SSL certificate
sudo certbot --nginx -d staybird.com -d www.staybird.com \
  --email mailme@ekanshjain.com --agree-tos

# Auto-renewal
sudo systemctl enable certbot.timer
```

---

## 🐳 Docker Deployment

### Docker Compose Setup

Create `docker-compose.yml`:

```yaml
version: '3.8'

services:
  mariadb:
    image: mariadb:10.6
    environment:
      MYSQL_ROOT_PASSWORD: ${DB_ROOT_PASSWORD}
      MYSQL_DATABASE: staybird
      MYSQL_USER: staybird
      MYSQL_PASSWORD: ${DB_PASSWORD}
    volumes:
      - mariadb-data:/var/lib/mysql
    networks:
      - staybird-network

  redis:
    image: redis:7-alpine
    networks:
      - staybird-network

  frappe:
    image: frappe/frappe-socketio:v15
    depends_on:
      - mariadb
      - redis
    environment:
      - DB_HOST=mariadb
      - DB_PORT=3306
      - REDIS_CACHE=redis://redis:6379/0
      - REDIS_QUEUE=redis://redis:6379/1
      - REDIS_SOCKETIO=redis://redis:6379/2
    volumes:
      - sites:/home/frappe/frappe-bench/sites
      - apps:/home/frappe/frappe-bench/apps
    networks:
      - staybird-network

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
      - sites:/usr/share/nginx/html
      - ssl-certs:/etc/letsencrypt
    depends_on:
      - frappe
    networks:
      - staybird-network

volumes:
  mariadb-data:
  sites:
  apps:
  ssl-certs:

networks:
  staybird-network:
    driver: bridge
```

### Docker Build & Deploy

```bash
# Clone repository
git clone https://github.com/ekanshjain/staybird-docker.git
cd staybird-docker

# Create environment file
cp .env.example .env
# Edit .env with your settings

# Build and start containers
docker-compose up -d

# Initialize site
docker-compose exec frappe bench new-site staybird.local \
  --admin-password admin \
  --db-root-password root

# Install apps
docker-compose exec frappe bench --site staybird.local install-app erpnext
docker-compose exec frappe bench --site staybird.local install-app staybird_core
# ... install other apps
```

---

## ☸️ Kubernetes Deployment

### Kubernetes Manifests

Create `k8s/deployment.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: staybird-app
  namespace: staybird
spec:
  replicas: 3
  selector:
    matchLabels:
      app: staybird
  template:
    metadata:
      labels:
        app: staybird
    spec:
      containers:
      - name: frappe
        image: ekanshjain/staybird:latest
        ports:
        - containerPort: 8000
        env:
        - name: DB_HOST
          valueFrom:
            secretKeyRef:
              name: staybird-secrets
              key: db-host
        - name: DB_PASSWORD
          valueFrom:
            secretKeyRef:
              name: staybird-secrets
              key: db-password
        volumeMounts:
        - name: sites
          mountPath: /home/frappe/frappe-bench/sites
      volumes:
      - name: sites
        persistentVolumeClaim:
          claimName: staybird-sites-pvc
---
apiVersion: v1
kind: Service
metadata:
  name: staybird-service
  namespace: staybird
spec:
  selector:
    app: staybird
  ports:
  - port: 80
    targetPort: 8000
  type: LoadBalancer
```

### Deploy to Kubernetes

```bash
# Create namespace
kubectl create namespace staybird

# Create secrets
kubectl create secret generic staybird-secrets \
  --from-literal=db-host=mariadb.staybird.svc.cluster.local \
  --from-literal=db-password=your_password \
  -n staybird

# Apply manifests
kubectl apply -f k8s/

# Check deployment
kubectl get pods -n staybird
kubectl get svc -n staybird
```

---

## ☁️ Cloud Deployment

### AWS EC2 Deployment

```bash
# Launch EC2 instance (Ubuntu 22.04, t3.large minimum)
# Configure security groups:
# - SSH (22)
# - HTTP (80)
# - HTTPS (443)

# Connect to instance
ssh -i your-key.pem ubuntu@ec2-instance-ip

# Follow production deployment steps above
```

### AWS RDS for Database

```bash
# Create RDS MariaDB instance
# - Engine: MariaDB 10.6
# - Instance: db.t3.medium
# - Storage: 100GB SSD
# - Multi-AZ: Yes (for production)

# Update bench config
bench --site staybird.com set-config db_host "your-rds-endpoint.amazonaws.com"
bench --site staybird.com set-config db_port 3306
```

### AWS ElastiCache for Redis

```bash
# Create ElastiCache Redis cluster
# - Engine: Redis 7.0
# - Node type: cache.t3.micro
# - Number of nodes: 2 (for redundancy)

# Update bench config
bench --site staybird.com set-config redis_cache "redis://your-elasticache-endpoint:6379/0"
bench --site staybird.com set-config redis_queue "redis://your-elasticache-endpoint:6379/1"
```

---

## 🔄 CI/CD Pipeline

### GitHub Actions Workflow

Create `.github/workflows/deploy.yml`:

```yaml
name: Deploy StayBird

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v4
      with:
        python-version: '3.10'
    
    - name: Install dependencies
      run: |
        pip install frappe-bench
        bench init test-bench --skip-redis-config-generation
        cd test-bench
        bench get-app ${{ github.repository }}
    
    - name: Run tests
      run: |
        cd test-bench
        bench --site test.local run-tests

  deploy:
    needs: test
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
    - uses: actions/checkout@v3
    
    - name: Deploy to production
      uses: appleboy/ssh-action@v0.1.5
      with:
        host: ${{ secrets.PROD_HOST }}
        username: ${{ secrets.PROD_USER }}
        key: ${{ secrets.PROD_SSH_KEY }}
        script: |
          cd /home/staybird/staybird-bench
          git pull origin main
          bench migrate
          bench build --production
          bench restart
```

---

## 📊 Monitoring & Logging

### Setup Monitoring Stack

```bash
# Install Prometheus
wget https://github.com/prometheus/prometheus/releases/download/v2.45.0/prometheus-2.45.0.linux-amd64.tar.gz
tar xvf prometheus-2.45.0.linux-amd64.tar.gz
sudo mv prometheus-2.45.0.linux-amd64 /opt/prometheus

# Install Grafana
sudo apt-get install -y software-properties-common
sudo add-apt-repository "deb https://packages.grafana.com/oss/deb stable main"
sudo apt-get update
sudo apt-get install grafana

# Start services
sudo systemctl start prometheus
sudo systemctl start grafana-server
```

### Application Monitoring

```python
# Add to hooks.py
monitoring_endpoints = [
    {
        "route": "/health",
        "handler": "staybird_core.monitoring.health_check"
    },
    {
        "route": "/metrics",
        "handler": "staybird_core.monitoring.prometheus_metrics"
    }
]
```

### Log Aggregation

```bash
# Install Elasticsearch
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.9.0-amd64.deb
sudo dpkg -i elasticsearch-8.9.0-amd64.deb

# Install Logstash
wget https://artifacts.elastic.co/downloads/logstash/logstash-8.9.0-amd64.deb
sudo dpkg -i logstash-8.9.0-amd64.deb

# Install Kibana
wget https://artifacts.elastic.co/downloads/kibana/kibana-8.9.0-amd64.deb
sudo dpkg -i kibana-8.9.0-amd64.deb

# Configure Logstash for Frappe logs
cat > /etc/logstash/conf.d/frappe.conf << EOF
input {
  file {
    path => "/home/staybird/staybird-bench/logs/*.log"
    start_position => "beginning"
  }
}

output {
  elasticsearch {
    hosts => ["localhost:9200"]
    index => "frappe-%{+YYYY.MM.dd}"
  }
}
EOF
```

---

## 🔐 Security Hardening

### Firewall Configuration

```bash
# Setup UFW firewall
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow ssh
sudo ufw allow http
sudo ufw allow https
sudo ufw enable
```

### Fail2ban Setup

```bash
# Install fail2ban
sudo apt install fail2ban

# Configure for Frappe
cat > /etc/fail2ban/jail.local << EOF
[frappe]
enabled = true
port = http,https
filter = frappe
logpath = /home/staybird/staybird-bench/logs/web.log
maxretry = 5
bantime = 3600
EOF
```

### Security Headers (Nginx)

```nginx
# Add to nginx.conf
add_header X-Frame-Options "SAMEORIGIN" always;
add_header X-Content-Type-Options "nosniff" always;
add_header X-XSS-Protection "1; mode=block" always;
add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;
add_header Content-Security-Policy "default-src 'self' https:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';" always;
```

---

## 🔄 Backup & Recovery

### Automated Backups

```bash
# Create backup script
cat > /home/staybird/backup.sh << 'EOF'
#!/bin/bash
BACKUP_DIR="/home/staybird/backups"
TIMESTAMP=$(date +%Y%m%d_%H%M%S)

# Create backup directory
mkdir -p $BACKUP_DIR

# Backup database
cd /home/staybird/staybird-bench
bench --site staybird.com backup

# Copy to backup directory
cp sites/staybird.com/private/backups/*.sql.gz $BACKUP_DIR/db_$TIMESTAMP.sql.gz

# Backup files
tar -czf $BACKUP_DIR/files_$TIMESTAMP.tar.gz sites/staybird.com/public/files

# Upload to S3 (optional)
aws s3 cp $BACKUP_DIR/db_$TIMESTAMP.sql.gz s3://staybird-backups/
aws s3 cp $BACKUP_DIR/files_$TIMESTAMP.tar.gz s3://staybird-backups/

# Clean old backups (keep last 30 days)
find $BACKUP_DIR -type f -mtime +30 -delete
EOF

chmod +x /home/staybird/backup.sh

# Add to crontab (daily at 2 AM)
echo "0 2 * * * /home/staybird/backup.sh" | crontab -
```

### Disaster Recovery

```bash
# Restore from backup
cd /home/staybird/staybird-bench

# Restore database
gunzip -c /path/to/backup.sql.gz | bench --site staybird.com mariadb

# Restore files
tar -xzf /path/to/files_backup.tar.gz -C sites/staybird.com/public/

# Run migrations
bench --site staybird.com migrate

# Clear cache
bench --site staybird.com clear-cache

# Restart services
bench restart
```

---

## 🚦 Health Checks

### Uptime Monitoring

```bash
# Using UptimeRobot or Pingdom
# Monitor endpoints:
# - https://staybird.com (Main site)
# - https://staybird.com/api/method/ping (API health)
# - https://staybird.com/health (Custom health endpoint)
```

### Health Check Script

```python
# health_check.py
import requests
import smtplib
from email.mime.text import MIMEText

def check_health():
    endpoints = [
        'https://staybird.com',
        'https://staybird.com/api/method/ping',
        'https://staybird.com/health'
    ]
    
    for endpoint in endpoints:
        try:
            response = requests.get(endpoint, timeout=10)
            if response.status_code != 200:
                send_alert(f"Endpoint {endpoint} returned {response.status_code}")
        except Exception as e:
            send_alert(f"Endpoint {endpoint} is down: {str(e)}")

def send_alert(message):
    msg = MIMEText(message)
    msg['Subject'] = 'StayBird Health Check Alert'
    msg['From'] = 'alerts@staybird.com'
    msg['To'] = 'mailme@ekanshjain.com'
    
    s = smtplib.SMTP('localhost')
    s.send_message(msg)
    s.quit()

if __name__ == '__main__':
    check_health()
```

---

## 📈 Scaling Strategies

### Horizontal Scaling

```bash
# Add more application servers
# Update nginx upstream configuration
upstream frappe_servers {
    server 10.0.1.10:8000;
    server 10.0.1.11:8000;
    server 10.0.1.12:8000;
}
```

### Database Replication

```sql
-- On master server
CREATE USER 'replicator'@'%' IDENTIFIED BY 'password';
GRANT REPLICATION SLAVE ON *.* TO 'replicator'@'%';

-- On slave server
CHANGE MASTER TO
  MASTER_HOST='master-ip',
  MASTER_USER='replicator',
  MASTER_PASSWORD='password',
  MASTER_LOG_FILE='mysql-bin.000001',
  MASTER_LOG_POS=0;

START SLAVE;
```

---

## 🛠️ Troubleshooting

### Common Issues

1. **502 Bad Gateway**
   ```bash
   # Check if frappe is running
   sudo supervisorctl status
   # Restart services
   sudo supervisorctl restart all
   ```

2. **Database Connection Error**
   ```bash
   # Check MariaDB status
   sudo systemctl status mariadb
   # Check credentials
   bench --site staybird.com mariadb
   ```

3. **Redis Connection Error**
   ```bash
   # Check Redis status
   redis-cli ping
   # Restart Redis
   sudo systemctl restart redis
   ```

4. **Permission Issues**
   ```bash
   # Fix permissions
   sudo chown -R staybird:staybird /home/staybird/staybird-bench
   ```

---

## 📞 Support

**Contact:** Ekansh Jain  
**Email:** mailme@ekanshjain.com  
**GitHub:** [@ekanshjain](https://github.com/ekanshjain)  
**Issues:** https://github.com/ekanshjain/staybird-bench/issues

---

*Last Updated: September 2025*
