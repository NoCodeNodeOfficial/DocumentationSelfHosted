# 05 - Docker Management Cheatsheet

**Difficulty Level:** 🟡 Intermediate  
**Risk Level:** 🟠 Medium - *Be careful with container operations*

Master Docker and Docker Compose commands for managing your containers. Learn how to start, stop, monitor, and troubleshoot your services.

---

## 📋 Table of Contents

1. [Understanding Docker Basics](#understanding-docker-basics)
2. [Docker Compose Essentials](#docker-compose-essentials)
3. [Viewing Container Status](#viewing-container-status)
4. [Starting & Stopping Containers](#starting--stopping-containers)
5. [Viewing Logs](#viewing-logs)
6. [Executing Commands in Containers](#executing-commands-in-containers)
7. [Managing Container Resources](#managing-container-resources)
8. [Docker Images](#docker-images)
9. [Docker Volumes](#docker-volumes)
10. [Docker Networks](#docker-networks)
11. [Updating Containers](#updating-containers)
12. [Cleaning Up Docker](#cleaning-up-docker)
13. [Troubleshooting Docker](#troubleshooting-docker)

---

## Understanding Docker Basics

### What is Docker?

Docker runs your applications in **containers** - isolated environments that include everything needed to run the application.

**Key concepts:**
- **Container** - A running instance of your application
- **Image** - The template used to create containers
- **Volume** - Persistent storage for container data
- **Network** - How containers communicate
- **Docker Compose** - Tool to manage multiple containers

---

### Your Docker Setup

Your server uses Docker Compose to manage multiple services:

```
~/docker/
├── docker-compose.yml    # Main configuration file
├── maintenance/          # Maintenance scripts
├── backups/              # Backups
├── my.cnf                # MySQL configuration file
└── .env                  # Environment variables (passwords, etc.)

```

**Your containers:**
- **traefik** - Reverse proxy (handles web traffic and SSL)
- **n8n** - Workflow automation.
- **db_core** - Database for storage.
- **postgres** - Database for n8n if selected.
- **phpmy** - Tool for mysql database management.
- **pgadmin** - Tool for postgres database management.

---

### Docker vs Docker Compose

**Docker commands** - Manage individual containers:
```bash
docker ps
docker logs n8n
docker stop n8n
```

**Docker Compose commands** - Manage multiple containers as a group:
```bash
docker compose ps
docker compose logs n8n
docker compose stop
```

> 💡 **TIP:** Always use `docker compose` commands when working with your setup! They understand your `docker-compose.yml` configuration.

---

## Docker Compose Essentials

### Navigate to Your Docker Folder

**Always run Docker Compose commands from your docker folder:**

```bash
cd ~/docker
```

Or use the full path:
```bash
docker compose -f /home/USERNAME/docker/docker-compose.yml ps
```

> ⚠️ **IMPORTANT:** All examples below assume you're in `~/docker/` directory!

---

### Basic Docker Compose Commands

```bash
# Start all containers in a detached mode
docker compose up -d

# Stop all containers
docker compose down

# Restart all containers
docker compose restart

# View status of all containers
docker compose ps

# View logs of all containers
docker compose logs

# Pull latest images
docker compose pull

# Rebuild and restart
docker compose up -d --build

# Validate docker-compose.yml
docker compose config
```

**Command breakdown:**
- `docker compose` - The command
- `up` - Start containers
- `-d` - Detached mode (run in background)
- `down` - Stop and remove containers
- `ps` - Show container status
- `logs` - Show container logs

---

## Viewing Container Status

### List Running Containers

**Docker Compose (recommended):**
```bash
cd ~/docker
docker compose ps
```

**Example output:**
```
NAME       IMAGE                     COMMAND                  SERVICE    CREATED        STATUS             PORTS
db_core    mysql:8.4                 "docker-entrypoint.s…"   db_core    11 hours ago   Up About an hour   3306/tcp, 33060/tcp
n8n        docker.n8n.io/n8nio/n8n   "tini -- /docker-ent…"   n8n        11 hours ago   Up About an hour   127.0.0.1:5678->5678/tcp
pgadmin    dpage/pgadmin4:9          "/entrypoint.sh"         pgadmin    11 hours ago   Up About an hour   80/tcp, 443/tcp
phpmy      phpmyadmin:5.2            "/docker-entrypoint.…"   phpmy      11 hours ago   Up About an hour   80/tcp
postgres   postgres:17               "docker-entrypoint.s…"   postgres   11 hours ago   Up About an hour   5432/tcp
traefik    traefik                   "/entrypoint.sh --ap…"   traefik    11 hours ago   Up About an hour   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp
```

---

**Docker (all containers):**
```bash
docker ps
```

**Example output:**
```
CONTAINER ID   IMAGE                     COMMAND                  CREATED        STATUS       PORTS                                      NAMES
b89dcc467175   phpmyadmin:5.2            "/docker-entrypoint.…"   11 hours ago   Up 2 hours   80/tcp                                     phpmy
27b635555c20   dpage/pgadmin4:9          "/entrypoint.sh"         11 hours ago   Up 2 hours   80/tcp, 443/tcp                            pgadmin
3ef0351fce81   docker.n8n.io/n8nio/n8n   "tini -- /docker-ent…"   11 hours ago   Up 2 hours   127.0.0.1:5678->5678/tcp                   n8n
d96aefe03d77   postgres:17               "docker-entrypoint.s…"   11 hours ago   Up 2 hours   5432/tcp                                   postgres
a6006df13d7f   mysql:8.4                 "docker-entrypoint.s…"   11 hours ago   Up 2 hours   3306/tcp, 33060/tcp                        db_core
d878b558c411   traefik                   "/entrypoint.sh --ap…"   11 hours ago   Up 2 hours   0.0.0.0:80->80/tcp, 0.0.0.0:443->443/tcp   traefik
```

---

**Show all containers (including stopped):**
```bash
docker ps -a
```

---

**Show only container names:**
```bash
docker ps --format "{{.Names}}"
```

---

**Show with specific format:**
```bash
docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

### Check Container Health

```bash
# Check if container is running
docker ps | grep n8n

# Detailed container info
docker inspect n8n

# Check container health status
docker inspect --format='{{.State.Status}}' n8n

# View container stats (CPU, memory)
docker stats

# Stats for specific container
docker stats n8n

# Stats without streaming (one-time)
docker stats --no-stream
```

---

## Starting & Stopping Containers

### Start Containers

**Start all containers:**
```bash
cd ~/docker
docker compose up -d
```

**Start specific container:**
```bash
docker compose up -d n8n
```

**Start multiple specific containers:**
```bash
docker compose up -d n8n postgres
```

**Start and view logs:**
```bash
docker compose up
# Press Ctrl+C to stop viewing logs and stop all the containers at the same time.
# Press d to detach yourself from viewing the log live.
# Press w to go into watch mode (if configured)
```

---

### Stop Containers

**Stop all containers:** (without removing them)
```bash
cd ~/docker
docker compose stop
```

**Stop specific container:**
```bash
docker compose stop n8n
```

**Stop and remove containers:**
```bash
docker compose down
```

> ⚠️ **NOTE:** `docker compose down` removes containers but keeps volumes (data is safe).

**Stop and remove containers AND volumes (DANGEROUS!):**
```bash
docker compose down -v
```

> 🚨 **DANGER:** This deletes all data! Only use if you want to completely reset.

---

### Restart Containers

**Restart all containers:**
```bash
cd ~/docker
docker compose restart
```

**Restart specific container:**
```bash
docker compose restart n8n
```

**Restart with timeout:**
```bash
docker compose restart -t 30 n8n
```

**Force restart (stop and start):**
```bash
docker compose stop n8n
docker compose up -d n8n
```

---

### Pause & Unpause Containers

```bash
# Pause container (freeze processes)
docker pause n8n

# Unpause container
docker unpause n8n

# Pause via Docker Compose
docker compose pause n8n

# Unpause via Docker Compose
docker compose unpause n8n
```

> 💡 **TIP:** Pause is useful for temporarily freezing a container without stopping it.

---

## Viewing Logs

### Docker Compose Logs

**View logs of all containers:**
```bash
cd ~/docker
docker compose logs
```

**Follow logs in real-time:**
```bash
docker compose logs -f
```

**View logs of specific container:**
```bash
docker compose logs n8n
```

**Follow specific container logs:**
```bash
docker compose logs -f n8n
```

**Show last N lines:**
```bash
docker compose logs --tail=100 n8n
```

**Show logs with timestamps:**
```bash
docker compose logs -t n8n
```

**Combine options:** (Live log + timestamp + last 50 lines)
```bash
docker compose logs -f --tail=50 -t n8n
```

---

### Docker Logs

**View container logs:**
```bash
docker logs n8n
```

**Follow logs:**
```bash
docker logs -f n8n
```

**Show last N lines:**
```bash
docker logs --tail=100 n8n
```

**Show logs since specific time:**
```bash
# Last hour
docker logs --since 1h n8n

# Last 30 minutes
docker logs --since 30m n8n

# Since specific date
docker logs --since 2024-01-15 n8n
```

**Show logs until specific time:**
```bash
docker logs --until 2024-01-15T10:30:00 n8n
```

**Search logs:**
```bash
docker logs n8n | grep error
docker logs n8n | grep -i "error\|warning\|fail"
```

---

### Save Logs to File

```bash
# Save all logs
docker compose logs > all-logs.txt

# Save specific container logs
docker compose logs n8n > n8n-logs.txt

# Save with timestamp
docker compose logs n8n > n8n-logs-$(date +%Y%m%d-%H%M%S).txt

# Save last 1000 lines
docker compose logs --tail=1000 n8n > n8n-recent.txt
```

---

### Real-Time Log Monitoring

**Monitor multiple containers:**
```bash
# In separate terminals or use tmux/screen
docker compose logs -f n8n
docker compose logs -f postgres
docker compose logs -f traefik
```

**Using grep for filtering:**
```bash
# Show only errors
docker compose logs -f n8n | grep -i error

# Show only specific patterns
docker compose logs -f | grep -E "n8n|db_core|error"

# Exclude certain patterns
docker compose logs -f | grep -v "debug"
```

---

## Executing Commands in Containers

### Docker Compose Exec

**Open bash shell in container:**
```bash
docker compose exec n8n bash
```

If bash is not available, try sh:
```bash
docker compose exec n8n sh
```

**Run single command:**
```bash
docker compose exec n8n ls -la
docker compose exec n8n pwd
docker compose exec n8n env
```

**Run as specific user:**
```bash
docker compose exec -u root n8n bash
```

**Run without TTY (for scripts):**
```bash
docker compose exec -T n8n command
```

---

### Docker Exec

**Execute command in running container:**
```bash
docker exec n8n ls -la
```

**Interactive shell:**
```bash
docker exec -it n8n bash
```

**Run as root:**
```bash
docker exec -u root -it n8n bash
```

---

### Common Container Commands

**Check container environment variables:**
```bash
docker compose exec n8n env
```

**Check processes in container:**
```bash
docker compose exec n8n ps aux
```

**Check container disk usage:**
```bash
docker compose exec n8n df -h
```

**Test network connectivity:**
```bash
docker compose exec n8n ping -c 4 google.com
docker compose exec n8n ping -c 4 postgres
```

**Check file contents:**
```bash
docker compose exec n8n cat /etc/hosts
```

---

### MySQL Database Commands

**Connect to MySQL:**
```bash
docker compose exec db_core mysql -u root -p
# Enter password from .env file (MYSQL_ROOT_PASSWORD)
```

**Run MySQL command directly:**
```bash
docker compose exec db_core mysql -u root -pYOUR_PASSWORD -e "SHOW DATABASES;"
```

**Export database:**
```bash
docker compose exec db_core mysqldump -u root -pYOUR_PASSWORD n8n > backup.sql
```

**Import database:**
```bash
cat backup.sql | docker compose exec -T db_core mysql -u root -pYOUR_PASSWORD n8n
```

---

## Managing Container Resources

### View Resource Usage

**Real-time stats:**
```bash
docker stats
```

**Stats for specific container:**
```bash
docker stats n8n
```

**One-time stats snapshot:**
```bash
docker stats --no-stream
```

**Formatted output:**
```bash
docker stats --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"
```

---

### Check Container Size

```bash
# Show container sizes
docker ps -s

# Show image sizes
docker images

# Detailed container info
docker inspect n8n | grep -i size
```

---

### Limit Container Resources

**In docker-compose.yml:**
```yaml
services:
  n8n:
    image: n8nio/n8n:latest
    deploy:
      resources:
        limits:
          cpus: '1.0'
          memory: 1G
        reservations:
          memory: 512M
```

**Update and restart modified container:** (After editing docker-compose.yml file)
```bash
docker compose up -d
```

---

## Docker Images

### List Images

```bash
# List all images
docker images

# List with full details
docker images --no-trunc

# List image IDs only
docker images -q

# Filter images
docker images | grep n8n
```

---

### Pull Images

**Pull latest version:**
```bash
docker pull n8nio/n8n:latest
```

**Pull specific version:**
```bash
docker pull n8nio/n8n:2.2.4
```

**Pull all images from docker-compose.yml:**
```bash
cd ~/docker
docker compose pull
```

---

### Remove Images

**Remove specific image:**
```bash
docker rmi n8nio/n8n:old-version
```

**Remove by ID:**
```bash
docker rmi a1b2c3d4e5f6
```

**Force remove:**
```bash
docker rmi -f n8nio/n8n:old-version
```

**Remove unused images:**
```bash
docker image prune
```

**Remove all unused images:**
```bash
docker image prune -a
```

---

### Image Information

```bash
# Show image history
docker history n8nio/n8n:latest

# Inspect image
docker inspect n8nio/n8n:latest

# Show image layers
docker inspect -f '{{.RootFS.Layers}}' n8nio/n8n:latest
```

---

## Docker Volumes

### Understanding Your Volumes

Your data is stored in Docker volumes (not mounted folders):

```bash
# List all volumes
docker volume ls

# Your typical volumes:
DRIVER    VOLUME NAME
local     docker_mysql_data
local     docker_n8n_data
local     docker_n8n_files
local     docker_traefik_letsencrypt
local     docker_pgadmin_data
local     docker_postgres_data

```

---

### List Volumes

```bash
# List all volumes
docker volume ls

# Filter volumes
docker volume ls | grep n8n

# List with additional info
docker volume ls --format "table {{.Name}}\t{{.Driver}}"
```

---

### Inspect Volumes

```bash
# View volume details
docker volume inspect docker_n8n_data

# Find volume location on host
docker volume inspect docker_n8n_data --format '{{.Mountpoint}}'

# Typical location: /var/lib/docker/volumes/docker_n8n_data/_data
```

---

### Access Volume Data

**View volume contents (requires root):**
```bash
# Navigate to volume location
sudo ls -la /var/lib/docker/volumes/docker_n8n_data/_data

# View files in volume
sudo find /var/lib/docker/volumes/docker_n8n_data/_data -type f

# Check volume size
sudo du -sh /var/lib/docker/volumes/docker_n8n_data/_data
```

**Access via container:**
```bash
docker compose exec n8n ls -la /home/node/.n8n
docker compose exec n8n du -sh /home/node/.n8n
```

---

### Backup Volumes

**Method 1: Using tar (recommended):**
```bash
# Backup n8n volume
docker run --rm \
  -v docker_n8n_data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/n8n-backup-$(date +%Y%m%d).tar.gz -C /data .

# Backup MySQL volume
docker run --rm \
  -v docker_mysql_data:/data \
  -v $(pwd):/backup \
  ubuntu tar czf /backup/mysql-backup-$(date +%Y%m%d).tar.gz -C /data .

```

**Method 2: Export database:**
```bash
# MySQL backup
docker compose exec db_core mysqldump -u root -pYOUR_PASSWORD n8n > n8n-db-backup-$(date +%Y%m%d).sql
```

**Method 3: Copy volume data:**
```bash
# Copy volume to local folder
sudo cp -r /var/lib/docker/volumes/docker_n8n_data/_data ./n8n-backup-$(date +%Y%m%d)
```

---

### Restore Volumes

**Restore from tar backup:**
```bash
# Restore n8n volume
docker run --rm \
  -v docker_n8n_data:/data \
  -v $(pwd):/backup \
  ubuntu bash -c "cd /data && tar xzf /backup/n8n-backup-20240115.tar.gz"
```

**Restore MySQL database:**
```bash
# Import SQL backup from a mysqldump
cat n8n-db-backup-20240115.sql | docker compose exec -T db_core mysql -u root -pYOUR_PASSWORD n8n
```

---

### Remove Volumes

> 🚨 **DANGER:** This permanently deletes data!

```bash
# Remove specific volume (must stop containers first)
docker compose down
docker volume rm docker_n8n_data

# Remove all unused volumes
docker volume prune

# Remove all volumes (DANGEROUS!)
docker volume prune -a
```

---

### Create Volumes

```bash
# Create volume manually
docker volume create my-volume

# Create with specific driver
docker volume create --driver local my-volume
```

---

## Docker Networks

### List Networks

```bash
# List all networks
docker network ls

# Your default network
docker network ls | grep proxy
```

---

### Inspect Network

```bash
# View network details (proxy is the one I created for your project)
docker network inspect proxy

# See connected containers
docker network inspect proxy --format '{{range .Containers}}{{.Name}} {{end}}'
```

---

### Test Container Connectivity

```bash
# Ping between containers (by name)
docker compose exec n8n ping -c 4 db_core
docker compose exec n8n ping -c 4 postgres
docker compose exec n8n ping -c 4 traefik

# Test port connectivity
docker compose exec n8n nc -zv db_core 3306
```

---

### Network Troubleshooting

```bash
# Check if container is on network
docker network inspect proxy | grep n8n

# View container's network settings
docker inspect n8n --format '{{.NetworkSettings.Networks}}'

# Get container IP address
docker inspect n8n --format '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
```

---

## Updating Containers

### Update Single Container

**Step-by-step update process:**

```bash
# 1. Navigate to docker folder
cd ~/docker

# 2. Pull latest image
docker compose pull n8n

# 3. Stop container
docker compose stop n8n

# 4. Start with new image
docker compose up -d n8n

# 5. Check logs
docker compose logs -f n8n

# 6. Verify it's running
docker compose ps
```

**Quick one-liner:**
```bash
cd ~/docker && docker compose pull n8n && docker compose up -d n8n
```

---

### Update All Containers

```bash
# 1. Navigate to docker folder
cd ~/docker

# 2. Pull all latest images
docker compose pull

# 3. Recreate all containers
docker compose up -d

# 4. Check status
docker compose ps

# 5. Monitor logs
docker compose logs -f
```

---

### Update to Specific Version

**Edit docker-compose.yml:**
```yaml
services:
  n8n:
    image: n8nio/n8n:2.2.4  # Change from :latest to specific version
```

**Apply changes:**
```bash
docker compose pull n8n
docker compose up -d n8n
```

---

### Rollback to Previous Version

```bash
# 1. Check available image versions
docker images | grep n8n

# 2. Edit docker-compose.yml to use old version
nano docker-compose.yml
# Change: image: n8nio/n8n:1.19.0

# 3. Apply changes
docker compose up -d n8n

# 4. Verify
docker compose ps
docker compose logs n8n
```

---

## Cleaning Up Docker

### Remove Stopped Containers

```bash
# Remove all stopped containers
docker container prune

# Remove with force (no confirmation)
docker container prune -f
```

---

### Remove Unused Images

```bash
# Remove dangling images (untagged)
docker image prune

# Remove all unused images
docker image prune -a

# Remove with force
docker image prune -af
```

---

### Remove Unused Volumes

```bash
# Remove unused volumes
docker volume prune

# Remove with force
docker volume prune -f
```

> ⚠️ **WARNING:** Make sure you have backups before pruning volumes!

---

### Remove Unused Networks

```bash
# Remove unused networks
docker network prune

# Remove with force
docker network prune -f
```

---

### Complete Docker Cleanup

```bash
# Clean everything (except volumes)
docker system prune

# Clean everything including volumes (DANGEROUS!)
docker system prune -a --volumes

# See what will be removed
docker system df
```

**Example output:**
```
TYPE            TOTAL     ACTIVE    SIZE      RECLAIMABLE
Images          7         6         5.418GB   5.299GB (97%)
Containers      6         6         61.12MB   0B (0%)
Local Volumes   6         6         449.8MB   0B (0%)
Build Cache     0         0         0B        0B
```

---

### Scheduled Cleanup Script

**Create cleanup script:**
```bash
nano ~/docker/maintenance/docker-cleanup.sh
```

**Script content:**
```bash
#!/bin/bash

echo "Starting Docker cleanup..."

# Remove stopped containers
echo "Removing stopped containers..."
docker container prune -f

# Remove unused images
echo "Removing unused images..."
docker image prune -af

# Remove unused networks
echo "Removing unused networks..."
docker network prune -f

# Show disk usage
echo ""
echo "Current Docker disk usage:"
docker system df

echo ""
echo "Cleanup complete!"
```

**Make executable:**
```bash
chmod +x ~/docker/maintenance/docker-cleanup.sh
```

**Run it:**
```bash
~/docker/maintenance/docker-cleanup.sh
```

---

## Troubleshooting Docker

### Container Won't Start

**Check logs:**
```bash
docker compose logs n8n
docker compose logs --tail=100 n8n
```

**Check container status:**
```bash
docker compose ps
docker ps -a | grep n8n
```

**Check for errors:**
```bash
docker inspect n8n
```

**Common issues:**

1. **Port already in use:**
```bash
# Find what's using the port
sudo ss -tulpn | grep :5678

# Change port in docker-compose.yml or kill the process
```

2. **Permission issues:**
```bash
# Check volume permissions
sudo ls -la /var/lib/docker/volumes/docker_n8n_data/_data

# Fix permissions (from inside container)
docker compose exec -u root n8n chown -R node:node /data
```

3. **Out of disk space:**
```bash
# Check disk space
df -h

# Clean up Docker
docker system prune -a
```

---

### Container Keeps Restarting

**Check restart policy:**
```bash
docker inspect n8n --format '{{.HostConfig.RestartPolicy}}'
```

**View restart count:**
```bash
docker inspect n8n --format '{{.RestartCount}}'
```

**Check for crash loop:**
```bash
# Watch logs
docker compose logs -f n8n

# Check exit code
docker inspect n8n --format '{{.State.ExitCode}}'
```

**Temporary disable restart:**
```bash
docker update --restart=no n8n
```

---

### Performance Issues

**Check resource usage:**
```bash
# Real-time monitoring
docker stats

# Check container limits
docker inspect n8n --format '{{.HostConfig.Memory}}'
docker inspect n8n --format '{{.HostConfig.CpuShares}}'
```

**Check logs for errors:**
```bash
docker compose logs n8n | grep -i "error\|warning\|memory\|timeout"
```

**Check database performance:**
```bash
# Connect to MySQL
docker compose exec db_core mysql -u root -p

# Run queries
SHOW PROCESSLIST;
SHOW ENGINE INNODB STATUS\G
```

---

### Network Issues

**Test container connectivity:**
```bash
# From host to container
ping YOUR_SERVER_IP

# From container to container
docker compose exec n8n ping db_core
docker compose exec n8n nc -zv db_core 3306

# From container to internet
docker compose exec n8n ping -c 4 google.com
```

**Check DNS resolution:**
```bash
docker compose exec n8n nslookup google.com
docker compose exec n8n nslookup db_core
```

**Check network configuration:**
```bash
docker network inspect proxy
```

---

### Volume Issues

**Check volume exists:**
```bash
docker volume ls | grep docker_n8n_data
```

**Check volume permissions:**
```bash
sudo ls -la /var/lib/docker/volumes/docker_n8n_data/_data
```

**Check volume size:**
```bash
sudo du -sh /var/lib/docker/volumes/docker_n8n_data/_data
```

**Recreate volume (DANGER - LOSE DATA!):**
```bash
docker compose down
docker volume rm docker_n8n_data
docker compose up -d
```

---

### Image Pull Issues

**Pull with verbose output:**
```bash
docker pull n8nio/n8n:latest --verbose
```

**Check Docker Hub status:**
```bash
curl -s https://status.docker.com/
```

**Use different registry mirror:**
```bash
# Edit /etc/docker/daemon.json
sudo nano /etc/docker/daemon.json

# Add mirror
{
  "registry-mirrors": ["https://mirror.gcr.io"]
}

# Restart Docker
sudo systemctl restart docker
```

---

### Docker Daemon Issues

**Check Docker daemon status:**
```bash
sudo systemctl status docker
```

**Restart Docker daemon:**
```bash
sudo systemctl restart docker
```

**View Docker daemon logs:**
```bash
sudo journalctl -u docker -n 100
sudo journalctl -u docker -f
```

**Check Docker version:**
```bash
docker version
docker info
```

---

## 📌 Quick Command Reference

```bash
# Navigate to docker folder
cd ~/docker

# Status & Monitoring
docker compose ps                          # Container status
docker compose logs -f n8n                 # Follow logs
docker stats                               # Resource usage

# Start & Stop
docker compose up -d                       # Start all
docker compose up -d n8n                   # Start specific
docker compose stop                        # Stop all
docker compose stop n8n                    # Stop specific
docker compose restart n8n                 # Restart specific
docker compose down                        # Stop and remove

# Updates
docker compose pull                        # Pull latest images
docker compose up -d                       # Recreate containers

# Maintenance
docker compose exec n8n bash               # Open shell
docker volume ls                           # List volumes
docker image prune -a                      # Remove unused images
docker system df                           # Disk usage

# Troubleshooting
docker compose logs --tail=100 n8n         # Last 100 log lines
docker inspect n8n                         # Container details
docker stats n8n                           # Resource usage
```

---

## 🎓 Common Workflows

### Daily Health Check

```bash
cd ~/docker
docker compose ps                # All running?
docker stats --no-stream         # Resource usage OK?
df -h                            # Disk space OK?
docker compose logs --tail=50    # Any errors?
```

---

### Weekly Maintenance

```bash
cd ~/docker

# Check for updates
docker compose pull

# Update containers
docker compose up -d

# Clean up
docker image prune -a
docker volume prune

# Check logs
docker compose logs --tail=100
```

---

### Restart Everything

```bash
cd ~/docker
docker compose down
docker compose up -d
docker compose ps
docker compose logs -f
```

---

### Backup Before Changes

```bash
cd ~/docker

# Backup volumes
docker run --rm -v n8n_data:/data -v $(pwd):/backup \
  ubuntu tar czf /backup/n8n-backup-$(date +%Y%m%d).tar.gz -C /data .

# Backup MySQL
docker compose exec db_core mysqldump -u root -p n8n > backup-$(date +%Y%m%d).sql

# Backup docker-compose.yml and .env
cp docker-compose.yml docker-compose.yml.backup
cp .env .env.backup
```

---

**Next Guide:** [06 - Backup & Recovery](06-backup-recovery.md)  
**Previous Guide:** [04 - Linux Command Cheatsheet](04-linux-cheatsheet.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: [07/01/2026]*
