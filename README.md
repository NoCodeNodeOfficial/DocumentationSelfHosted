# n8n Self-Hosted Server - Administration Documentation

Welcome to your n8n self-hosted server documentation. This guide will help you manage and maintain your server independently.

## 📋 Documentation Overview

This documentation is organized into focused guides to help you find what you need quickly. Each guide includes step-by-step instructions with examples for different operating systems where applicable.

## 📚 Table of Contents

### 🎯 Getting Started
- **[01 - Getting Started](01-getting-started.md)** 🟢 *Start here!*
  - Initial setup verification
  - First SSH connection
  - Service health checks
  - Changing default passwords

- **[02 - SSH Connection Guide](02-ssh-guide.md)** 🟢
  - SSH key setup
  - Connection instructions
  - Troubleshooting connection issues

### 🔒 Security & Management
- **[03 - Security & Access Management](03-security-access.md)** 🟡
  - SSH key management
  - Password management
  - Login history review
  - Fail2ban management
  - Firewall basics

### 📖 Command References
- **[04 - Linux Command Cheatsheet](04-linux-cheatsheet.md)** 🟢
  - Essential Linux commands
  - File operations
  - System monitoring

- **[05 - Docker Management Cheatsheet](05-docker-cheatsheet.md)** 🟢
  - Docker commands
  - Container management
  - Docker Compose operations

### 💾 Backup & Updates
- **[06 - Backup & Recovery](06-backup-recovery.md)** 🟡
  - Backup procedures
  - Restore procedures
  - Backup verification

- **[07 - Monitoring & Maintenance](07-monitoring-maintenance.md)** 🟡
  - System monitoring
  - Health checks
  - Routine maintenance

- **[08 - Updates & Upgrades](08-updates-upgrades.md)** 🟡
  - Updating services
  - System updates
  - Update best practices

### 🚨 Emergency & Troubleshooting
- **[09 - Emergency Procedures](09-emergency-procedures.md)** 🔴
  - Server completely down
  - Data corruption
  - Security breach response
  - Emergency contacts

### 📚 Best Practices & Reference
- **[10 - Best Practices & Tips](10-best-practices.md)** 🟢
  - Daily operations
  - Documentation tips
  - Troubleshooting approach

- **[11 - Quick Reference & Cheat Sheet](11-quick-reference-cheat-sheet.md)** 📌 *Print this!*
  - One-page command reference
  - Important file paths
  - Port reference
  - Common tasks

---

## 🎯 Quick Navigation by Task

**I need to...**
- 🔐 Connect to my server → [SSH Connection Guide](02-ssh-guide.md)
- 💾 Backup my data → [Backup & Recovery](06-backup-recovery.md)
- 🔄 Update n8n → [Updates & Upgrades](08-updates-upgrades.md)
- ❌ Fix a problem → [09 - Emergency Procedures](09-emergency-procedures.md)
- 👥 Add a team member → [Security & Access Management](03-security-access.md)
- 📊 Check server health → [Monitoring & Maintenance](07-monitoring-maintenance.md)

## 📞 Support Information

**Your Setup Details:**
- Server IP: `YOUR_SERVER_IP`
- SSH Port: `1077`
- Domain: `yourdomain.website`
- Username: `USERNAME`

**Emergency Contact:**
- If you've purchased ongoing support, contact details are in your credentials PDF
- For critical issues, refer to [09 - Emergency Procedures](09-emergency-procedures.md)

## 🛠️ Available Scripts

All maintenance scripts are located in `~/docker/maintenance/`:

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

**Note:** No cron jobs are configured by default. Set up automation based on your needs.

## 📝 Documentation

- **User Documentation:** This repository (guides for server users)
- **Server Changelog:** `~/docker/doc/CHANGELOG.md` (server admin changes)
- **Doc Changelog:** [CHANGELOG.md](CHANGELOG.md) (documentation updates)

---

## 🚀 First Time Here?

If this is your first time accessing the documentation, please start with:
1. [Getting Started Guide](01-getting-started.md) - Initial setup and verification
2. [SSH Connection Guide](02-ssh-guide.md) - Connect to your server
3. [11 - Quick Reference & Cheat Sheet](11-quick-reference-cheat-sheet.md) - Bookmark this for daily use

---

## 📖 About This Documentation

**Difficulty Levels:**
- 🟢 **Beginner** - Safe for new users
- 🟡 **Intermediate** - Requires basic understanding
- 🔴 **Advanced** - Proceed with caution

**Risk Indicators:**
- ✅ **Safe** - No risk to data or services
- ⚠️ **Caution** - Could affect running services
- 🚨 **Dangerous** - Could cause data loss or downtime

Each guide includes these indicators to help you assess before proceeding.

---

**Last Updated:** 29/12/2025 
**Version:** 1.0
