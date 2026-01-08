# 07 - Monitoring & Maintenance

**Difficulty Level:** 🟡 Intermediate  
**Risk Level:** 🟢 Safe - Monitoring commands don't change anything  

> ⚠️ **IMPORTANT:** Regular monitoring helps you catch problems BEFORE they become emergencies. Spending 5 minutes daily can save hours of troubleshooting later.  

This guide covers routine monitoring and maintenance tasks to keep your server healthy. Think of this as your daily/weekly server checkup checklist.

### What You'll Monitor

- **Disk Space**: Prevent the server from running out of storage
- **CPU & Memory**: Detect performance issues early
- **Service Health**: Ensure all containers are running properly
- **System Logs**: Spot errors and unusual activity

---

## 📋 Table of Contents
1. [Daily Health Checks](#daily-health-checks)
2. [Disk Space Management](#disk-space-management)
3. [CPU & Memory Monitoring](#cpu--memory-monitoring)
4. [Service Health Checks](#service-health-checks)
5. [System Logs](#system-logs)
6. [Automated Monitoring](#automated-monitoring)
7. [Maintenance Schedule](#maintenance-schedule)
8. [Quick Reference](#quick-reference)

---

## Daily Health Checks

### Quick Health Check Routine (5 minutes)

Run these commands each day (or whenever you log in):

```bash
# 1. Check all containers are running
docker ps

# Expected output:
CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS       PORTS                                      NAMES
b89dcc467175   phpmyadmin:5.2            "/docker-entrypoint.…"   17 hours ago   Up 7 hours   80/tcp                                     phpmy
27b635555c20   dpage/pgadmin4:9          "/entrypoint.sh"         17 hours ago   Up 7 hours   80/tcp, 443/tcp                            pgadmin
3ef0351fce81   docker.n8n.io/n8nio/n8n   "tini -- /docker-ent…"   17 hours ago   Up 7 hours   127.0.0.1:5678->5678/tcp                   n8n
d96aefe03d77   postgres:17               "docker-entrypoint.s…"   17 hours ago   Up 7 hours   5432/tcp                                   postgres
a6006df13d7f   mysql:8.4                 "docker-entrypoint.s…"   17 hours ago   Up 7 hours   3306/tcp, 33060/tcp                        db_core
d878b558c411   traefik                   "/entrypoint.sh --ap…"   17 hours ago   Up 7 hours   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   traefik
```

> 💡 **TIP:** Look for the "STATUS" column. It should say "Up X days" or "Up X hours". If you see "Restarting" or "Exited", something is wrong.

```bash
# 2. Check disk space
df -h

# Expected output:
Filesystem      Size  Used Avail Use% Mounted on
tmpfs           392M  1.3M  391M   1% /run
/dev/vda1        38G  7.6G   29G  22% /
tmpfs           2.0G     0  2.0G   0% /dev/shm
tmpfs           5.0M     0  5.0M   0% /run/lock
tmpfs           392M  8.0K  392M   1% /run/user/1000
```

> ⚠️ **IMPORTANT:** If "Use%" is above 80%, you need to free up space soon or contact me to upgrade your server. Above 90% is critical!

```bash
# 3. Check memory usage
free -h

# Expected output:
               total        used        free      shared  buff/cache   available
Mem:           3.8Gi       1.5Gi       979Mi        39Mi       1.6Gi       2.3Gi
Swap:             0B          0B          0B
```

> 💡 **TIP:** "available" memory should be at least 500Mo. If it's consistently low, you may need to upgrade your server or optimize services.

---

## Disk Space Management

### Checking Disk Usage

```bash
# Overall disk space
df -h /

# Example output:
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        38G  7.6G   29G  22% /
```

```bash
# What's using the most space in your home directory?
du -sh ~/* | sort -h

# Example output:
150M    /home/USERNAME/docker
2.3G    /home/USERNAME/backups
450K    /home/USERNAME/.ssh
```

```bash
# Check Docker disk usage
docker system df

# Example output:
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          7         6         5.42GB    5.301GB (97%)
Containers      6         6         62.57MB   0B (0%)
Local Volumes   6         6         449.9MB   0B (0%)
Build Cache     0         0         0B        0B
```

### Cleaning Up Disk Space

> ⚠️ **IMPORTANT:** Always backup before cleaning up Docker resources!

```bash
# Remove unused Docker images (safe)
docker image prune -a

# You'll see:
WARNING! This will remove all images without at least one container associated to them.
Are you sure you want to continue? [y/N] y
Deleted Images:
untagged: ubuntu:latest
deleted: sha256:c35e29c9450151419d9448b0fd75374fec4fff364a27f176fb458d472dfc9e54
```

> 💡 **TIP:** Run this monthly to remove old, unused images.

```bash
# Remove old log files (be careful!)
sudo journalctl --vacuum-time=7d

# Output:
Vacuuming done, freed 250.0M of archived journals from /var/log/journal
```

```bash
# Clean old backups (older than 30 days)
find ~/docker/backups -name "*.tar.gz" -mtime +30 -delete

# Check what would be deleted first:
find ~/docker/backups -name "*.tar.gz" -mtime +30 -ls
```

### Disk Space Alerts

Create a simple disk space monitoring script:

```bash
nano ~/docker/maintenance/check-disk.sh
```

Paste this content:

```bash
#!/bin/bash

# Disk space alert threshold (percentage)
THRESHOLD=80

# Get current disk usage
USAGE=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

if [ "$USAGE" -gt "$THRESHOLD" ]; then
    echo "⚠️  WARNING: Disk usage is at ${USAGE}%"
    echo "Available space:"
    df -h /
    echo ""
    echo "Top space consumers:"
    du -sh ~/* | sort -h | tail -5
else
    echo "✅ Disk usage OK: ${USAGE}%"
fi
```

Make it executable and run it:

```bash
chmod +x ~/docker/maintenance/check-disk.sh
~/docker/maintenance/check-disk.sh

# Example output:
✅ Disk usage OK: 22%
```

---

## CPU & Memory Monitoring

### Quick CPU & Memory Check

```bash
# See what's using CPU and memory right now
top

# Press 'q' to exit
```

**Understanding `top` output:**
```
top - 09:04:47 up  7:00,  2 users,  load average: 0.01, 0.01, 0.00
Tasks: 148 total,   1 running, 147 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.5 us,  0.5 sy,  0.0 ni, 98.8 id,  0.0 wa,  0.0 hi,  0.0 si,  0.2 st
MiB Mem :   3915.9 total,    957.3 free,   1580.3 used,   1665.4 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   2335.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
  12537 999       20   0 2254068 494684  36992 S   1.3  12.3   2:11.48 mysqld
     37 root      20   0       0      0      0 S   0.3   0.0   0:01.52 kcompactd0
```

> 💡 **TIP:** Look at the "load average" numbers. They should be below the number of CPU cores. For a 2-core server, load should be below 2.0. For 4 cores, below 4.0.

### Better Monitoring with htop

```bash
# Install htop (more user-friendly than top)
sudo apt install htop -y

# Run it
htop

# Press F10 or 'q' to exit
```

**What to look for in htop:**
- CPU bars: Should mostly be green (normal), not red (high usage)
- Memory bar: Should have some green (free) space
- Load average: Numbers on top-right should be reasonable

### Docker Container Resource Usage

```bash
# See CPU/memory usage per container
docker stats

# Example output (updates every second):
CONTAINER ID   NAME       CPU %     MEM USAGE / LIMIT     MEM %     NET I/O           BLOCK I/O         PIDS
b89dcc467175   phpmy      0.01%     117.9MiB / 3.824GiB   3.01%     666kB / 1.29MB    70.7MB / 1.98MB   11
27b635555c20   pgadmin    0.04%     361.4MiB / 3.824GiB   9.23%     2.04MB / 41.2MB   173MB / 4.42MB    15
3ef0351fce81   n8n        0.30%     407MiB / 3.824GiB     10.39%    6.32MB / 5.79MB   240MB / 47.6MB    20
d96aefe03d77   postgres   0.00%     66.67MiB / 3.824GiB   1.70%     5.81MB / 6.5MB    49.1MB / 4.02MB   7
a6006df13d7f   db_core    1.69%     449.9MiB / 3.824GiB   11.49%    222B / 126B       43.6MB / 17.8MB   34
d878b558c411   traefik    0.00%     170.3MiB / 3.824GiB   4.35%     44.5MB / 49.1MB   162MB / 0B        8
```

> 💡 **TIP:** Press Ctrl+C to exit. Look for any container using close to 100% CPU consistently - that's unusual.

```bash
# Quick snapshot (doesn't update)
docker stats --no-stream
```

### Memory Usage Check Script

Create a monitoring script:

```bash
nano ~/docker/maintenance/check-resources.sh
```

Paste this:

```bash
#!/bin/bash

echo "=== System Resources Check ==="
echo ""

# CPU Load
echo "📊 CPU Load Average:"
uptime | awk '{print "   " $8 $9 $10 $11 $12}'

# Memory
echo ""
echo "💾 Memory Usage:"
free -h | awk 'NR==2 {printf "   Used: %s / %s (%.0f%%)\n", $3, $2, ($3/$2)*100}'

# Disk
echo ""
echo "💿 Disk Usage:"
df -h / | awk 'NR==2 {printf "   Used: %s / %s (%s)\n", $3, $2, $5}'

# Docker containers
echo ""
echo "🐳 Docker Containers:"
docker ps --format "   {{.Names}}: {{.Status}}" 2>/dev/null || echo "   Docker not accessible"

echo ""
echo "✅ Check completed at $(date)"
```

Make it executable:

```bash
chmod +x ~/docker/maintenance/check-resources.sh
~/docker/maintenance/check-resources.sh

# Example output:
=== System Resources Check ===

📊 CPU Load Average:
   0.00,0.00,0.00

💾 Memory Usage:
   Used: 1.5Gi / 3.8Gi (39%)

💿 Disk Usage:
   Used: 7.6G / 38G (22%)

🐳 Docker Containers:
phpmy: Up 7 hours
pgadmin: Up 7 hours
n8n: Up 7 hours
postgres: Up 7 hours
db_core: Up 3 hours
traefik: Up 7 hours

✅ Check completed at Wed Jan  7 09:07:19 CET 2026
```
---

## Service Health Checks

### Checking Container Status

```bash
# List all containers with detailed status (in ~/docker directory)
docker compose ps

# Example output:
NAME       IMAGE                     COMMAND                  SERVICE    CREATED        STATUS       PORTS
db_core    mysql:8.4                 "docker-entrypoint.s…"   db_core    17 hours ago   Up 7 hours   3306/tcp, 33060/tcp
n8n        docker.n8n.io/n8nio/n8n   "tini -- /docker-ent…"   n8n        17 hours ago   Up 7 hours   127.0.0.1:5678->5678/tcp
pgadmin    dpage/pgadmin4:9          "/entrypoint.sh"         pgadmin    17 hours ago   Up 7 hours   80/tcp, 443/tcp
phpmy      phpmyadmin:5.2            "/docker-entrypoint.…"   phpmy      17 hours ago   Up 7 hours   80/tcp
postgres   postgres:17               "docker-entrypoint.s…"   postgres   17 hours ago   Up 7 hours   5432/tcp
traefik    traefik                   "/entrypoint.sh --ap…"   traefik    17 hours ago   Up 7 hours   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
```

> 💡 **TIP:** "Up X days" is good. "Restarting" or "Exited" means there's a continuous problem.

### Individual Container Health

```bash
# Check specific container
docker inspect n8n --format='{{.State.Status}}'

# Expected output:
# running
```

```bash
# Check when container started
docker inspect n8n --format='{{.State.StartedAt}}'

# Example output:
# 2024-01-13T10:23:45.123456789Z
```

```bash
# Check if container has restarted recently
docker inspect n8n --format='{{.RestartCount}}'

# Expected output:
# 0
```

> ⚠️ **IMPORTANT:** If RestartCount is high (>5), the container is crashing and restarting. Check logs immediately.

### Testing Service Connectivity

```bash
# Test if n8n is responding
curl -I http://localhost:5678

# Expected output:
# HTTP/1.1 200 OK
# Content-Type: text/html; charset=utf-8
# ...
```

```bash
# Test if MySQL is responding (from inside network)
docker compose exec db_core mysqladmin -u root -p ping

# Expected output:
# mysqld is alive
```

```bash
# Test Traefik dashboard (if enabled)
curl -I https://traefik.yourdomain.website

# Expected output:
HTTP/2 401 # <= 401 is because the page is protected behind a basic auth.
content-type: text/plain
www-authenticate: Basic realm="traefik"
# ...
```

### Service Health Check Script

```bash
nano ~/docker/maintenance/check-services.sh
```

Paste this:

```bash
#!/bin/bash

echo "=== Service Health Check ==="
echo ""

# Check if Docker is running
if ! docker info > /dev/null 2>&1; then
    echo "❌ Docker is not running!"
    exit 1
fi

echo "✅ Docker is running"
echo ""

# Check each service
services=("traefik" "n8n" "db_core" "phpmy" "pgadmin" "postgres")

for service in "${services[@]}"; do
    status=$(docker inspect "$service" --format='{{.State.Status}}' 2>/dev/null)
    restarts=$(docker inspect "$service" --format='{{.RestartCount}}' 2>/dev/null)
    
    if [ "$status" == "running" ]; then
        echo "✅ $service: Running (Restarts: $restarts)"
    else
        echo "❌ $service: $status (Restarts: $restarts)"
    fi
done

echo ""
echo "Check completed at $(date)"
```

Make executable and run:

```bash
chmod +x ~/docker/maintenance/check-services.sh
~/docker/maintenance/check-services.sh

# Example output:
=== Service Health Check ===

✅ Docker is running

✅ traefik: Running (Restarts: 0)
✅ n8n: Running (Restarts: 0)
✅ db_core: Running (Restarts: 0)
✅ phpmy: Running (Restarts: 0)
✅ pgadmin: Running (Restarts: 0)
✅ postgres: Running (Restarts: 0)

Check completed at Wed Jan  7 09:12:50 CET 2026

```

---

## System Logs

### Viewing Docker Container Logs

```bash
# View recent logs from n8n
docker compose logs n8n --tail=5

# Example output:
n8n  | Registered runner "JS Task Runner" (-aaqJtHGhMG6wDBzBJTaQ)
n8n  | Version: 2.2.3
n8n  |
n8n  | Editor is now accessible via:
n8n  | https://n8n.yourdomain.website
```

```bash
# Follow logs in real-time (Ctrl+C to exit)
docker compose logs -f n8n

# You'll see logs as they happen
```

```bash
# View logs from all containers
docker compose logs --tail=20

# Shows last 20 lines from each container
```

```bash
# View logs from specific time
docker compose logs --since 1h n8n

# Shows logs from last hour
```

```bash
# Search logs for errors
docker compose logs n8n | grep -i error

# Example output:
# n8n  | 2024-01-15 12:45:33 | ERROR | Failed to connect to webhook endpoint
# n8n  | 2024-01-15 13:22:11 | ERROR | Workflow execution timeout
```

> 💡 **TIP:** Common search terms: `error`, `warning`, `fail`, `timeout`, `exception`

### System Logs with journalctl

```bash
# View Docker service logs
sudo journalctl -u docker --since today

# Example output:
# Jan 15 10:23:45 server dockerd[1234]: time="2024-01-15T10:23:45Z" level=info msg="Container started"
```

```bash
# View last 50 lines of system logs
sudo journalctl -n 50

# Follow system logs in real-time
sudo journalctl -f
```

```bash
# Check for system errors
sudo journalctl -p err --since today

# Shows only error-level logs from today
```

### Log Monitoring Script

```bash
nano ~/docker/maintenance/check-logs.sh
```

Paste this:

```bash
#!/bin/bash

echo "=== Recent Log Issues ==="
echo ""

# Check for errors in last hour
echo "🔍 Checking for errors in last 24h..."
echo ""

# Docker logs
error_logs=$(docker compose logs --since 24h 2>&1 | grep -i "error\|critical\|fatal")
errors=$(echo "$error_logs" | wc -l)

if [ "$errors" -gt 0 ]; then
    echo "⚠️  Found $errors error(s) in container logs:"
    echo "$error_logs" | tail -n "$errors"
else
    echo "✅ No errors found in container logs"
fi


echo ""
echo "Check completed at $(date)"
```

Make executable:

```bash
chmod +x ~/docker/maintenance/check-logs.sh
~/docker/maintenance/check-logs.sh
```

---

## Automated Monitoring

### Setting Up Daily Health Checks

Add to your crontab for automated daily checks:

```bash
# Edit crontab
crontab -e

# Add these lines:

# Daily health check at 8 AM
0 8 * * * /home/USERNAME/docker/maintenance/check-resources.sh >> /home/USERNAME/health-check.log 2>&1

# Daily service check at 9 AM
0 9 * * * /home/USERNAME/docker/maintenance/check-services.sh >> /home/USERNAME/health-check.log 2>&1

# Weekly log check every Monday at 10 AM
0 10 * * 1 /home/USERNAME/docker/maintenance/check-logs.sh >> /home/USERNAME/health-check.log 2>&1

# Daily disk space check at 11 AM
0 11 * * * /home/USERNAME/docker/maintenance/check-disk.sh >> /home/USERNAME/health-check.log 2>&1
```

> 💡 **TIP:** Change "USERNAME" to your actual username.

### View Automated Check Results

```bash
# View today's health check results
cat ~/health-check.log

# View last 50 lines
tail -50 ~/health-check.log

# Follow in real-time
tail -f ~/health-check.log
```

### Email Alerts (Optional)

If you want email alerts for critical issues:

```bash
# Install mail utility
sudo apt install mailutils -y

# Test sending email
echo "Test email from server" | mail -s "Server Test" your-email@example.com
```

Modify your scripts to send email on errors:

```bash
# Example: Add to check-disk.sh
if [ "$USAGE" -gt 90 ]; then
    echo "Disk usage critical: ${USAGE}%" | mail -s "⚠️  Server Alert: Disk Space" your-email@example.com
fi
```

---

## Maintenance Schedule

### Daily Tasks (5 minutes)

- [ ] Run `docker ps` - verify all containers running
- [ ] Run `df -h` - check disk space
- [ ] Run `free -h` - check memory usage
- [ ] Quick glance at `docker compose logs --tail=20`

**Quick daily command:**
```bash
# One-liner for daily check
docker ps && echo "---" && df -h / && echo "---" && free -h
```

### Weekly Tasks (15 minutes)

- [ ] Review full week's logs: `docker compose logs --since 7d | grep -i error`
- [ ] Check for Docker updates: `docker images`
- [ ] Verify backups completed: `ls -lh ~/docker/backups/`
- [ ] Check container restart counts
- [ ] Review system updates: `sudo apt update && apt list --upgradable`

**Weekly check script:**
```bash
nano ~/docker/maintenance/weekly-check.sh
```

```bash
#!/bin/bash

echo "=== Weekly Maintenance Check ==="
echo ""

echo "1. Checking for Docker image updates..."
docker images

echo ""
echo "2. Recent backups:"
ls -lht ~/docker/backups/ | head -7

echo ""
echo "3. Container restart counts:"
docker ps --format "{{.Names}}" | while read container; do
    restarts=$(docker inspect "$container" --format='{{.RestartCount}}')
    echo "   $container: $restarts restarts"
done

echo ""
echo "4. System updates available:"
sudo apt update > /dev/null 2>&1
apt list --upgradable 2>/dev/null | grep -v "Listing"

echo ""
echo "Weekly check completed at $(date)"
```
---

## Comprehensive Health Check Script

Create a complete health check script:

```bash
nano ~/docker/maintenance/health-check.sh
```

**Add this content:**

```bash
#!/bin/bash

#############################################
# Comprehensive Docker Health Check Script
# Checks system resources, containers, and logs
#############################################

# Colors for output
RED='\033[0;31m'
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
BLUE='\033[0;34m'
NC='\033[0m' # No Color

echo -e "${BLUE}==========================================${NC}"
echo -e "${BLUE}      Docker Environment Health Check     ${NC}"
echo -e "${BLUE}==========================================${NC}"
echo ""
echo "Check started at: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

# Track issues
ISSUES=0

#############################################
# 1. DOCKER SERVICE CHECK
#############################################
echo -e "${BLUE}[1/7] Docker Service Status${NC}"

if ! docker info > /dev/null 2>&1; then
    echo -e "${RED}✗ Docker is not running!${NC}"
    ((ISSUES++))
    exit 1
else
    echo -e "${GREEN}✓ Docker is running${NC}"
fi
echo ""

#############################################
# 2. SYSTEM RESOURCES
#############################################
echo -e "${BLUE}[2/7] System Resources${NC}"

# CPU Load
load=$(uptime | awk -F'load average:' '{print $2}' | awk '{print $1}' | tr -d ',')
echo -e "CPU Load Average: $load"

# Memory
mem_total=$(free -m | awk 'NR==2 {print $2}')
mem_used=$(free -m | awk 'NR==2 {print $3}')
mem_percent=$(awk "BEGIN {printf \"%.0f\", ($mem_used/$mem_total)*100}")

echo -ne "Memory Usage: ${mem_used}MB / ${mem_total}MB (${mem_percent}%)"
if [ "$mem_percent" -gt 85 ]; then
    echo -e " ${RED}✗ HIGH${NC}"
    ((ISSUES++))
elif [ "$mem_percent" -gt 70 ]; then
    echo -e " ${YELLOW}⚠ WARNING${NC}"
else
    echo -e " ${GREEN}✓${NC}"
fi

# Disk space
disk_usage=$(df -h / | awk 'NR==2 {print $5}' | tr -d '%')
disk_used=$(df -h / | awk 'NR==2 {print $3}')
disk_total=$(df -h / | awk 'NR==2 {print $2}')

echo -ne "Disk Usage: ${disk_used} / ${disk_total} (${disk_usage}%)"
if [ "$disk_usage" -gt 85 ]; then
    echo -e " ${RED}✗ HIGH${NC}"
    ((ISSUES++))
elif [ "$disk_usage" -gt 70 ]; then
    echo -e " ${YELLOW}⚠ WARNING${NC}"
else
    echo -e " ${GREEN}✓${NC}"
fi

# Docker disk usage
docker_size=$(docker system df --format "{{.Size}}" | head -1)
echo "Docker Storage: ${docker_size}"
echo ""

#############################################
# 3. CONTAINER STATUS
#############################################
echo -e "${BLUE}[3/7] Container Status${NC}"

# Change to docker directory
cd ~/docker 2>/dev/null || cd /home/*/docker 2>/dev/null || {
    echo -e "${RED}✗ Cannot find docker directory${NC}"
    ((ISSUES++))
    exit 1
}

# Get expected containers from docker-compose.yml
expected_containers=$(docker compose config --services 2>/dev/null)

if [ -z "$expected_containers" ]; then
    echo -e "${RED}✗ Cannot read docker-compose.yml${NC}"
    ((ISSUES++))
else
    all_running=true
    
    while IFS= read -r service; do
        status=$(docker compose ps "$service" --format "{{.Status}}" 2>/dev/null)
        
        if [[ "$status" == *"Up"* ]]; then
            echo -e "${GREEN}✓${NC} $service: Running"
        else
            echo -e "${RED}✗${NC} $service: $status"
            all_running=false
            ((ISSUES++))
        fi
    done <<< "$expected_containers"
    
    if [ "$all_running" = true ]; then
        echo -e "${GREEN}All containers are running${NC}"
    fi
fi
echo ""

#############################################
# 4. CONTAINER HEALTH (if health checks configured)
#############################################
echo -e "${BLUE}[4/7] Container Health Checks${NC}"

unhealthy=$(docker ps --filter health=unhealthy --format "{{.Names}}" 2>/dev/null)
if [ -n "$unhealthy" ]; then
    echo -e "${RED}✗ Unhealthy containers found:${NC}"
    echo "$unhealthy"
    ((ISSUES++))
else
    echo -e "${GREEN}✓ No unhealthy containers${NC}"
fi
echo ""

#############################################
# 5. RECENT ERRORS IN LOGS
#############################################
echo -e "${BLUE}[5/7] Recent Error Log Check${NC}"

error_count=$(docker compose logs --since 12h 2>&1 | grep -i "error\|critical\|fatal" | wc -l)

if [ "$error_count" -gt 0 ]; then
    echo -e "${RED}✗ Found $error_count errors in last 12 hours${NC}"
    echo "Recent errors:"
    docker compose logs --since 12h 2>&1 | grep -i "error\|critical\|fatal" | tail -n "$error_count"
    ((ISSUES++))
else
    echo -e "${GREEN}✓ No errors found in last 12 hours${NC}"
fi
echo ""

#############################################
# 6. BACKUP STATUS
#############################################
echo -e "${BLUE}[6/7] Backup Status${NC}"

backup_dir="$HOME/docker/backups"

if [ -d "$backup_dir" ]; then
    latest_backup=$(ls -t "$backup_dir"/*.tar.gz 2>/dev/null | head -1)
    
    if [ -n "$latest_backup" ]; then
        backup_name=$(basename "$latest_backup")
        backup_size=$(du -h "$latest_backup" | cut -f1)
        backup_date=$(stat -c %y "$latest_backup" | cut -d' ' -f1)
        backup_age_days=$(( ($(date +%s) - $(stat -c %Y "$latest_backup")) / 86400 ))
        
        echo "Latest backup: $backup_name"
        echo "Size: $backup_size"
        echo "Date: $backup_date"
        
        if [ "$backup_age_days" -gt 3 ]; then
            echo -e "${RED}✗ Backup is $backup_age_days days old!${NC}"
            ((ISSUES++))
        elif [ "$backup_age_days" -gt 1 ]; then
            echo -e "${YELLOW}⚠ Backup is $backup_age_days days old${NC}"
        else
            echo -e "${GREEN}✓ Backup is recent (${backup_age_days}d old)${NC}"
        fi
        
        # Count total backups
        backup_count=$(ls "$backup_dir"/*.tar.gz 2>/dev/null | wc -l)
        echo "Total backups: $backup_count"
    else
        echo -e "${RED}✗ No backups found!${NC}"
        ((ISSUES++))
    fi
else
    echo -e "${RED}✗ Backup directory not found!${NC}"
    ((ISSUES++))
fi
echo ""

#############################################
# 7. NETWORK CONNECTIVITY
#############################################
echo -e "${BLUE}[7/7] Network & Service Checks${NC}"

# Check internet connectivity
if ping -c 1 -W 2 8.8.8.8 > /dev/null 2>&1; then
    echo -e "${GREEN}✓${NC} Internet connectivity"
else
    echo -e "${RED}✗${NC} No internet connectivity"
    ((ISSUES++))
fi

# Check DNS resolution
if getent hosts google.com > /dev/null 2>&1; then
    echo -e "${GREEN}✓${NC} DNS resolution working"
else
    echo -e "${RED}✗${NC} DNS resolution failed"
    ((ISSUES++))
fi

# Check if ports are listening
ports_to_check="80 443"
for port in $ports_to_check; do
    if sudo netstat -tlnp 2>/dev/null | grep -q ":$port "; then
        echo -e "${GREEN}✓${NC} Port $port is listening"
    else
        echo -e "${RED}✗${NC} Port $port is NOT listening"
        ((ISSUES++))
    fi
done

echo ""

#############################################
# SUMMARY
#############################################
echo -e "${BLUE}==========================================${NC}"
echo -e "${BLUE}              Summary                     ${NC}"
echo -e "${BLUE}==========================================${NC}"
echo ""

if [ "$ISSUES" -eq 0 ]; then
    echo -e "${GREEN}✓ All checks passed - System is healthy!${NC}"
    exit_code=0
elif [ "$ISSUES" -le 2 ]; then
    echo -e "${YELLOW}⚠ Found $ISSUES minor issue(s) - Review recommended${NC}"
    exit_code=1
else
    echo -e "${RED}✗ Found $ISSUES issue(s) - Action required!${NC}"
    exit_code=2
fi

echo ""
echo "Check completed at: $(date '+%Y-%m-%d %H:%M:%S')"
echo ""

if [ "$ISSUES" -gt 0 ]; then
    echo "Recommended actions:"
    echo "1. Review issues listed above"
    echo "2. Check detailed logs: docker compose logs"
    echo "3. Check disk space: df -h && docker system df"
    echo "4. Create backup if needed: ~/docker/maintenance/backup.sh"
    echo "5. Contact administrator if issues persist"
fi

exit $exit_code
```

Make it executable:

```bash
chmod +x ~/docker/maintenance/health-check.sh
```

---

## Usage Examples

### Basic Health Check

```bash
~/docker/maintenance/health-check.sh
```

### Example Output - Healthy System

```
==========================================
      Docker Environment Health Check
==========================================

Check started at: 2026-01-07 12:03:00

[1/7] Docker Service Status
✓ Docker is running

[2/7] System Resources
CPU Load Average: 0.00
Memory Usage: 1599MB / 3915MB (41%) ✓
Disk Usage: 7.6G / 38G (22%) ✓
Docker Storage: 5.42GB

[3/7] Container Status
✓ db_core: Running
✓ postgres: Running
✓ n8n: Running
✓ pgadmin: Running
✓ phpmy: Running
✓ traefik: Running
All containers are running

[4/7] Container Health Checks
✓ No unhealthy containers

[5/7] Recent Error Log Check
✓ No errors found in last 12 hours

[6/7] Backup Status
Latest backup: docker-compose-20260107-045503.tar.gz
Size: 4.0K
Date: 2026-01-07
✓ Backup is recent (0d old)
Total backups: 7

[7/7] Network & Service Checks
✓ Internet connectivity
✓ DNS resolution working
✓ Port 80 is listening
✓ Port 443 is listening

==========================================
              Summary
==========================================

✓ All checks passed - System is healthy!

Check completed at: 2026-01-07 12:03:03

```

---

## Automated Daily Checks

Add to crontab for automated daily health checks:

```bash
# Edit crontab
crontab -e

# Add this line for daily health check at 9 AM
0 9 * * * ~/docker/maintenance/health-check.sh > ~/docker/maintenance/health-check.log 2>&1
```

Review the log:

```bash
cat ~/docker/maintenance/health-check.log
```

---

## Integration with Other Scripts

The health check script returns exit codes:
- `0` = All good
- `1` = Minor issues (warnings)
- `2` = Major issues (critical)

You can use this in other scripts:

```bash
#!/bin/bash

# Run health check before performing maintenance
if ~/docker/maintenance/health-check.sh; then
    echo "System healthy, proceeding with maintenance..."
    # Your maintenance tasks here
else
    echo "System has issues, aborting maintenance!"
    exit 1
fi
```

---

### Monthly Tasks (30 minutes)

- [ ] Test backup restoration
- [ ] Clean up old Docker images: `docker image prune -a`
- [ ] Review and clean old logs: `sudo journalctl --vacuum-time=30d`
- [ ] Update system packages: `sudo apt update && sudo apt upgrade`
- [ ] Review security: Check SSL certificates, review access logs
- [ ] Performance review: Check if containers need more resources

### Quarterly Tasks (1 hour)

- [ ] Full system audit
- [ ] Review and update documentation
- [ ] Test disaster recovery procedures
- [ ] Capacity planning (disk, memory projections)
- [ ] Security audit (review users, permissions, firewall rules)

---

## Quick Reference

### Essential Monitoring Commands

```bash
# Container status
docker ps                              # All running containers
docker compose ps                      # Compose stack status
docker stats --no-stream               # Resource usage snapshot

# Disk space
df -h                                  # Overall disk usage
docker system df                       # Docker disk usage
du -sh ~/* | sort -h                   # Home directory usage

# Memory & CPU
free -h                                # Memory usage
top                                    # CPU/memory live view
htop                                   # Better live view

# Logs
docker compose logs --tail=50          # Recent logs
docker compose logs -f servicename     # Follow specific service
sudo journalctl -u docker --since today # Docker service logs

# Health checks
docker inspect container --format='{{.State.Status}}'  # Container state
curl -I http://localhost:5678          # Test service response
```

### Warning Thresholds

| Metric | Warning | Critical | Action |
|--------|---------|----------|--------|
| Disk Usage | >80% | >90% | Clean up old files, backups |
| Memory Usage | >80% | >90% | Restart containers, upgrade RAM |
| CPU Load (2 cores) | >1.5 | >2.0 | Check for runaway processes |
| Container Restarts | >5 | >10 | Check logs, fix underlying issue |

### Log File Locations

```bash
# Docker container logs
~/.local/share/docker/containers/*/        # Container logs
docker compose logs                        # Via Docker command

# System logs
/var/log/syslog                           # General system log
sudo journalctl -u docker                  # Docker daemon log

# Your custom logs
~/health-check.log                        # Automated health checks
~/docker/backups/backup.log               # Backup logs
```

---

## Troubleshooting Common Issues

### High Disk Usage

```bash
# Find what's using space
docker system df -v                   # Detailed Docker usage
du -sh /var/lib/docker/*              # Docker system files
find ~ -type f -size +100M            # Large files in home

# Clean up
docker system prune -a                # Remove unused Docker data
sudo journalctl --vacuum-time=7d      # Clean old system logs
```

### High Memory Usage

```bash
# Identify memory hogs
docker stats --no-stream              # Check per-container usage
top -o %MEM                           # System-wide memory users

# Free up memory
docker restart container-name         # Restart specific container
docker compose restart                # Restart all services
```

### Container Keeps Restarting

```bash
# Check why it's restarting
docker compose logs container-name --tail=100
docker inspect container-name --format='{{.State.Status}}'

# Common fixes
docker compose down && docker compose up -d    # Fresh restart
docker compose pull && docker compose up -d    # Update and restart
```

### Cannot View Logs

```bash
# If permission denied:
sudo docker compose logs

# If container doesn't exist:
docker ps -a                          # List all containers (including stopped)

# If logs too large:
docker compose logs --tail=50         # View last 50 lines only
```

---

## Navigation

**Previous Guide:** [06 - Backup & Recovery](06-backup-recovery.md)  
**Next Guide:** [08 - Updates & Upgrades](08-updates-upgrades.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: [07/01/2026]*
