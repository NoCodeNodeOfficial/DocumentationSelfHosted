# 08 - Updates & Upgrades

**Difficulty Level:** 🟡 Intermediate  
**Risk Level:** 🟠 Medium - *Be careful always backup first!*

Regular updates keep your system secure, stable, and feature-rich. This guide covers updating:

- **n8n** - Your automation platform
- **Docker containers** - Supporting services (MySQL, Traefik, etc.)
- **Operating system** - Ubuntu/Debian packages
- **Docker itself** - The container engine  

> ⚠️ **IMPORTANT:** ALWAYS create a backup before updating anything! Updates can occasionally break things, and you'll want a way back.

## 📋 Table of Contents
1. [Update Philosophy](#update-philosophy)
2. [Before You Update](#before-you-update)
3. [Updating n8n](#updating-n8n)
4. [Updating Other Docker Containers](#updating-other-docker-containers)
5. [Operating System Updates](#operating-system-updates)
6. [Rollback Procedures](#rollback-procedures)
7. [Update Schedule & Strategy](#update-schedule--strategy)
8. [Troubleshooting Updates](#troubleshooting-updates)
9. [Quick Reference](#quick-reference)

---

## Update Philosophy

> 💡 **TIP:** Follow the "test, backup, update" approach:
> 1. Test updates in a non-production environment (if possible)
> 2. Backup everything
> 3. Update during low-traffic periods
> 4. Monitor for issues after updating

---

## Before You Update

### Pre-Update Checklist

**Every time before updating, complete this checklist:**

- [ ] Create a backup (see [06 - Backup & Recovery](06-backup-recovery.md))
- [ ] Check disk space (`df -h` - need at least 2GB free)
- [ ] Note current versions (for rollback if needed)
- [ ] Read release notes for breaking changes
- [ ] Choose a low-traffic time window
- [ ] Have terminal access (not just browser)

### Document Current Versions

```bash
# Create a version snapshot before updating
cd ~/docker

# Check n8n version
docker compose exec n8n n8n --version
2.1.2

# Check traefik version
docker compose exec traefik traefik version
Version:      3.6.5
Codename:     ramequin
Go version:   go1.24.11
Built:        2025-12-16T14:56:48Z
OS/Arch:      linux/amd64

# Check MySQL version
docker compose exec db_core mysql -V
mysql  Ver 8.4.7 for Linux on x86_64 (MySQL Community Server - GPL)

# Check all container versions
docker compose ps --format "table {{.Service}}\t{{.Image}}"

# Example output:
SERVICE   IMAGE
db_core   mysql:8.4
n8n       docker.n8n.io/n8nio/n8n
phpmy     phpmyadmin:5.2
traefik   traefik
```

### Create Pre-Update Backup

```bash
# Run your backup script seen in [06 - Backup & Recovery]
cd ~/docker
./backup.sh

# Verify backup was created
ls -lh backups/ | head -5

# Example output:
# -rw-r--r-- 1 user user 234M Jan 15 14:30 mysql-20240115-143000.tar.gz
# -rw-r--r-- 1 user user  45M Jan 15 14:30 n8n-20240115-143000.tar.gz
# -rw-r--r-- 1 user user 2.1M Jan 15 14:30 traefik-20240115-143000.tar.gz
```

> ⚠️ **IMPORTANT:** Don't skip the backup! Even "small" updates can cause issues.

---

## Updating n8n

### Check for n8n Updates

```bash
# See available versions on Docker Hub
curl -s https://registry.hub.docker.com/v2/repositories/n8nio/n8n/tags/ | grep -oP '"name":"\K[^"]+' | head -10

# Example output:
latest
nightly
nightly-arm64
nightly-amd64
stable
2.1.4
2.1.4-arm64
2.1.4-amd64
next
beta
```

Or visit: https://hub.docker.com/r/n8nio/n8n/tags

### Method 1: Update to Latest Version

> 💡 **TIP:** Use `latest` tag for automatic updates to newest stable version.

**Step 1: Backup (if not done already)**
```bash
cd ~/docker
./backup.sh
```

**Step 2: Update docker-compose.yml**
```bash
nano docker-compose.yml
```

Change n8n image line:
```yaml
# FROM:
image: n8nio/n8n:1.19.4

# TO:
image: n8nio/n8n:latest
```

**Step 3: Pull new image and restart**
```bash
# Pull the latest image
docker compose pull n8n

# Example output:
# [+] Pulling 1/1
#  ✔ n8n Pulled                                                     15.2s

# Stop n8n
docker compose stop n8n

# Start with new image
docker compose up -d n8n

# Example output:
# [+] Running 1/1
#  ✔ Container n8n  Started                                         0.8s
```

**Step 4: Verify update**
```bash
# Check version
docker compose exec n8n n8n --version

# Example output:
# 1.20.0

# Check logs for startup
docker compose logs n8n --tail=30

# Example output:
# n8n  | 2024-01-15 14:35:12 | INFO  | n8n ready on port 5678
# n8n  | 2024-01-15 14:35:13 | INFO  | Version: 1.20.0
# n8n  | 2024-01-15 14:35:14 | INFO  | Database migrations completed
```

**Step 5: Test n8n**

Visit your n8n URL in browser and verify:
- [ ] Login works
- [ ] Workflows are visible
- [ ] Test a simple workflow execution
- [ ] Check credentials are intact

> ⚠️ **IMPORTANT:** If anything doesn't work, see [Rollback Procedures](#rollback-procedures) below.

### Method 2: Update to Specific Version

If you want to update to a specific version (more controlled):

**Step 1: Choose version**
```bash
# Check available versions
curl -s https://registry.hub.docker.com/v2/repositories/n8nio/n8n/tags/ | grep -oP '"name":"\K[^"]+' | grep -E '^[12]\.' | head -10
```

**Step 2: Update docker-compose.yml**
```bash
nano docker-compose.yml
```

Change to specific version:
```yaml
# Specific version (recommended for production)
image: n8nio/n8n:2.1.4
```

**Step 3: Apply update**
```bash
docker compose pull n8n
docker compose stop n8n
docker compose up -d n8n
docker compose logs n8n --tail=30
```

### n8n Update Script

Create a reusable update script:

```bash
nano ~/docker/update-n8n.sh
```

Paste this content:

```bash
#!/bin/bash

set -e

echo "=== n8n Update Script ==="
echo ""

# Check if version specified
if [ -z "$1" ]; then
    VERSION="latest"
    echo "📦 Updating to latest version"
else
    VERSION="$1"
    echo "📦 Updating to version: $VERSION"
fi

echo ""
read -p "⚠️  Have you created a backup? (yes/no): " backup_confirm

if [ "$backup_confirm" != "yes" ]; then
    echo "❌ Please create a backup first using: ./backup.sh"
    exit 1
fi

echo ""
echo "1️⃣  Stopping n8n..."
docker compose stop n8n

echo "2️⃣  Pulling new image..."
docker compose pull n8n

echo "3️⃣  Starting n8n with new version..."
docker compose up -d n8n

echo "4️⃣  Waiting for n8n to start..."
sleep 10

echo "5️⃣  Checking version..."
docker compose exec n8n n8n --version

echo ""
echo "6️⃣  Recent logs:"
docker compose logs n8n --tail=20

echo ""
echo "✅ Update completed!"
echo "🌐 Test your n8n instance at your URL"
echo ""
echo "If there are issues, rollback with:"
echo "   cd ~/docker && docker compose down n8n"
echo "   # Edit docker-compose.yml to previous version"
echo "   docker compose up -d n8n"
```

Make executable and use:

```bash
chmod +x ~/docker/update-n8n.sh

# Update to latest
./update-n8n.sh

# Update to specific version
./update-n8n.sh 2.1.4
```

---

## Updating Other Docker Containers

### Update All Containers at Once

> ⚠️ **IMPORTANT:** This updates ALL containers (If they are set to version "latest"). They'll restart briefly.

```bash
cd ~/docker

# 1. Backup first!
./backup.sh

# 2. Pull all new images
docker compose pull

# Example output:
# [+] Pulling 4/4
#  ✔ n8n Pulled          15.2s
#  ✔ traefik Pulled      8.3s
#  ✔ db_core Pulled      12.1s
#  ✔ phpmyadmin Pulled   6.7s

# 3. Recreate all containers with new images
docker compose up -d

# Example output:
# [+] Running 4/4
#  ✔ Container traefik     Started    0.8s
#  ✔ Container db_core     Started    1.2s
#  ✔ Container n8n         Started    1.5s
#  ✔ Container phpmyadmin  Started    0.9s

# 4. Check everything started
docker compose ps

# Example output:
# NAME          IMAGE              STATUS         PORTS
# db_core       mysql:8.0          Up 30 seconds  3306/tcp
# n8n           n8nio/n8n:latest   Up 28 seconds  0.0.0.0:5678->5678/tcp
# phpmyadmin    phpmyadmin:latest  Up 27 seconds  0.0.0.0:8080->80/tcp
# traefik       traefik:v2.10      Up 31 seconds  0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
```

### Update Individual Containers

**Example: Update MySQL**

```bash
cd ~/docker

# 1. Check current version
docker compose exec db_core mysql -V

# Example output:
mysql  Ver 8.4.7 for Linux on x86_64 (MySQL Community Server - GPL)

# 2. Backup database first!
./backup.sh

# 3. Update docker-compose.yml if needed
nano docker-compose.yml

# Change MySQL version:
# image: mysql:8.0    # or mysql:8.0.36 for specific version

# 4. Pull new image
docker compose pull db_core

# 5. Stop database
docker compose stop db_core

# 6. Start with new image
docker compose up -d db_core

# 7. Wait for MySQL to be ready
sleep 15

# 8. Verify database is running
docker compose exec db_core mysql -u root -p -e "SELECT VERSION();"

# Example output:
+-----------+
| VERSION() |
+-----------+
| 8.4.7     |
+-----------+
```

**Example: Update Traefik**

```bash
cd ~/docker

# 1. Check current version
docker compose exec traefik traefik version

# Example output:
Version:      3.6.5
Codename:     ramequin
Go version:   go1.24.11
Built:        2025-12-16T14:56:48Z
OS/Arch:      linux/amd64

# 2. Update in docker-compose.yml
nano docker-compose.yml

# Change:
# image: traefik:v2.11  # or traefik:latest

# 3. Pull and restart
docker compose pull traefik
docker compose stop traefik
docker compose up -d traefik

# 4. Check logs
docker compose logs traefik --tail=30

```

**Example: Update phpMyAdmin**

```bash
cd ~/docker

# Simple update (no data stored in container)
docker compose pull phpmyadmin
docker compose up -d phpmyadmin

```

### Update Script for All Services

```bash
nano ~/docker/update-all.sh
```

```bash
#!/bin/bash

set -e

echo "=== Docker Services Update Script ==="
echo ""

# Confirmation
read -p "⚠️  This will update ALL services. Continue? (yes/no): " confirm

if [ "$confirm" != "yes" ]; then
    echo "❌ Update cancelled"
    exit 0
fi

echo ""
read -p "⚠️  Have you created a backup? (yes/no): " backup_confirm

if [ "$backup_confirm" != "yes" ]; then
    echo "❌ Please create a backup first using: ./backup.sh"
    exit 1
fi

echo ""
echo "📸 Saving current versions..."
docker compose ps --format "{{.Service}}: {{.Image}}" > ~/pre-update-versions.txt
echo "Saved to ~/pre-update-versions.txt"

echo ""
echo "📦 Pulling new images..."
docker compose pull

echo ""
echo "🔄 Recreating containers..."
docker compose up -d

echo ""
echo "⏳ Waiting for services to start..."
sleep 15

echo ""
echo "✅ Checking service status..."
docker compose ps

echo ""
echo "📋 New versions:"
docker compose ps --format "{{.Service}}: {{.Image}}"

echo ""
echo "🎉 Update completed!"
echo ""
echo "Please test your services:"
echo "  - n8n: Visit your n8n URL"
echo "  - Database: docker compose exec db_core mysql -u root -p"
echo "  - Traefik: Check HTTPS certificates"
echo ""
echo "If issues occur, see rollback procedures in the documentation."
```

Make executable:

```bash
chmod +x ~/docker/update-all.sh
./update-all.sh
```

---

## Operating System Updates

### Regular System Updates

> 💡 **TIP:** Run OS updates weekly or monthly depending on your security requirements.

**Step 1: Check for updates**
```bash
# Update package list
sudo apt update

# Example output:
# Hit:1 http://archive.ubuntu.com/ubuntu jammy InRelease
# Get:2 http://security.ubuntu.com/ubuntu jammy-security InRelease [110 kB]
# Fetched 1,234 kB in 2s (617 kB/s)
# Reading package lists... Done
# Building dependency tree... Done
# 15 packages can be upgraded. Run 'apt list --upgradable' to see them.

# See what's available
apt list --upgradable

# Example output:
# Listing...
# curl/jammy-updates 7.81.0-1ubuntu1.15 amd64 [upgradable from: 7.81.0-1ubuntu1.14]
# openssh-server/jammy-updates 1:8.9p1-3ubuntu0.6 amd64 [upgradable from: 1:8.9p1-3ubuntu0.5]
```

**Step 2: Install updates**
```bash
# Install all available updates
sudo apt upgrade -y

# Example output:
# Reading package lists... Done
# Building dependency tree... Done
# Calculating upgrade... Done
# The following packages will be upgraded:
#   curl openssh-server ...
# 15 upgraded, 0 newly installed, 0 to remove and 0 not upgraded.
# Need to get 23.4 MB of archives.
# After this operation, 124 kB of additional disk space will be used.
# Get:1 http://archive.ubuntu.com/ubuntu jammy-updates/main amd64 curl amd64 7.81.0-1ubuntu1.15 [194 kB]
# ...
# Processing triggers for man-db (2.10.2-1) ...
# Processing triggers for libc-bin (2.35-0ubuntu3.4) ...
```

**Step 3: Clean up**
```bash
# Remove old packages
sudo apt autoremove -y

# Example output:
# Reading package lists... Done
# Building dependency tree... Done
# The following packages will be REMOVED:
#   linux-headers-5.15.0-88 linux-image-5.15.0-88
# 0 upgraded, 0 newly installed, 2 to remove and 0 not upgraded.
# After this operation, 342 MB disk space will be freed.

# Clean package cache
sudo apt clean
```

**Step 4: Check if reboot needed**
```bash
# Check if reboot required
if [ -f /var/run/reboot-required ]; then
    echo "⚠️  Reboot required!"
    cat /var/run/reboot-required.pkgs
else
    echo "✅ No reboot required"
fi

# Example output (if reboot needed):
# ⚠️  Reboot required!
# linux-image-5.15.0-89-generic
# linux-base
```

> ⚠️ **IMPORTANT:** If reboot is required, schedule it during a maintenance window. See [12 - Emergency Procedures](12-emergency-procedures.md) for safe reboot process.

### Kernel Updates

Kernel updates require a reboot:

```bash
# Check current kernel
uname -r

# Example output:
# 5.15.0-88-generic

# After kernel update, schedule a reboot
sudo reboot
```

**Pre-reboot checklist:**
- [ ] Backup everything
- [ ] Notify users of downtime
- [ ] Ensure you have console/KVM access (in case of boot issues)
- [ ] Test after reboot that all services start

### Security Updates Only

For critical security patches without full upgrade:

```bash
# Install only security updates (Ubuntu)
sudo apt install unattended-upgrades -y
sudo unattended-upgrades -d

# Or manually install security updates
sudo apt update
sudo apt upgrade -y --only-upgrade $(apt list --upgradable 2>/dev/null | grep security | cut -d/ -f1)
```

### OS Update Script

```bash
nano ~/os-update.sh
```

```bash
#!/bin/bash

echo "=== Operating System Update Script ==="
echo ""

# Check if running as root or with sudo
if [ "$EUID" -ne 0 ]; then 
    echo "❌ Please run with sudo: sudo ./os-update.sh"
    exit 1
fi

echo "1️⃣  Updating package lists..."
apt update

echo ""
echo "2️⃣  Checking for upgradable packages..."
upgradable=$(apt list --upgradable 2>/dev/null | grep -v "Listing" | wc -l)

if [ "$upgradable" -eq 0 ]; then
    echo "✅ System is up to date!"
    exit 0
fi

echo "📦 Found $upgradable package(s) to upgrade"
echo ""

read -p "Show package list? (yes/no): " show_list

if [ "$show_list" == "yes" ]; then
    apt list --upgradable
    echo ""
fi

read -p "Proceed with upgrade? (yes/no): " proceed

if [ "$proceed" != "yes" ]; then
    echo "❌ Update cancelled"
    exit 0
fi

echo ""
echo "3️⃣  Installing updates..."
apt upgrade -y

echo ""
echo "4️⃣  Cleaning up..."
apt autoremove -y
apt clean

echo ""
echo "5️⃣  Checking if reboot required..."
if [ -f /var/run/reboot-required ]; then
    echo "⚠️  REBOOT REQUIRED"
    echo "Packages requiring reboot:"
    cat /var/run/reboot-required.pkgs
    echo ""
    echo "Schedule a reboot soon with: sudo reboot"
else
    echo "✅ No reboot required"
fi

echo ""
echo "✅ System update completed!"
echo "Current kernel: $(uname -r)"
```

Make executable:

```bash
chmod +x ~/os-update.sh
sudo ./os-update.sh
```

---

## Rollback Procedures

### When to Rollback

Rollback if you experience:
- n8n won't start or crashes immediately
- Database connection errors
- Workflows don't execute
- Missing credentials or data
- Performance degradation
- Error messages you didn't have before

> 💡 **TIP:** Don't panic! Docker makes rollbacks relatively easy.

### Rolling Back n8n

**Method 1: Revert to Previous Image Version**

```bash
cd ~/docker

# 1. Stop n8n
docker compose stop n8n

# 2. Edit docker-compose.yml
nano docker-compose.yml

# Change back to previous version:
# image: n8nio/n8n:1.19.4  # (use version from pre-update-versions.txt)

# 3. Remove current container
docker compose rm -f n8n

# 4. Pull the old image (if not cached)
docker compose pull n8n

# 5. Start with old version
docker compose up -d n8n

# 6. Check logs
docker compose logs n8n --tail=50

# 7. Verify version
docker compose exec n8n n8n --version

# Example output:
# 1.19.4  ← Back to previous version
```

**Method 2: Restore from Backup**

If data was corrupted or changed:

```bash
cd ~/docker

# 1. Stop n8n
docker compose down n8n

# 2. Remove current n8n data (only if you have a backup)
docker volume rm docker_n8n_data

# 3. Restore from backup (use your backup filename)
docker run --rm \
  -v docker_n8n_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/n8n-REPLACE_BY_YOUR_VERSION.tar.gz"

# 4. Revert docker-compose.yml to old version
nano docker-compose.yml
# Change image version back

# 5. Start n8n
docker compose up -d n8n

# 6. Verify
docker compose logs n8n --tail=50
```

### Rolling Back Database (MySQL)

> ⚠️ **IMPORTANT:** Database rollbacks are serious. You'll lose any data created since the backup.

```bash
cd ~/docker

# 1. Stop all services using the database
docker compose stop n8n # Stop anything using DB

# 2. Stop database
docker compose stop db_core

# 3. Remove current database volume (WARNING: This deletes current database!)
# Make sure you really want to do this!
docker volume rm docker_mysql_data

# 4. Restore from backup
docker run --rm \
  -v docker_mysql_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/mysql-REPLACE_BY_YOUR_VERSION.tar.gz"

# 5. Start database
docker compose up -d db_core

# 6. Wait for MySQL to initialize
sleep 20

# 7. Verify database is working
docker compose exec db_core mysql -u root -p${MYSQL_ROOT_PWD} -e "SHOW DATABASES;"

# 8. Start other services
docker compose up -d
```

### Rolling Back All Services

Complete rollback to previous state:

```bash
cd ~/docker

# 1. Stop everything
docker compose down

# 2. Restore docker-compose.yml from backup
cp backups/docker-compose-REPLACE_BY_YOUR_VERSION.tar.gz .
tar xzf docker-compose-REPLACE_BY_YOUR_VERSION.tar.gz

# 3. Restore all volumes
# n8n
docker volume rm docker_n8n_data
docker run --rm -v docker_n8n_data:/data -v $(pwd)/backups:/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/n8n-REPLACE_BY_YOUR_VERSION.tar.gz"

# MySQL
docker volume rm docker_mysql_data
docker run --rm -v docker_mysql_data:/data -v $(pwd)/backups:/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/mysql-REPLACE_BY_YOUR_VERSION.tar.gz"

# Traefik
docker volume rm docker_traefik_letsencrypt
docker run --rm -v docker_traefik_letsencrypt:/data -v $(pwd)/backups:/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/traefik-REPLACE_BY_YOUR_VERSION.tar.gz"

# 4. Start everything
docker compose up -d

# 5. Monitor startup
docker compose logs -f
```

### Rollback Script

```bash
nano ~/docker/rollback.sh
```

```bash
#!/bin/bash

set -e

echo "=== Docker Services Rollback Script ==="
echo ""
echo "⚠️  WARNING: This will restore from backup!"
echo "Any data created since the backup will be LOST!"
echo ""

# List available backups
echo "Available backups:"
ls -lht backups/*.tar.gz | head -10

echo ""
read -p "Enter backup date (YYYYMMDD-HHMMSS, e.g., 20240115-143000): " backup_date

if [ -z "$backup_date" ]; then
    echo "❌ No backup date provided"
    exit 1
fi

# Verify backup files exist
if [ ! -f "backups/n8n-${backup_date}.tar.gz" ]; then
    echo "❌ n8n backup not found: backups/n8n-${backup_date}.tar.gz"
    exit 1
fi

echo ""
read -p "Rollback n8n, database, or both? (n8n/db/both): " rollback_type

echo ""
read -p "⚠️  Are you ABSOLUTELY SURE? Type 'YES' to continue: " confirm

if [ "$confirm" != "YES" ]; then
    echo "❌ Rollback cancelled"
    exit 0
fi

echo ""
echo "🔄 Starting rollback..."

if [ "$rollback_type" == "n8n" ] || [ "$rollback_type" == "both" ]; then
    echo "1️⃣  Rolling back n8n..."
    docker compose stop n8n
    docker volume rm docker_n8n_data
    docker run --rm -v docker_n8n_data:/data -v $(pwd)/backups:/backup \
      ubuntu bash -c "cd /data && tar xzf /backup/n8n-${backup_date}.tar.gz"
    docker compose up -d n8n
fi

if [ "$rollback_type" == "db" ] || [ "$rollback_type" == "both" ]; then
    echo "2️⃣  Rolling back database..."
    docker compose stop n8n uptime-kuma db_core
    docker volume rm docker_mysql_data
    docker run --rm -v docker_mysql_data:/data -v $(pwd)/backups:/backup \
      ubuntu bash -c "cd /data && tar xzf /backup/mysql-${backup_date}.tar.gz"
    docker compose up -d db_core
    sleep 20
    docker compose up -d n8n uptime-kuma
fi

echo ""
echo "⏳ Waiting for services to start..."
sleep 15

echo ""
echo "📋 Service status:"
docker compose ps

echo ""
echo "✅ Rollback completed!"
echo "Please verify your services are working correctly."
```

Make executable:

```bash
chmod +x ~/docker/rollback.sh
./rollback.sh
```

---

## Update Schedule & Strategy

### Recommended Update Frequency

| Component | Frequency | Priority | Best Time |
|-----------|-----------|----------|-----------|
| **n8n** | Monthly | High | After major release stabilizes (wait 1-2 weeks) |
| **MySQL** | Quarterly | Medium | During maintenance window |
| **Traefik** | Quarterly | Medium | During maintenance window |
| **OS Security** | Weekly | Critical | Automated or weekly check |
| **OS Full** | Monthly | High | After testing |

### Update Strategy

**Conservative Approach (Recommended for Production)**
```
1. Wait 1-2 weeks after major release
2. Read release notes and changelog
3. Backup everything
4. Update to specific version (not 'latest')
5. Test thoroughly
6. Monitor for 24-48 hours
```

**Aggressive Approach (Testing/Development)**
```
1. Use 'latest' tags
2. Update frequently
3. Accept occasional issues
4. Quick rollback if needed
```

### Monthly Update Routine

Create a monthly update checklist:

```bash
nano ~/monthly-update-checklist.md
```

```markdown
# Monthly Update Checklist

## Preparation (Day 1)
- [ ] Check for available updates
- [ ] Read all release notes and changelogs
- [ ] Check community forums for issues
- [ ] Schedule maintenance window
- [ ] Notify users if applicable

## Backup (Day 2)
- [ ] Run full backup
- [ ] Verify backup files created
- [ ] Test backup integrity
- [ ] Download backup off-server

## Update (Day 3 - During maintenance window)
- [ ] Update OS packages
- [ ] Update Docker containers
- [ ] Update n8n
- [ ] Document versions
- [ ] Check logs for errors

## Testing (Day 3-4)
- [ ] Test n8n workflows
- [ ] Verify database connectivity
- [ ] Check SSL certificates
- [ ] Monitor performance
- [ ] Check disk space

## Monitoring (Day 4-7)
- [ ] Check logs daily
- [ ] Monitor resource usage
- [ ] Verify backups running
- [ ] Watch for errors
```

### Automated Update Notifications

Setup update checking:

```bash
nano ~/check-updates.sh
```

```bash
#!/bin/bash

echo "=== Update Check Report ==="
echo "Generated: $(date)"
echo ""

# Check OS updates
echo "📦 Operating System Updates:"
sudo apt update > /dev/null 2>&1
upgradable=$(apt list --upgradable 2>/dev/null | grep -v "Listing" | wc -l)
if [ "$upgradable" -gt 0 ]; then
    echo "   ⚠️  $upgradable package(s) available"
    apt list --upgradable 2>/dev/null | grep security | head -5
else
    echo "   ✅ System up to date"
fi

# Check Docker images
echo ""
echo "🐳 Docker Image Updates:"
cd ~/docker
for image in $(docker compose config --images); do
    echo "   Checking: $image"
    docker pull $image > /dev/null 2>&1
done

# Check n8n version
echo ""
echo "🔧 n8n Version:"
current=$(docker compose exec -T n8n n8n --version 2>/dev/null)
echo "   Current: $current"
latest=$(curl -s https://api.github.com/repos/n8n-io/n8n/releases/latest | jq -r .tag_name | sed 's/n8n@//')
echo "   Latest: $latest"

if [ "$current" != "$latest" ]; then
    echo "   ⚠️  Update available!"
else
    echo "   ✅ Up to date"
fi

echo ""
echo "=== End Report ==="
```

Add to crontab for weekly checking:

```bash
crontab -e

# Add:
# Weekly update check every Monday at 9 AM
0 9 * * 1 /home/USERNAME/check-updates.sh >> /home/USERNAME/update-check.log 2>&1
```

---

## Troubleshooting Updates

### Update Won't Complete

**Symptom:** `docker compose pull` hangs or fails

```bash
# Check disk space
df -h

# If low, clean up
docker system prune -a

# Try pulling individual images
docker pull n8nio/n8n:latest

# Check Docker daemon
sudo systemctl status docker
```

### Container Won't Start After Update

**Symptom:** Container status shows "Restarting" or "Exited"

```bash
# Check logs immediately
docker compose logs container-name --tail=100

# Common issues:
# - Database migration failed
# - Port already in use
# - Volume permissions changed
# - Configuration incompatible

# Try starting in foreground to see errors
docker compose up container-name
```

### Database Migration Failed

**Symptom:** n8n shows database error after update

```bash
# Check n8n logs
docker compose logs n8n | grep -i "migration\|database"

# Check database is running
docker compose exec db_core mysql -u root -p -e "SHOW DATABASES;"

# If migration stuck, may need manual intervention
# Contact n8n support or check GitHub issues
```

### Lost Credentials After Update

**Symptom:** n8n credentials missing or encrypted differently

```bash
# Check if encryption key changed
docker compose exec n8n env | grep N8N_ENCRYPTION_KEY

# If key missing or wrong, check .env file
cat ~/docker/.env | grep N8N_ENCRYPTION_KEY

# Restore from backup if credentials lost
# See Rollback Procedures above
```

### SSL Certificates Stopped Working

**Symptom:** HTTPS shows certificate error after Traefik update

```bash
# Check Traefik logs
docker compose logs traefik | grep -i "certificate\|acme"

# Check certificate file
docker run --rm -v docker_traefik_letsencrypt:/data ubuntu ls -la /data

# May need to delete and regenerate
docker compose stop traefik
docker volume rm docker_traefik_letsencrypt
docker compose up -d traefik

# Wait 2-3 minutes for new certificates
docker compose logs traefik -f
```

### Performance Issues After Update

**Symptom:** Services slow or unresponsive after update

```bash
# Check resource usage
docker stats --no-stream

# Check logs for errors
docker compose logs --since 1h | grep -i "error\|warning"

# Try restarting
docker compose restart

# Check if new version has higher requirements
# May need to allocate more resources
```

---

## Quick Reference

### Update Commands Cheatsheet

```bash
# n8n
cd ~/docker
./backup.sh
docker compose pull n8n && docker compose up -d n8n

# All containers
cd ~/docker
./backup.sh
docker compose pull && docker compose up -d

# OS updates
sudo apt update && sudo apt upgrade -y

# Check versions
docker compose exec n8n n8n --version
docker compose ps --format "{{.Service}}: {{.Image}}"

# Rollback n8n
docker compose stop n8n
# Edit docker-compose.yml to previous version
docker compose up -d n8n
```

### Pre-Update Checklist

```
✅ Create backup
✅ Check disk space (need 2GB+ free)
✅ Document current versions
✅ Read release notes
✅ Choose low-traffic time
✅ Have rollback plan ready
```

### Post-Update Checklist

```
✅ Check all containers running
✅ Test n8n workflows
✅ Verify database connectivity
✅ Check SSL certificates
✅ Monitor logs for errors
✅ Check resource usage
✅ Verify backups still working
```

### When to Rollback

```
❌ Container won't start
❌ Database errors
❌ Missing/corrupt data
❌ Critical features broken
❌ Performance severely degraded
❌ Security vulnerability introduced
```

---

## Navigation

**Previous Guide:** [07 - Monitoring & Maintenance](07-monitoring-maintenance.md)  
**Next Guide:** [09 - Emergency procedures](09-emergency-procedures.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: 29/12/2025*
