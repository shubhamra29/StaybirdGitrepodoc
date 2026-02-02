# StayBird Platform - Quick Setup Guide

## 🚀 Quick Start (5 minutes)

### Prerequisites
- macOS/Linux/WSL2
- Python 3.10+ 
- Node.js 18+
- MariaDB 10.6+ or MySQL 8.0+
- Redis
- Git

### One-Line Setup (for developers)
```bash
curl -sSL https://raw.githubusercontent.com/ekanshjain/staybird-setup/main/install.sh | bash
```

---

## 📋 Manual Setup Steps

### 1. Clone the Repository
```bash
# Clone the bench
git clone https://github.com/ekanshjain/staybird-bench.git
cd staybird-bench
```

### 2. Install Frappe Bench
```bash
# Install bench CLI
pip3 install frappe-bench

# Initialize bench (if starting fresh)
bench init staybird-bench --frappe-branch version-15
cd staybird-bench
```

### 3. Database Setup
```bash
# macOS (Homebrew)
brew install mariadb redis
brew services start mariadb
brew services start redis

# Ubuntu/Debian
sudo apt-get install mariadb-server redis-server
sudo systemctl start mariadb redis

# Set MariaDB root password (optional)
sudo mysql_secure_installation
```

### 4. Get Core Apps
```bash
# Get ERPNext
bench get-app erpnext --branch version-15

# Get StayBird apps from ekanshjain's GitHub
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

### 5. Create Site & Install Apps
```bash
# Create new site
bench new-site staybird.local \
  --db-root-password [your-mariadb-root-password] \
  --admin-password [choose-admin-password]

# Set as default site
bench use staybird.local

# Install apps
bench --site staybird.local install-app erpnext
bench --site staybird.local install-app staybird_core
bench --site staybird.local install-app staybird_pms
bench --site staybird.local install-app staybird_pos
bench --site staybird.local install-app staybird_inventory
bench --site staybird.local install-app staybird_travel
bench --site staybird.local install-app staybird_connectors
bench --site staybird.local install-app staybird_crm_portal
bench --site staybird.local install-app staybird_finance_bridge
bench --site staybird.local install-app staybird_banquets
```

### 6. Start Development Server
```bash
# Start bench (development mode)
bench start

# Access the application
# URL: http://localhost:8001
# Username: Administrator
# Password: [admin-password-you-set]
```

---

## 🐳 Docker Setup (Alternative)

### Using Docker Compose
```bash
# Clone repository
git clone https://github.com/ekanshjain/staybird-docker.git
cd staybird-docker

# Copy environment file
cp .env.example .env

# Edit .env with your settings
nano .env

# Start containers
docker-compose up -d

# Access at http://localhost:8001
```

### Docker Commands
```bash
# View logs
docker-compose logs -f

# Stop containers
docker-compose down

# Reset everything
docker-compose down -v
```

---

## 🛠️ Common Commands

### Bench Management
```bash
# Update bench
bench update

# Restart bench
bench restart

# Clear cache
bench clear-cache

# Run migrations
bench migrate

# Backup site
bench backup
```

### Development Commands
```bash
# Enter Python console
bench --site staybird.local console

# Run tests
bench --site staybird.local run-tests

# Build assets
bench build

# Watch for changes
bench watch
```

### Database Commands
```bash
# Access MariaDB
bench mariadb

# Export database
bench --site staybird.local backup

# Import database
bench --site staybird.local restore [backup-file]
```

---

## 🔧 Troubleshooting

### Issue: Module Not Found
```bash
# Reinstall app in development mode
cd apps/[app-name]
pip install -e .
cd ../..
bench restart
```

### Issue: Port Already in Use
```bash
# Kill existing processes
pkill -f "bench start"
pkill -f redis
pkill -f frappe

# Start fresh
bench start
```

### Issue: Database Connection Error
```bash
# Check MariaDB status
brew services list | grep mariadb  # macOS
systemctl status mariadb           # Linux

# Reset MariaDB password
sudo mysql -u root
ALTER USER 'root'@'localhost' IDENTIFIED BY '';
FLUSH PRIVILEGES;
```

### Issue: Redis Connection Error
```bash
# Check Redis status
redis-cli ping

# Restart Redis
brew services restart redis  # macOS
sudo systemctl restart redis # Linux
```

---

## 🔐 Default Credentials

| Service | Username | Password | URL |
|---------|----------|----------|-----|
| Frappe | Administrator | (set during setup) | http://localhost:8001 |
| MariaDB | root | (empty or set) | localhost:3306 |
| Redis | - | - | localhost:6379 |
| GitHub | ekanshjain | - | https://github.com/ekanshjain |

---

## 📱 Frontend Setup (Vue.js)

```bash
# Navigate to frontend
cd staybird-frontend

# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build
```

---

## 🚢 Production Deployment

### Using Supervisor
```bash
bench setup production [user]
sudo supervisorctl restart all
```

### Using Systemd
```bash
bench setup systemd
sudo systemctl restart bench
```

### Using PM2
```bash
npm install -g pm2
pm2 start ecosystem.config.js
```

---

## 📚 Resources

- **GitHub Repository:** https://github.com/ekanshjain/staybird-bench
- **Documentation:** https://github.com/ekanshjain/staybird-docs
- **Issues & Support:** https://github.com/ekanshjain/staybird-bench/issues
- **Contact:** mailme@ekanshjain.com

---

## ⚡ Quick Tips

1. **Use aliases for common commands:**
```bash
alias bs='bench start'
alias br='bench restart'
alias bc='bench --site staybird.local console'
```

2. **Enable hot reload:**
```bash
bench set-config developer_mode 1
bench clear-cache
```

3. **View logs in real-time:**
```bash
bench --site staybird.local logs --tail
```

4. **Quick database reset:**
```bash
bench reinstall --yes
```

---

*Last Updated: September 2025*
