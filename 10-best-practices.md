# 10 - Best Practices & Tips

**Difficulty Level:** 🟢 Beginner  
**Risk Level:** 🟢 Low - *Guidelines for safe and efficient server management*

Essential best practices, helpful tips, and recommendations for maintaining a healthy, secure, and efficient server environment.

---

## 📋 Table of Contents

1. [Daily Habits](#daily-habits)
2. [Security Best Practices](#security-best-practices)
3. [Backup Best Practices](#backup-best-practices)
4. [Docker Management Tips](#docker-management-tips)
5. [Performance Optimization](#performance-optimization)
6. [Documentation & Organization](#documentation--organization)
7. [Troubleshooting Tips](#troubleshooting-tips)
8. [Common Mistakes to Avoid](#common-mistakes-to-avoid)
9. [Useful Aliases & Shortcuts](#useful-aliases--shortcuts)
10. [Quick Reference Checklist](#quick-reference-checklist)

---

## Daily Habits

### Quick Morning Check (2 minutes)

```bash
# Connect to server
ssh your-server

# Quick health check
docker compose ps
df -h
free -h

# Check for issues
docker compose logs --tail=20

# Done!
exit
```

**What to look for:**
- ✅ All containers show "Up" status
- ✅ Disk usage below 80%
- ✅ No error messages in logs

### Weekly Routine (10 minutes)

**Every Monday morning:**

```bash
# 1. Check system health
~/docker/health-check.sh

# 2. Review backup status
ls -lh ~/docker/backups/ | tail -10

# 3. Check for available updates
sudo apt update
docker images

# 4. Review security
sudo last -n 20
sudo grep "Failed password" /var/log/auth.log | tail -10
```

### Monthly Tasks (30 minutes)

**First of every month:**

1. **Update everything:**
   ```bash
   sudo apt update && sudo apt upgrade -y
   docker compose pull
   docker compose up -d
   ```

2. **Clean up old backups:**
   ```bash
   # Keep only last 30 days
   find ~/docker/backups -name "*.tar.gz" -mtime +30 -delete
   ```

3. **Review disk space:**
   ```bash
   ncdu /home
   docker system df
   ```

4. **Test a backup restoration** (on test environment if possible)

5. **Review access logs** for any suspicious activity

---

## Security Best Practices

### SSH Security

**✅ DO:**
- Use SSH keys instead of passwords
- Keep your private key secure and password-protected
- Use a non-standard SSH port if possible
- Regularly review `~/.ssh/authorized_keys`
- Keep SSH sessions short when not actively working

**❌ DON'T:**
- Share your SSH private key
- Leave SSH sessions open indefinitely
- Use root account for daily tasks
- Ignore failed login attempts
- Copy SSH keys over unsecured channels

### Password Management

```bash
# Good password practices
- Use unique passwords for each service
- Minimum 16 characters
- Mix of letters, numbers, symbols
- Store in a password manager

# Change passwords if:
- You suspect a breach
- An employee/contractor leaves
- Every 6-12 months (recommended)
```

### Access Control

```bash
# Review who has access
sudo last | head -20

# Check active sessions
who

# Review sudo usage
sudo grep sudo /var/log/auth.log | tail -20
```

> ⚠️ **IMPORTANT:** Immediately revoke access for anyone who no longer needs it!

### Keep Software Updated

```bash
# Check for security updates
sudo apt update
sudo apt list --upgradable

# Apply security updates
sudo apt upgrade -y

# Check Docker images
docker images
```

> 💡 **TIP:** Subscribe to security mailing lists for Ubuntu and your applications to stay informed about critical updates.

---

## Backup Best Practices

### The 3-2-1 Rule

**3** copies of your data  
**2** different storage types  
**1** off-site copy

**Implementation:**
```
1st copy: Production data (server)
2nd copy: Local backups (~/docker/backups)
3rd copy: Off-site backups (downloaded to your computer)
```

### Backup Schedule

```bash
# Automated (via cron)
Daily:   Full backup at 2 AM
Weekly:  Download to local computer
Monthly: Archive to external drive

# Manual before changes
Before: Any major update or configuration change
Before: Container upgrades
Before: System updates
```

### Test Your Backups!

```bash
# Monthly restoration test
# Use a test environment or verify backup integrity

# Quick verification
tar -tzf ~/docker/backups/n8n_data-YYYYMMDD-HHMMSS.tar.gz > /dev/null
echo "Backup integrity: OK"
```

> ⚠️ **IMPORTANT:** An untested backup is not a backup! Test restoration quarterly.

### Backup Hygiene

```bash
# Keep backups organized
~/docker/backups/
├── daily/     # Last 7 days
├── weekly/    # Last 4 weeks
├── monthly/   # Last 12 months
└── archive/   # Important milestones

# Automate cleanup
# Delete backups older than 30 days
find ~/docker/backups/daily -name "*.tar.gz" -mtime +30 -delete
```

---

## Docker Management Tips

### Container Health

```bash
# Check container resource usage
docker stats --no-stream

# Monitor specific container
docker stats n8n --no-stream

# Check container logs
docker compose logs --tail=50 --follow
```

**Expected resource usage:**
```
n8n:          ~200-500 MB RAM (normal operation)
MySQL:        ~400-800 MB RAM (depends on data)
Traefik:      ~50-100 MB RAM
```

> 💡 **TIP:** If a container is using significantly more resources than usual, check its logs for issues.

### Image Management

```bash
# Update images regularly
cd ~/docker
docker compose pull

# Check for unused images
docker images

# Clean up old images (saves space)
docker image prune -a

# See what Docker is using
docker system df
```

### Log Management

```bash
# Prevent logs from growing too large
# Add to docker-compose.yml for each service:

logging:
  driver: "json-file"
  options:
    max-size: "10m"
    max-file: "3"

# View logs efficiently
docker compose logs --tail=100 n8n
docker compose logs --since=1h
docker compose logs --follow
```

### Clean Restarts

```bash
# Graceful restart (preferred)
docker compose restart n8n

# Full restart with cleanup
docker compose down
docker compose up -d

# Nuclear option (only if needed)
docker compose down
docker system prune -a
docker compose up -d
```

> ⚠️ **IMPORTANT:** Always check logs after restarting: `docker compose logs`

---

## Performance Optimization

### Disk Space Management

```bash
# Check disk usage
df -h
du -sh ~/docker/*

# Find large files
ncdu /home

# Clean Docker cache
docker system prune -a --volumes  # ⚠️ Careful with --volumes!

# Clean package cache
sudo apt clean
sudo apt autoremove
```

**Disk space thresholds:**
- 🟢 < 70% - Healthy
- 🟡 70-85% - Monitor
- 🔴 > 85% - Take action
- ⚠️ > 95% - Critical

### Memory Management

```bash
# Check memory usage
free -h

# See what's using memory
top
# Press 'M' to sort by memory

# Check swap usage
swapon --show
```

**Memory optimization tips:**
- Restart containers that have been running for weeks
- Check for memory leaks (gradual increase over time)
- Consider upgrading server if consistently at 80%+ usage

### Network Performance

```bash
# Check network connectivity
ping -c 4 google.com

# Check DNS resolution
nslookup n8n.yourdomain.website

# Test SSL certificate
curl -I https://n8n.yourdomain.website

# Check port accessibility
sudo netstat -tlnp | grep -E ':(80|443|1077)'
```

---

## Documentation & Organization

### Keep a Change Log

Create `~/docker/CHANGELOG.md`:

```markdown
# Server Change Log

## 2024-01-15
- Updated n8n to version 1.22.0
- Added Uptime Kuma monitoring
- Backed up all data before changes

## 2024-01-10
- Restarted Traefik to apply changes

## 2024-01-05
- System update: Ubuntu security patches
- No issues observed
```

### Document Your Setup

Create `~/docker/SETUP.md`:

```markdown
# Server Setup Documentation

## Server Details
- Provider: HostVDS
- Plan: 2 vCPU, 4GB RAM
- IP: XXX.XXX.XXX.XXX
- Domain: your-domain.com

## Installed Services
- n8n: Workflow automation
- MySQL: Database
- Traefik: Reverse proxy

## Important Dates
- Server created: 2024-01-01
- Last major update: 2024-01-15
- SSL renewal: Auto (Let's Encrypt)

## Emergency Contacts
- System Admin: [Contact info]
- Provider Support: [Contact info]
```

### Command History

```bash
# Keep useful commands documented
nano ~/docker/useful-commands.txt

# Example content:
# Backup all volumes
cd ~/docker && ./backup.sh

# Check disk space
df -h && docker system df

# View n8n logs
docker compose logs n8n --tail=100

# Restart all containers
docker compose restart
```

---

## Troubleshooting Tips

### When Something Goes Wrong

**Step-by-step approach:**

1. **Don't panic** - Take a breath
2. **Document** - What were you doing when it happened?
3. **Check basics** - Are containers running? Is server accessible?
4. **Review logs** - What do the logs say?
5. **Try simple fixes** - Restart, check disk space
6. **Search** - Google the error message
7. **Escalate** - Contact your admin if needed

### Common Issues Quick Fix

```bash
# Container won't start
docker compose logs [container-name]
docker compose restart [container-name]

# Out of disk space
docker system prune -a
sudo apt clean

# Can't access via browser
# Check if ports are open
sudo netstat -tlnp | grep -E ':(80|443)'

# SSL certificate issues
docker compose restart traefik
docker compose logs traefik

# Database connection issues
docker compose restart mysql
docker compose logs mysql
```

### Debug Mode

```bash
# Run containers in foreground to see output
docker compose up

# See real-time logs
docker compose logs --follow

# Check container details
docker inspect n8n

# Get a shell inside container
docker exec -it n8n /bin/sh
```

> 💡 **TIP:** Before making changes, always create a backup: `~/docker/backup.sh`

---

## Common Mistakes to Avoid

### ❌ Don't Do This

**1. Running updates without backups**
```bash
# WRONG
docker compose pull && docker compose up -d

# RIGHT
~/docker/backup.sh
docker compose pull && docker compose up -d
docker compose logs
```

**2. Ignoring disk space warnings**
```bash
# If you see disk usage > 85%, act immediately!
# Don't wait until it hits 100%
```

**3. Using `docker system prune --volumes` without thinking**
```bash
# This DELETES all unused volumes (including backups!)
# Only use if you know what you're doing
```

**4. Editing files without backups**
```bash
# WRONG
nano docker-compose.yml

# RIGHT
cp docker-compose.yml docker-compose.yml.backup
nano docker-compose.yml
```

**5. Leaving sensitive data in logs or history**
```bash
# Clear sensitive commands from history
history -c

# Don't commit passwords to files
# Use environment variables instead
```

**6. Running as root unnecessarily**
```bash
# Use sudo only when needed
# Don't: sudo su - and stay as root
```

**7. Not testing before implementing**
```bash
# Test commands in a safe way first
# Read documentation before running unknown commands
```

### ✅ Always Do This

1. **Backup before changes**
2. **Read the full command before running**
3. **Check logs after changes**
4. **Document what you did**
5. **Test in small steps**
6. **Keep credentials secure**
7. **Update regularly**
8. **Monitor resource usage**

---

## Useful Aliases & Shortcuts

### Create Bash Aliases

Add these to `~/.bashrc`:

```bash
# Edit aliases
nano ~/.bashrc

# Add these at the end:
alias dps='docker compose ps'
alias dlogs='docker compose logs'
alias dup='docker compose up -d'
alias ddown='docker compose down'
alias drestart='docker compose restart'
alias dbackup='cd ~/docker && ./backup.sh'
alias dhealth='cd ~/docker && ./health-check.sh'
alias diskspace='df -h && echo && docker system df'
alias update-check='sudo apt update && apt list --upgradable'

# Save and reload
source ~/.bashrc
```

### Usage Examples

```bash
# Now you can use short commands
dps              # Instead of: docker compose ps
dlogs n8n        # Instead of: docker compose logs n8n
dbackup          # Instead of: cd ~/docker && ./backup.sh
diskspace        # Check both system and Docker disk usage
```

### Useful One-Liners

```bash
# Complete health check
dps && diskspace && docker compose logs --tail=10

# Update everything
dbackup && docker compose pull && dup && dlogs

# Check for errors
docker compose logs | grep -i error

# See container resource usage
docker stats --no-stream --format "table {{.Name}}\t{{.CPUPerc}}\t{{.MemUsage}}"

# List all containers with their status
docker ps -a --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
```

---

## Quick Reference Checklist

### Before Making Changes

- [ ] Create backup of affected data  
- [ ] Document what you're about to do  
- [ ] Check current status (docker compose ps)
- [ ] Check disk space (df -h)
- [ ] Review relevant documentation
- [ ] Have rollback plan ready

### After Making Changes

- [ ] Check container status (docker compose ps)
- [ ] Review logs for errors (docker compose logs)
- [ ] Test functionality (access via browser)
- [ ] Document what you changed
- [ ] Monitor for 10-15 minutes
- [ ] Clean up temporary files if any

### Monthly Security Check

- [ ] Apply system updates (apt upgrade)
- [ ] Update Docker containers (docker compose pull)
- [ ] Review failed login attempts
- [ ] Check backup integrity
- [ ] Review disk usage
- [ ] Test backup restoration
- [ ] Update documentation
- [ ] Review access logs

### Quarterly Review

- [ ] Full system backup
- [ ] Review all installed software
- [ ] Check for EOL (End of Life) software
- [ ] Review security settings
- [ ] Test disaster recovery plan
- [ ] Update emergency procedures
- [ ] Review and update documentation
- [ ] Clean up old backups/logs

---

## Final Tips

### Stay Organized

```bash
# Keep your home directory clean
~/
├── docker/              # All Docker configs
│   ├── docker-compose.yml
│   ├── backup.sh
│   ├── restore.sh
│   ├── backups/         # Backup files
│   ├── CHANGELOG.md     # Track changes
│   └── SETUP.md         # Document setup
└── .ssh/                # SSH keys
```

### Keep Learning

- Read error messages carefully
- Google error messages (often someone else solved it)
- Check official documentation
- Ask your administrator when unsure
- Document solutions for future reference

### Trust Your Instincts

> If something feels wrong, it probably is. Stop and investigate.

### When in Doubt

1. **Stop** - Don't proceed if unsure
2. **Backup** - Always have a backup
3. **Ask** - Contact your administrator
4. **Document** - Keep notes of what happened

---

## Navigation

**Previous Guide:** [09 - Emergency Procedures](09-emergency-procedures.md)  
**Next Guide:** [11 - Quick Reference & Cheat Sheet](11-quick-reference-cheat-sheet.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: 02/01/2026*
