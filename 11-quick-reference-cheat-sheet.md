# 11 - Quick Reference & Cheat Sheet

**Difficulty Level:** 🟢 Reference  
**Risk Level:** 🟢 None - *Read-only reference guide*

Your one-stop reference for all important commands, paths, ports, and terms. Print this page and keep it handy!

---

## 📋 Table of Contents

1. [Essential Commands](#essential-commands)
2. [File Paths Reference](#file-paths-reference)
3. [Ports & URLs](#ports--urls)
4. [Container Names](#container-names)
5. [Docker Volume Names](#docker-volume-names)
6. [Important Log Locations](#important-log-locations)
7. [Quick Troubleshooting Commands](#quick-troubleshooting-commands)
8. [Glossary of Terms](#glossary-of-terms)
9. [One-Page Printable Reference](#one-page-printable-reference)

---

## Essential Commands

### Connecting to Server

```bash
# SSH connection
ssh USERNAME@your-server-ip

# SSH with specific port and ssh key
ssh -p 1077 USERNAME@your-server-ip -i pathfile/to/your/private/key

# Exit SSH session
exit
```

### Docker Compose Commands

```bash
# View running containers
docker compose ps

# View all containers (including stopped)
docker compose ps -a

# Start all services
docker compose up -d

# Stop all services
docker compose down

# Restart all services
docker compose restart

# Restart specific service
docker compose restart n8n

# View logs (all services)
docker compose logs

# View logs (specific service)
docker compose logs n8n

# Follow logs in real-time
docker compose logs -f n8n

# View last 50 log lines
docker compose logs --tail=50 n8n

# Pull latest images
docker compose pull

# Update containers
docker compose pull && docker compose up -d
```

### System Monitoring

```bash
# Check disk space
df -h

# Check memory usage
free -h

# Check CPU and load
top
htop  # (if installed)

# Check Docker disk usage
docker system df

# Check system uptime
uptime

# Check running processes
ps aux | grep docker
```

### Backup & Restore

```bash
# Run backup script
~/docker/maintenance/backup.sh

# Restore specific volume
~/docker/maintenance/restore.sh volume_name /path/to/backup.tar.gz

# Example: Restore n8n data
~/docker/maintenance/restore.sh n8n_data ~/docker/backups/n8n_data-20240115-020000.tar.gz

# List available backups
ls -lh ~/docker/backups/

# Check backup age
ls -lht ~/docker/backups/ | head -10
```

### Health Checks

```bash
# Run comprehensive health check
~/docker/maintenance/health-check.sh

# Quick container status
docker compose ps

# Check container resource usage
docker stats

# Check specific container health
docker inspect --format='{{.State.Health.Status}}' container_name
```

### File Management

```bash
# Navigate to docker directory
cd ~/docker

# Edit docker-compose.yml
nano ~/docker/docker-compose.yml

# View file contents
cat ~/docker/docker-compose.yml

# Check file permissions
ls -lah ~/docker/

# Create directory
mkdir -p ~/docker/backups

# Remove file
rm filename

# Remove directory and contents
rm -rf directory_name
```

---

## File Paths Reference

### Configuration Files

| File                  | Path                          | Purpose                 |
|-----------------------|-------------------------------|-------------------------|
| Docker Compose        | `~/docker/docker-compose.yml` | Main configuration file |
| MySQL Config          | `~/docker/my.cnf`             | MySQL config file       |
| Environment Variables | `~/docker/.env`               | Secrets and passwords   |

### Script Files

| Script             | Path                                     | Purpose                                  |
|--------------------|------------------------------------------|------------------------------------------|
| Backup Script      | `~/docker/maintenance/backup.sh`         | Create backups                           |
| Restore Script     | `~/docker/maintenance/restore.sh`        | Restore from backups                     |
| Health Check       | `~/docker/maintenance/health-check.sh`   | System health monitoring                 |
| Check disk usage   | `~/docker/maintenance/check-disk.sh`   | Monitor system disk usage                |
| Check logs         | `~/docker/maintenance/check-logs.sh`   | Check for relevant log                   |
| Check resources    | `~/docker/maintenance/check-resources.sh`   | Check for abnormal use of resources      |
| Check services     | `~/docker/maintenance/check-services.sh`   | Check for services health                |
| Check updates      | `~/docker/maintenance/check-updates.sh`   | Look for system update                   |
| Cleaning docker    | `~/docker/maintenance/docker-cleanup.sh`   | Clean all the unused docker files        |
| Update OS          | `~/docker/maintenance/os-updates.sh`   | Process OS update                        |
| Service rollback   | `~/docker/maintenance/rollback.sh`   | Rollback a service after a failed update |
| Update everything  | `~/docker/maintenance/update-all.sh`   | Update all the services                  |
| Update one service | `~/docker/maintenance/update-n8n.sh`   | Update only one service                  |
| Weekly check       | `~/docker/maintenance/weekly-check.sh`   | Do the system weekly check               |

### Data Directories

| Directory     | Path                                                  | Contents         |
|---------------|-------------------------------------------------------|------------------|
| Docker Root   | `~/docker/`                                           | All Docker files |
| Backups       | `~/docker/backups/`                                   | Backup archives  |
| Traefik Certs | `/var/lib/docker/volumes/docker_traefik_letsencrypt/` | SSL certificates |
| n8n Data      | `/var/lib/docker/volumes/docker_n8n_data/`            | n8n workflows    |
| n8n Files     | `/var/lib/docker/volumes/docker_n8n_files/`           | n8n files        |
| MySQL Data    | `/var/lib/docker/volumes/docker_mysql_data/`          | Database files   |

### Documentation Files

| File | Path                    | Purpose |
|------|-------------------------|---------|
| Setup Notes | `~/docker/doc/SETUP.md` | Installation documentation |
| Changelog | `~/docker/doc/CHANGELOG.md` | Change history |
| README | `~/docker/doc/README.md`    | Overview and quick start |

---

## Ports & URLs

### External Ports (Public Access)

| Port | Service | Purpose | URL                                  |
|------|---------|---------|--------------------------------------|
| 80   | Traefik | HTTP (redirects to HTTPS) | `http://traefik.yourdomain.website`  |
| 443  | Traefik | HTTPS | `https://traefik.yourdomain.website` |
| 1077 | SSH | Server access | N/A                                  |

### Internal Ports (Container-to-Container)

| Port | Service | Purpose |
|------|---------|---------|
| 5678 | n8n | Web interface |
| 3306 | MySQL | Database |

### Service URLs

| Service | URL                                 | Access       |
|---------|-------------------------------------|--------------|
| n8n | `https://n8n.yourdomain.website`    | Authenticated |
| Traefik Dashboard | `https://traefik.yourdomain.website`    | Basic Auth   |
| phpMyAdmin | `https://phpmy.yourdomain.website` | Basic Auth   |

---

## Container Names

When running Docker commands, use these container names:

| Service | Container Name | Purpose |
|---------|----------------|---------|
| Traefik | `traefik` | Reverse proxy & SSL |
| n8n | `n8n` | Workflow automation |
| MySQL | `db_core` | Database |
| phpMyAdmin | `phpmy` | Database admin |

**Example usage:**
```bash
docker compose logs traefik
docker compose restart n8n
docker exec -it db_core mysql -u root -p
```

---

## Docker Volume Names

Full volume names (as Docker sees them):

| Logical Name        | Full Docker Name             | Contents                   |
|---------------------|------------------------------|----------------------------|
| n8n_data            | `docker_n8n_data`            | n8n workflows and settings |
| n8n_files           | `docker_n8n_files`           | n8n files                  |
| mysql_data          | `docker_mysql_data`          | MySQL database files       |
| traefik_letsencrypt | `docker_traefik_letsencrypt` | SSL certificates           |

**Check volumes:**
```bash
docker volume ls
docker volume inspect docker_n8n_data
```

---

## Important Log Locations

### Container Logs

```bash
# All services
docker compose logs

# Specific service
docker compose logs traefik
docker compose logs n8n
docker compose logs db_core

# Real-time logs
docker compose logs -f n8n

# Last 100 lines
docker compose logs --tail=100 n8n

# Logs since specific time
docker compose logs --since 1h n8n
docker compose logs --since "2024-01-15T14:00:00"
```

### System Logs

| Log Type | Location | Command |
|----------|----------|---------|
| SSH Authentication | `/var/log/auth.log` | `sudo tail -f /var/log/auth.log` |
| System Messages | `/var/log/syslog` | `sudo tail -f /var/log/syslog` |
| Kernel Messages | `/var/log/kern.log` | `sudo tail -f /var/log/kern.log` |

### Health Check Logs

```bash
# Manual health check
~/docker/maintenance/health-check.sh

# Automated health check log
cat ~/docker/health-check.log
```

---

## Quick Troubleshooting Commands

### Service Not Working

```bash
# 1. Check container status
docker compose ps

# 2. Check recent logs
docker compose logs --tail=50 service_name

# 3. Restart service
docker compose restart service_name

# 4. Check if port is listening
sudo netstat -tlnp | grep :443
```

### High Resource Usage

```bash
# Check overall usage
docker stats

# Check disk space
df -h

# Check Docker disk usage
docker system df

# Check memory
free -h

# Check CPU load
uptime
```

### Container Won't Start

```bash
# Check logs for errors
docker compose logs service_name

# Try starting manually
docker compose up service_name

# Check configuration syntax
docker compose config

# Verify file permissions
ls -la ~/docker/
```

### Can't Access Service

```bash
# 1. Is container running?
docker compose ps

# 2. Is Traefik running?
docker compose logs traefik

# 3. Check DNS resolution
nslookup yourdomain.website

# 4. Check certificates
docker compose logs traefik | grep -i cert
```

### Database Connection Issues

```bash
# 1. Is MySQL running?
docker compose ps db_core

# 2. Check MySQL logs
docker compose logs db_core

# 3. Test connection
docker exec -it db_core mysql -u root -p

# 4. Check n8n can reach database
docker compose logs n8n | grep -i database
```

---

## Glossary of Terms

### A-C

**API (Application Programming Interface)**  
A way for different software applications to communicate with each other.

**Backup**  
A copy of your data saved to restore in case of data loss.

**Container**  
A lightweight, standalone package that includes everything needed to run a piece of software.

**Compose (Docker Compose)**  
A tool for defining and running multi-container Docker applications using a YAML file.

**CPU (Central Processing Unit)**  
The "brain" of your computer that performs calculations and tasks.

**cron / crontab**  
A time-based job scheduler in Unix-like systems for running automated tasks.

### D-H

**Docker**  
A platform for developing, shipping, and running applications in containers.

**DNS (Domain Name System)**  
Translates human-readable domain names (like google.com) into IP addresses.

**Disk Space**  
Available storage space on your server's hard drive.

**Environment Variable**  
A value that can affect how programs behave, often used for configuration.

**HTTPS (Hypertext Transfer Protocol Secure)**  
Secure version of HTTP used for encrypted web communication.

**Host**  
The physical or virtual machine running your Docker containers.

### I-M

**Image (Docker Image)**  
A template used to create Docker containers, like a blueprint.

**IP Address**  
A unique numerical identifier for devices on a network (e.g., 192.168.1.100).

**Let's Encrypt**  
A free service that provides SSL/TLS certificates for HTTPS.

**Load Average**  
A measure of how much work your system is doing (CPU usage over time).

**Log**  
A record of events and activities that happened in your system or application.

**Memory (RAM)**  
Temporary storage your computer uses to run programs actively.

**MySQL**  
An open-source relational database management system.

### N-R

**n8n**  
An open-source workflow automation tool.

**Namespace**  
A way to organize and isolate resources in Docker.

**netstat**  
A command-line tool for displaying network connections and statistics.

**Port**  
A virtual endpoint for network communications (like different doors in a building).

**Process**  
A running instance of a program on your computer.

**Reverse Proxy**  
A server that sits between clients and backend servers, forwarding requests (Traefik does this).

**Restart Policy**  
Rules that define when and how Docker should restart a container.

**Root**  
The superuser account with full system privileges (use carefully!).

### S-Z

**SSH (Secure Shell)**  
A protocol for securely connecting to remote computers.

**SSL/TLS Certificate**  
Digital certificates that enable HTTPS encryption.

**Service**  
An application or process running in a container.

**Subdomain**  
A subdivision of a domain (e.g., `n8n.yourdomain.com`).

**Traefik**  
A modern HTTP reverse proxy and load balancer.

**Uptime**  
How long a system has been running without interruption.

**Uptime Kuma**  
A monitoring tool that checks if your services are running.

**Volume (Docker Volume)**  
Persistent storage used by Docker containers to save data.

**YAML**  
A human-readable data format often used for configuration files (`.yml` extension).

---

## One-Page Printable Reference

```
╔══════════════════════════════════════════════════════════════════════╗
║                    DOCKER SERVER QUICK REFERENCE                     ║
╚══════════════════════════════════════════════════════════════════════╝

┌─────────────────────────────────────────────────────────────────────┐
│ CONNECTING                                                          │
└─────────────────────────────────────────────────────────────────────┘
  ssh USERNAME@server-ip                    Connect to server
  exit                                      Disconnect from server

┌─────────────────────────────────────────────────────────────────────┐
│ VIEWING STATUS                                                      │
└─────────────────────────────────────────────────────────────────────┘
  docker compose ps                         Show running containers
  docker stats                              Show resource usage
  df -h                                     Show disk space
  free -h                                   Show memory usage
  ~/docker/health-check.sh                  Run full health check

┌─────────────────────────────────────────────────────────────────────┐
│ VIEWING LOGS                                                        │
└─────────────────────────────────────────────────────────────────────┘
  docker compose logs n8n                   View n8n logs
  docker compose logs -f n8n                Follow logs in real-time
  docker compose logs --tail=50 n8n         Last 50 lines only

┌─────────────────────────────────────────────────────────────────────┐
│ CONTROLLING SERVICES                                                │
└─────────────────────────────────────────────────────────────────────┘
  docker compose up -d                      Start all services
  docker compose down                       Stop all services
  docker compose restart n8n                Restart single service
  docker compose restart                    Restart all services

┌─────────────────────────────────────────────────────────────────────┐
│ BACKUP & RESTORE                                                    │
└─────────────────────────────────────────────────────────────────────┘
  ~/docker/backup.sh                        Create backup
  ls -lh ~/docker/backups/                  List backups
  ~/docker/restore.sh n8n_data file.tar.gz  Restore backup

┌─────────────────────────────────────────────────────────────────────┐
│ IMPORTANT PATHS                                                     │
└─────────────────────────────────────────────────────────────────────┘
  ~/docker/docker-compose.yml               Main configuration
  ~/docker/.env                             Passwords & secrets
  ~/docker/backups/                         Backup location
  ~/docker/traefik.yml                      Traefik config

┌─────────────────────────────────────────────────────────────────────┐
│ SERVICE URLS                                                        │
└─────────────────────────────────────────────────────────────────────┘
  https://n8n.yourdomain.com                n8n automation
  https://traefik.yourdomain.com            Traefik dashboard
  https://uptime.yourdomain.com             Monitoring
  https://phpmyadmin.yourdomain.com         Database admin

┌─────────────────────────────────────────────────────────────────────┐
│ CONTAINER NAMES                                                     │
└─────────────────────────────────────────────────────────────────────┘
  traefik                                   Reverse proxy
  n8n                                       Workflow automation
  db_core                                   MySQL database
  uptime-kuma                               Monitoring
  phpmyadmin                                Database admin

┌─────────────────────────────────────────────────────────────────────┐
│ PORTS                                                               │
└─────────────────────────────────────────────────────────────────────┘
  80                                        HTTP (redirects to 443)
  443                                       HTTPS (main access)
  22                                        SSH

┌─────────────────────────────────────────────────────────────────────┐
│ QUICK TROUBLESHOOTING                                               │
└─────────────────────────────────────────────────────────────────────┘
  Service down?
    1. docker compose ps                    Check status
    2. docker compose logs service_name     Check logs
    3. docker compose restart service_name  Try restart

  Can't access website?
    1. docker compose ps                    All containers up?
    2. docker compose logs traefik          Traefik working?
    3. ping yourdomain.com                  DNS resolving?

  High resource usage?
    1. docker stats                         Which container?
    2. df -h                                Disk full?
    3. free -h                              Memory full?

┌─────────────────────────────────────────────────────────────────────┐
│ EMERGENCY CONTACTS                                                  │
└─────────────────────────────────────────────────────────────────────┘
  Administrator: _______________________
  Phone: _______________________________
  Email: _______________________________

┌─────────────────────────────────────────────────────────────────────┐
│ BEFORE MAKING CHANGES                                               │
└─────────────────────────────────────────────────────────────────────┘
  ☐ Create backup: ~/docker/backup.sh
  ☐ Document what you're doing
  ☐ Check current status: docker compose ps
  ☐ Know how to rollback

┌─────────────────────────────────────────────────────────────────────┐
│ WHEN IN DOUBT                                                       │
└─────────────────────────────────────────────────────────────────────┘
  1. STOP - Don't proceed if unsure
  2. BACKUP - Always create a backup first
  3. ASK - Contact your administrator
  4. DOCUMENT - Keep notes of what happened

╔══════════════════════════════════════════════════════════════════════╗
║  Last Updated: _______________          Server: _______________      ║
╚══════════════════════════════════════════════════════════════════════╝
```

---

## Quick Command Templates

Copy and customize these templates:

### Check Service Status
```bash
# Replace SERVICE_NAME with: traefik, n8n, db_core
docker compose logs --tail=50 SERVICE_NAME
docker compose ps SERVICE_NAME
docker compose restart SERVICE_NAME
```

### Backup Before Changes
```bash
# Always run before making changes
~/docker/backup.sh
ls -lh ~/docker/backups/ | tail -5
```

### View Recent Activity
```bash
# See what happened in the last hour
docker compose logs --since 1h
docker compose logs --since 1h | grep -i error
```

### Check Resource Usage
```bash
# Quick resource check
docker stats --no-stream
df -h
free -h
```

---

## Navigation

**Previous Guide:** [10 - Best Practices & Tips](10-best-practices.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: 02/01/2026*
