# 06 - Backup & Recovery Guide
**Difficulty Level:** 🟡 Intermediate  
**Risk Level:** 🟠 Medium - *Be careful with restorations*

A practical guide to backing up and restoring your Docker volumes

---

## 📋 Table of Contents
1. [Understanding What to Backup](#understanding-what-to-backup)
2. [Manual Backup Process](#manual-backup-process)
3. [Automated Backups](#automated-backups)
4. [Restoration Process](#restoration-process)
4. [Restoration Script](#restoration-script)
4. [Using the Restoration Script](#using-the-restoration-script)
5. [Backup Best Practices](#backup-best-practices) 

---

## Understanding What to Backup

Your server uses **Docker volumes** to store all important data. These volumes need regular backups.

### Critical Volumes

```bash
# View your volumes
docker volume ls

# You should see:
DRIVER    VOLUME NAME
local     docker_mysql_data           # Database (most critical)
local     docker_n8n_data             # n8n workflows and settings (most critical)
local     docker_n8n_files            # n8n file storage
local     docker_traefik_letsencrypt  # SSL certificates
local     docker_pgadmin_data         # Data pgAdmin
local     docker_postgres_data        # Database (most critical)

```

### What Gets Backed Up

| Volume | Contains                          | Backup Priority |
|--------|-----------------------------------|-----------------|
| `docker_mysql_data` | All databases (your data, etc.)   | **Critical**    |
| `docker_n8n_data` | Workflows, credentials, settings  | **Critical**    |
| `docker_n8n_files` | N8n files                         | High            |
| `docker_traefik_letsencrypt` | SSL certificates                  | Medium          |
| `docker_pgadmin_data` | Data for Postgres admin           | Low             |
| `docker_postgres_data` | All databases. Can contain n8n DB | **Critical**    |

> 💡 **TIP:** Your `docker-compose.yml` and `.env` files also need backup (they live in `/home/USERNAME/docker/`)

---

## Manual Backup Process

### Creating Backups

```bash
# 1. Navigate to your docker folder
cd ~/docker

# 2. Create backup directory
mkdir -p backups

# 3. Backup MySQL database
docker run --rm \
  -v docker_mysql_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu tar czf /backup/mysql-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

# 4. Backup n8n data
docker run --rm \
  -v docker_n8n_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu tar czf /backup/n8n-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

# 5. Backup Traefik certificates
docker run --rm \
  -v docker_traefik_letsencrypt:/data \
  -v $(pwd)/backups:/backup \
  ubuntu tar czf /backup/traefik-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

# 6. Backup Pgadmin
docker run --rm \
  -v docker_pgadmin_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu tar czf /backup/pgadmin-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .

# 7. Backup PostgreSQL
docker run --rm \
  -v docker_postgres_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu tar czf /backup/postgres-$(date +%Y%m%d-%H%M%S).tar.gz -C /data .
  
# 8. Backup docker-compose files
tar czf backups/docker-compose-$(date +%Y%m%d-%H%M%S).tar.gz \
  docker-compose.yml .env my.cnf
```

### Quick Backup Script

Create a file called `backup.sh`:

```bash
nano ~/docker/maintenance/backup.sh
```

Paste this content:

```bash
#!/bin/bash

# Backup script for Docker volumes
BACKUP_DIR="$HOME/docker/backups"
DATE=$(date +%Y%m%d-%H%M%S)

# Create backup directory
mkdir -p "$BACKUP_DIR"

echo "Starting backup at $(date)"

# Backup MySQL
echo "Backing up MySQL..."
docker run --rm \
  -v docker_mysql_data:/data \
  -v "$BACKUP_DIR":/backup \
  ubuntu tar czf /backup/mysql_data-$DATE.tar.gz -C /data .

# Backup n8n data
echo "Backing up n8n data..."
docker run --rm \
  -v docker_n8n_data:/data \
  -v "$BACKUP_DIR":/backup \
  ubuntu tar czf /backup/n8n_data-$DATE.tar.gz -C /data .

# Backup n8n files
echo "Backing up n8n files..."
docker run --rm \
  -v docker_n8n_files:/data \
  -v "$BACKUP_DIR":/backup \
  ubuntu tar czf /backup/n8n_files-$DATE.tar.gz -C /data .

# Backup Traefik
echo "Backing up Traefik..."
docker run --rm \
  -v docker_traefik_letsencrypt:/data \
  -v "$BACKUP_DIR":/backup \
  ubuntu tar czf /backup/traefik_letsencrypt-$DATE.tar.gz -C /data .

# Backup Pgadmin
echo "Backing up Pgadmin..."
docker run --rm \
  -v docker_pgadmin_data:/data \
  -v "$BACKUP_DIR":/backup \
  ubuntu tar czf /backup/pgadmin_data-$DATE.tar.gz -C /data .

# Backup PostgreSQL
echo "Backing up PostgreSQL..."
docker run --rm \
  -v docker_postgres_data:/data \
  -v "$BACKUP_DIR":/backup \
  ubuntu tar czf /backup/postgres_data-$DATE.tar.gz -C /data .

# Backup compose files
echo "Backing up docker-compose files..."
cd ~/docker
tar czf "$BACKUP_DIR/docker-compose-$DATE.tar.gz" docker-compose.yml .env my.cnf

echo "Backup completed at $(date)"
echo "Backups saved to: $BACKUP_DIR"
```

Make it executable:

```bash
chmod +x ~/docker/maintenance/backup.sh
```

Run it:

```bash
~/docker/maintenance/backup.sh
```

---

## Automated Backups

### Setting Up Daily Automated Backups

```bash
# 1. Open crontab editor
crontab -e

# 2. Add this line (runs daily at 2 AM)
0 2 * * * /home/USERNAME/docker/maintenance/backup.sh >> /home/USERNAME/docker/backups/backup.log 2>&1

# 3. Save and exit (Ctrl+X, then Y, then Enter)
```

### Verify Cron Job

```bash
# Check if cron job is scheduled
crontab -l

# Check backup logs
tail -f ~/docker/backups/backup.log
```

---

## Restoration Process

### Restoring from Backup

> ⚠️ **IMPORTANT:** This will overwrite existing data. Stop containers first!

```bash
# 1. Stop all containers
cd ~/docker
docker compose down

# 2. List your backups
ls -lh backups/

# 3. Restore MySQL (replace DATE with your backup date)
docker run --rm \
  -v docker_mysql_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "rm -rf /data/* && tar xzf /backup/mysql_data-DATE.tar.gz -C /data"

# 4. Restore n8n data
docker run --rm \
  -v docker_n8n_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "rm -rf /data/* && tar xzf /backup/n8n_data-DATE.tar.gz -C /data"

# 5. Restore Traefik
docker run --rm \
  -v docker_traefik_letsencrypt:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "rm -rf /data/* && tar xzf /backup/traefik_letsencrypt-DATE.tar.gz -C /data"

# 6. Restore Pgadmin
docker run --rm \
  -v docker_pgadmin_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "rm -rf /data/* && tar xzf /backup/pgadmin_data-DATE.tar.gz -C /data"

# 7. Restore Traefik
docker run --rm \
  -v docker_postgres_data:/data \
  -v $(pwd)/backups:/backup \
  ubuntu bash -c "rm -rf /data/* && tar xzf /backup/postgres_data-DATE.tar.gz -C /data"
  
# 8. Restart containers
docker compose up -d
```

### Testing a Restoration

Before you need it, test your backups work:

```bash
# 1. Create test volumes
docker volume create test_postgres_restore

# 2. Restore to test volumes (replace LATEST with your backup date)
docker run --rm \
  -v test_postgres_restore:/data \
  -v $(pwd)/backups:/backup \
  ubuntu tar xzf /backup/postgres_data-LATEST.tar.gz -C /data

# 3. Verify contents
docker run --rm \
  -v test_postgres_restore:/data \
  ubuntu ls -lah /data

# 4. Clean up test volumes
docker volume rm test_postgres_restore
```

## Restoration Script

Create a restoration script for flexible recovery:

```bash
nano ~/docker/maintenance/restore.sh
```

**Add this content:**

```bash
#!/bin/bash

#############################################
# Docker Volume Restoration Script
# Restores individual volumes from backups
#############################################

set -e  # Exit on any error

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

# Configuration
DOCKER_DIR="$HOME/docker"
BACKUP_DIR="$HOME/docker/backups"

echo -e "${GREEN}==================================${NC}"
echo -e "${GREEN}Docker Volume Restoration Script${NC}"
echo -e "${GREEN}==================================${NC}"
echo

# Function to show usage
show_usage() {
    echo "Usage: $0 <volume-name> <backup-file>"
    echo
    echo "Available volumes:"
    echo "  n8n_data            - n8n workflows and settings"
    echo "  n8n_files           - n8n files"
    echo "  mysql_data          - MySQL database"
    echo "  traefik_letsencrypt - SSL certificates"
    echo "  pgadmin_data        - Pgadmin data"
    echo "  postgres_data       - PostgreSQL database"
    echo
    echo "Examples:"
    echo "  $0 n8n_data $BACKUP_DIR/n8n_data-20260107-024827.tar.gz"
    echo "  $0 postgres_data $BACKUP_DIR/postgres_data-20260107-024827.tar.gz"
    echo
    echo "Available backups:"
    ls -lh "$BACKUP_DIR"/*.tar.gz 2>/dev/null | awk '{print "  " $9 " (" $5 ")"}'
}

# Check if arguments are provided
if [ -z "$1" ] || [ -z "$2" ]; then
    echo -e "${RED}Error: Missing required arguments${NC}"
    echo
    show_usage
    exit 1
fi

VOLUME_NAME="$1"
BACKUP_FILE="$2"

# Add docker_ prefix if not present
if [[ ! "$VOLUME_NAME" =~ ^docker_ ]]; then
    FULL_VOLUME_NAME="docker_${VOLUME_NAME}"
else
    FULL_VOLUME_NAME="$VOLUME_NAME"
fi

# Check if backup file exists
if [ ! -f "$BACKUP_FILE" ]; then
    echo -e "${RED}Error: Backup file not found: $BACKUP_FILE${NC}"
    echo
    show_usage
    exit 1
fi

# Check if volume exists
if ! docker volume inspect "$FULL_VOLUME_NAME" &>/dev/null; then
    echo -e "${RED}Error: Volume '$FULL_VOLUME_NAME' does not exist${NC}"
    echo
    echo "Available volumes:"
    docker volume ls --filter name=docker_ --format "  {{.Name}}"
    exit 1
fi

echo -e "${BLUE}Volume to restore: ${NC}$FULL_VOLUME_NAME"
echo -e "${BLUE}Backup file: ${NC}$BACKUP_FILE"
echo -e "${BLUE}Backup size: ${NC}$(du -h "$BACKUP_FILE" | cut -f1)"
echo -e "${BLUE}Backup date: ${NC}$(stat -c "%y" "$BACKUP_FILE" 2>/dev/null | cut -d'.' -f1)"
echo

# Determine which containers use this volume
AFFECTED_CONTAINERS=()
case "$VOLUME_NAME" in
    *n8n*)
        AFFECTED_CONTAINERS=("n8n")
        ;;
    *mysql*)
        AFFECTED_CONTAINERS=("db_core")
        ;;
    *traefik*)
        AFFECTED_CONTAINERS=("traefik")
        ;;
    *pgadmin*)
        AFFECTED_CONTAINERS=("pgadmin")
        ;;
    *postgres*)
        AFFECTED_CONTAINERS=("postgres")
        ;;
esac

if [ ${#AFFECTED_CONTAINERS[@]} -gt 0 ]; then
    echo -e "${YELLOW}This will affect the following containers:${NC}"
    for container in "${AFFECTED_CONTAINERS[@]}"; do
        echo "  - $container"
    done
    echo
fi

# Warning
echo -e "${RED}⚠️  WARNING ⚠️${NC}"
echo -e "${RED}This will OVERWRITE all current data in $FULL_VOLUME_NAME!${NC}"
echo -e "${RED}Current data will be permanently lost.${NC}"
echo
read -p "Are you sure you want to continue? (type 'yes' to proceed): " confirm

if [ "$confirm" != "yes" ]; then
    echo -e "${YELLOW}Restoration cancelled.${NC}"
    exit 0
fi

echo
echo -e "${GREEN}Starting restoration process...${NC}"
echo

# Step 1: Stop affected containers
if [ ${#AFFECTED_CONTAINERS[@]} -gt 0 ]; then
    echo -e "${YELLOW}[1/5] Stopping affected containers...${NC}"
    cd "$DOCKER_DIR"
    for container in "${AFFECTED_CONTAINERS[@]}"; do
        if docker compose config --services | grep -q "^${container}$"; then
            echo "  → Stopping $container..."
            docker compose stop "$container" 2>/dev/null || true
        fi
    done
    echo -e "${GREEN}✓ Containers stopped${NC}"
    echo
    STEP=2
else
    echo -e "${YELLOW}[1/5] No containers need to be stopped${NC}"
    echo
    STEP=2
fi

# Step 2: Verify volume is not in use
echo -e "${YELLOW}[$STEP/5] Verifying volume is not in use...${NC}"
VOLUME_IN_USE=$(docker ps --filter volume="$FULL_VOLUME_NAME" --format "{{.Names}}" | wc -l)
if [ "$VOLUME_IN_USE" -gt 0 ]; then
    echo -e "${RED}Error: Volume is still in use by running containers:${NC}"
    docker ps --filter volume="$FULL_VOLUME_NAME" --format "  - {{.Names}}"
    echo
    echo "Please stop these containers first or use: docker compose down"
    exit 1
fi
echo -e "${GREEN}✓ Volume is not in use${NC}"
echo
STEP=$((STEP + 1))

# Step 3: Backup current data (safety measure)
echo -e "${YELLOW}[$STEP/5] Creating safety backup of current data...${NC}"
SAFETY_BACKUP="$BACKUP_DIR/SAFETY-${VOLUME_NAME}-$(date +%Y%m%d-%H%M%S).tar.gz"
docker run --rm \
    -v ${FULL_VOLUME_NAME}:/data:ro \
    -v "$BACKUP_DIR":/backup \
    ubuntu tar czf /backup/$(basename "$SAFETY_BACKUP") -C /data .
echo -e "${GREEN}✓ Safety backup created: $SAFETY_BACKUP${NC}"
echo -e "  ${BLUE}(You can delete this later if restoration is successful)${NC}"
echo
STEP=$((STEP + 1))

# Step 4: Clear and restore volume
echo -e "${YELLOW}[$STEP/5] Restoring volume from backup...${NC}"
echo "  → Clearing current data..."
docker run --rm \
    -v ${FULL_VOLUME_NAME}:/data \
    ubuntu bash -c "rm -rf /data/* /data/..?* /data/.[!.]* 2>/dev/null || true"

echo "  → Extracting backup..."
docker run --rm \
    -v ${FULL_VOLUME_NAME}:/data \
    -v "$(dirname "$BACKUP_FILE")":/backup:ro \
    ubuntu tar xzf /backup/$(basename "$BACKUP_FILE") -C /data

echo -e "${GREEN}✓ Volume restored${NC}"
echo
STEP=$((STEP + 1))

# Step 5: Restart affected containers
if [ ${#AFFECTED_CONTAINERS[@]} -gt 0 ]; then
    echo -e "${YELLOW}[$STEP/5] Starting affected containers...${NC}"
    cd "$DOCKER_DIR"
    for container in "${AFFECTED_CONTAINERS[@]}"; do
        if docker compose config --services | grep -q "^${container}$"; then
            echo "  → Starting $container..."
            docker compose up -d "$container"
        fi
    done

    # Wait for containers to initialize
    echo "  → Waiting for services to initialize..."
    sleep 5

    echo -e "${GREEN}✓ Containers started${NC}"
    echo

    # Show container status
    echo -e "${GREEN}Container Status:${NC}"
    for container in "${AFFECTED_CONTAINERS[@]}"; do
        if docker compose ps --services | grep -q "^${container}$"; then
            docker compose ps "$container"
        fi
    done
else
    echo -e "${YELLOW}[$STEP/5] No containers to restart${NC}"
fi

echo
echo -e "${GREEN}==================================${NC}"
echo -e "${GREEN}Restoration Complete!${NC}"
echo -e "${GREEN}==================================${NC}"
echo
echo "Volume restored: $FULL_VOLUME_NAME"
echo "From backup: $BACKUP_FILE"
echo "Safety backup: $SAFETY_BACKUP"
echo
echo "Next steps:"
echo "1. Test that the service works correctly"
echo "2. Check logs for any errors:"
for container in "${AFFECTED_CONTAINERS[@]}"; do
    echo "   docker compose logs $container"
done
echo "3. If everything works, you can delete the safety backup:"
echo "   rm $SAFETY_BACKUP"
echo
echo "If you encounter issues:"
echo "- Restore the safety backup: $0 $VOLUME_NAME $SAFETY_BACKUP"
echo "- Check container logs for errors"
echo "- Contact your administrator if needed"
```

**Save and make it executable:**

```bash
chmod +x ~/docker/maintenance/restore.sh
```

---

## Using the Restoration Script

### Basic Usage

```bash
# Restore a specific volume
~/docker/maintenance/restore.sh n8n_data ~/docker/backups/n8n_data-20260107-024827.tar.gz
~/docker/maintenance/restore.sh mysql_data ~/docker/backups/mysql_data-20260107-024827.tar.gz
```

### List Available Backups

```bash
# See all backups
ls -lh ~/docker/backups/

# See backups for a specific volume
ls -lh ~/docker/backups/n8n_data-*.tar.gz
ls -lh ~/docker/backups/mysql_data-*.tar.gz
```

### Example Restoration Session

```bash
$ ./maintenance/restore.sh mysql_data /home/USERNAME/docker/backups/mysql_data-20260107-045503.tar.gz
==================================
Docker Volume Restoration Script
==================================

Volume to restore: docker_mysql_data
Backup file: /home/USERNAME/docker/backups/mysql_data-20260107-045503.tar.gz
Backup size: 7.3M
Backup date: 2026-01-07 04:55:08

This will affect the following containers:
  - db_core

⚠️  WARNING ⚠️
This will OVERWRITE all current data in docker_mysql_data!
Current data will be permanently lost.

Are you sure you want to continue? (type 'yes' to proceed): yes

Starting restoration process...

docker dir : /home/USERNAME/docker
[1/5] Stopping affected containers...
  → Stopping db_core...
✓ Containers stopped

[2/5] Verifying volume is not in use...
✓ Volume is not in use

[3/5] Creating safety backup of current data...
✓ Safety backup created: /home/USERNAME/docker/backups/SAFETY-mysql_data-20260107-060333.tar.gz
  (You can delete this later if restoration is successful)

[4/5] Restoring volume from backup...
  → Clearing current data...
  → Extracting backup...
✓ Volume restored

[5/5] Starting affected containers...
  → Starting db_core...
[+] start 1/1
 ✔ Container db_core Started                                                                                                                                                                         0.3s
  → Waiting for services to initialize...
✓ Containers started

Container Status:
NAME      IMAGE       COMMAND                  SERVICE   CREATED        STATUS         PORTS
db_core   mysql:8.4   "docker-entrypoint.s…"   db_core   14 hours ago   Up 5 seconds   3306/tcp, 33060/tcp

==================================
Restoration Complete!
==================================

Volume restored: docker_mysql_data
From backup: /home/USERNAME/docker/backups/mysql_data-20260107-045503.tar.gz
Safety backup: /home/USERNAME/docker/backups/SAFETY-mysql_data-20260107-060333.tar.gz

Next steps:
1. Test that the service works correctly
2. Check logs for any errors:
   docker compose logs db_core
3. If everything works, you can delete the safety backup:
   rm /home/USERNAME/docker/backups/SAFETY-mysql_data-20260107-060333.tar.gz

If you encounter issues:
- Restore the safety backup: ./maintenance/restore.sh mysql_data /home/mehdi/docker/backups/SAFETY-mysql_data-20260107-060333.tar.gz
- Check container logs for errors
- Contact your administrator if needed
```
---

## Backup Best Practices

### Storage Recommendations

✅ **Do:**
- Keep at least 7 daily backups
- Store backups on a different server or cloud storage
- Test restoration regularly (monthly)
- Backup before major changes
- Keep backups of `docker-compose.yml` and `.env`

❌ **Don't:**
- Keep backups only on the same server
- Forget to test restoring
- Delete old backups without keeping some history
- Skip backing up environment files

### Backup Retention Script

Add this to your `backup.sh` to automatically clean old backups:

```bash
# At the end of backup.sh, add:

# Keep only last 7 days of backups
echo "Cleaning old backups..."
find "$BACKUP_DIR" -name "*.tar.gz" -mtime +7 -delete
echo "Cleanup completed"
```

### Off-Site Backup Options

**Option 1: SFTP to Another Server**
```bash
# Install lftp
sudo apt install lftp -y

# Add to backup.sh
lftp -c "open -u username,password sftp://backup-server.com; \
  mput -O /remote/backup/path $BACKUP_DIR/*.tar.gz"
```

**Option 2: Rsync to Remote Server**
```bash
# Add to backup.sh
rsync -avz ~/docker/backups/ USERNAME@backup-server:/backups/docker/
```

---

## Quick Reference

### Backup Commands
```bash
# Manual backup
~/docker/maintenance/backup.sh

# Check backup size
du -sh ~/docker/backups/

# List backups
ls -lht ~/docker/backups/

# Test cron schedule
crontab -l
```

### Emergency Restoration
```bash
# 1. Stop everything
docker compose down

# 2. Restore volumes (replace DATE)
docker run --rm -v docker_mysql_data:/data -v $(pwd)/backups:/backup ubuntu tar xzf /backup/mysql-DATE.tar.gz -C /data
docker run --rm -v docker_n8n_data:/data -v $(pwd)/backups:/backup ubuntu tar xzf /backup/n8n-DATE.tar.gz -C /data

# 3. Start everything
docker compose up -d
```

---

## Troubleshooting

### Backup File is Too Large
```bash
# Check what's taking space
docker run --rm -v docker_mysql_data:/data ubuntu du -sh /data/*
```

### Backup Failed
```bash
# Check if volume exists
docker volume ls | grep mysql

# Check disk space
df -h

# Check backup log
cat ~/docker/backups/backup.log
```

### Cannot Restore Backup
```bash
# Verify backup file integrity
tar -tzf backups/mysql-DATE.tar.gz | head

# Check volume permissions
docker run --rm -v docker_mysql_data:/data ubuntu ls -la /data
```

---
**Next Guide:** [07 - Monitoring & Maintenance](07-monitoring-maintenance.md)  
**Previous Guide:** [06 - Backup & Recovery](06-backup-recovery.md)  
**Back to:** [Documentation Home](README.md)  
---

*Last Updated: [07/01/2026]*
