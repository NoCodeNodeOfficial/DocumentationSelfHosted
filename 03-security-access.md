# 03 - Security & Access Management

**Difficulty Level:** 🟡 Intermediate  
**Risk Level:** 🟠 Medium - *Be careful when modifying security settings*

Learn how to manage SSH keys, passwords, review login history, and maintain your server's security.

---

## 📋 Table of Contents

1. [Managing SSH Keys](#managing-ssh-keys)
   - [Adding SSH Keys for Team Members](#adding-ssh-keys-for-team-members)
   - [Removing SSH Keys](#removing-ssh-keys)
   - [Viewing Current SSH Keys](#viewing-current-ssh-keys)
2. [Password Management](#password-management)
   - [Changing SSH Password](#changing-ssh-password)
   - [Enabling Password Authentication](#enabling-password-authentication)
   - [Disabling Password Authentication](#disabling-password-authentication)
3. [Reviewing Login History](#reviewing-login-history)
   - [Recent Logins](#recent-logins)
   - [Failed Login Attempts](#failed-login-attempts)
   - [Currently Logged In Users](#currently-logged-in-users)
4. [Fail2ban Management](#fail2ban-management)
   - [Checking Ban Status](#checking-ban-status)
   - [Unbanning an IP Address](#unbanning-an-ip-address)
   - [Whitelisting IP Addresses](#whitelisting-ip-addresses)
5. [Firewall Basics (UFW)](#firewall-basics-ufw)
   - [Checking Firewall Status](#checking-firewall-status)
   - [Opening Ports](#opening-ports)
   - [Closing Ports](#closing-ports)

---

## Managing SSH Keys

SSH keys are the primary way to access your server. Here's how to manage them.

### Adding SSH Keys for Team Members

When you want to give someone access to your server, you'll add their **public key** (not private key!).

#### Step 1: Get Their Public Key

Ask the team member to send you their **public key** (the `.pub` file). It looks like this:

```
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJfF7... user@example.com
```

> ⚠️ **IMPORTANT:** Never ask for their private key! Only the `.pub` file.

---

#### Step 2: Add the Key to the Server

**Method 1: Using nano (Recommended for beginners)**

```bash
# Connect to your server
ssh -p 1077 USERNAME@YOUR_SERVER_IP

# Edit the authorized_keys file
nano ~/.ssh/authorized_keys
```

1. Scroll to the bottom of the file (use arrow keys)
2. Add a new line
3. Paste the public key (right-click or Ctrl+Shift+V)
4. Press `Ctrl + X` to exit
5. Press `Y` to save
6. Press `Enter` to confirm

**Method 2: Using echo (One command)**

```bash
# Connect to your server
ssh -p 1077 USERNAME@YOUR_SERVER_IP

# Add the key directly (replace with actual public key)
echo "ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJfF7... user@example.com" >> ~/.ssh/authorized_keys
```

**Method 3: Using ssh-copy-id (From team member's computer)**

If the team member has access to a terminal and already have functional access to the server, they can add their own key:

```bash
# From their computer (not the server)
ssh-copy-id -p 1077 USERNAME@YOUR_SERVER_IP
```

---

#### Step 3: Verify the Key Was Added

```bash
# View all authorized keys
cat ~/.ssh/authorized_keys
```

---

#### Step 4: Test the Connection

Ask the team member to try connecting:

```bash
ssh -p 1077 USERNAME@YOUR_SERVER_IP
```

> 💡 **TIP:** Add a comment with a hashtag at the end of the line to each key, so you know who it belongs to:
> ```
> ssh-ed25519 AAAAC3Nza... john@company.com # John Doe - Added 2024-01-15
> ```

---

### Removing SSH Keys

When someone leaves your team or no longer needs access, remove their key.

#### Step 1: Identify the Key to Remove

```bash
# View all authorized keys with line numbers
cat -n ~/.ssh/authorized_keys
```

This shows:
```
     1  ssh-ed25519 AAAAC3... admin@company.com
     2  ssh-ed25519 AAAAC3... john@company.com
     3  ssh-ed25519 AAAAC3... jane@company.com
```

---

#### Step 2: Remove the Key

**Method 1: Using nano (Visual editing)**

```bash
nano ~/.ssh/authorized_keys
```

1. Navigate to the line with the key you want to remove
2. Press `Ctrl + K` to delete the entire line
3. Press `Ctrl + X` to exit
4. Press `Y` to save
5. Press `Enter` to confirm

**Method 2: Using sed (Delete specific line)**

```bash
# Delete line 2 (John's key)
sed -i '2d' ~/.ssh/authorized_keys
```

**Method 3: Recreate file without the key**

```bash
# Backup first
cp ~/.ssh/authorized_keys ~/.ssh/authorized_keys.backup

# Create new file with only keys you want to keep
cat > ~/.ssh/authorized_keys << 'EOF'
ssh-ed25519 AAAAC3... admin@company.com
ssh-ed25519 AAAAC3... jane@company.com
EOF
```

---

#### Step 3: Verify the Key Was Removed

```bash
# View remaining keys
cat ~/.ssh/authorized_keys

# Verify permissions are correct
ls -la ~/.ssh/authorized_keys
# Should show: -rw------- (600 permissions)
```

> ⚠️ **WARNING:** If you accidentally delete all keys, you'll be locked out! Always keep at least one working key (yours).

---

### Viewing Current SSH Keys

**See all authorized keys:**
```bash
cat ~/.ssh/authorized_keys
```

**See keys with better formatting:**
```bash
# Show with line numbers and comments
cat -n ~/.ssh/authorized_keys | grep -E "ssh-|#"
```

**Check permissions (should be 600):**
```bash
ls -la ~/.ssh/authorized_keys
```

**Fix permissions if needed:**
```bash
chmod 600 ~/.ssh/authorized_keys
chmod 700 ~/.ssh
```

> ⚠️ **WARNING:** Don't forget that the root user may also have access to the server. His key is in the directory /root/.ssh

---

## Password Management

Your server uses SSH key authentication by default, but passwords can still be reactivated if you want to keep that option. It's a less secure option, but since fail2ban is installed, bruteforce your server is impossible.

### Changing SSH Password

**Change your own password:**

```bash
# Run the passwd command
passwd

# You'll be prompted:
# Current password: [type current password]
# New password: [type new password]
# Retype new password: [confirm new password]
```

**Password requirements:**
- Minimum 8 characters (12+ recommended)
- Mix of uppercase, lowercase, numbers, symbols
- Avoid common words or patterns

**Change another user's password (if you have sudo):**
```bash
sudo passwd USERNAME
```

> 💡 **TIP:** Use a password manager to generate and store strong passwords!

---

### Enabling Password Authentication

By default, password authentication is **disabled** for security. Here's how to enable it if needed.

> ⚠️ **WARNING:** Enabling password authentication reduces security. Only do this if absolutely necessary.

#### Step 1: Edit SSH Configuration

```bash
# Open SSH config file
sudo nano /etc/ssh/sshd_config
```

#### Step 2: Find and Modify These Lines

Look for these settings and change them:

```bash
# Change this:
PasswordAuthentication no

# To this:
PasswordAuthentication yes

```

> 💡 **TIP:** Use `Ctrl + W` in nano to search for "PasswordAuthentication"

#### Step 3: Restart SSH Service

```bash
# Restart SSH to apply changes
sudo systemctl restart sshd

# Verify SSH is still running
sudo systemctl status sshd
```

> ⚠️ **IMPORTANT:** Keep your current SSH session open while testing! Open a new terminal to test password login before closing your original connection.

#### Step 4: Test Password Authentication

**From a new terminal:**
```bash
# Try connecting with password
ssh -p 1077 USERNAME@YOUR_SERVER_IP
# You should now be prompted for a password
```

---

### Disabling Password Authentication

To return to key-only authentication (more secure):

#### Step 1: Edit SSH Configuration

```bash
sudo nano /etc/ssh/sshd_config
```

#### Step 2: Change Settings Back

```bash
# Change these:
PasswordAuthentication yes

# Back to:
PasswordAuthentication no
```

#### Step 3: Restart SSH Service

```bash
sudo systemctl restart sshd
```

> ✅ **BEST PRACTICE:** Always use SSH keys instead of passwords when possible.

---

## Reviewing Login History

Regularly check who's accessing your server and when.

### Recent Logins

**View recent successful logins:**

```bash
# Show last 20 logins
last -n 20

# Example output:
# admin    pts/0    192.168.1.100   Mon Jan 15 14:32   still logged in
# admin    pts/0    192.168.1.100   Mon Jan 15 10:15 - 12:30  (02:15)
```

**View last login for each user:**
```bash
lastlog

# Example output:
# Username    Port     From             Latest
# admin       pts/0    192.168.1.100    Mon Jan 15 14:32:10 +0000 2024
# john        pts/1    203.0.113.50     Sun Jan 14 09:21:43 +0000 2024
```

**View last login for specific user:**
```bash
lastlog -u USERNAME
```

**Filter by time:**
```bash
# Logins in the last 7 days
last -s -7days

# Logins since specific date
last -s 20240101
```

---

### Failed Login Attempts

**Check authentication log for failed attempts:**

```bash
# View recent auth events
sudo tail -n 50 /var/log/auth.log

# Search for failed password attempts
sudo grep "Failed password" /var/log/auth.log | tail -n 20

# Search for failed SSH key attempts
sudo grep "publickey" /var/log/auth.log | grep "Failed" | tail -n 20
```

**Count failed attempts by IP:**
```bash
sudo grep "Failed password" /var/log/auth.log | awk '{print $(NF-3)}' | sort | uniq -c | sort -nr
```

**Example output:**
```
     15 203.0.113.45
      8 198.51.100.23
      3 192.0.2.100
```

> 🔍 **INTERPRETATION:** If you see many failed attempts from unknown IPs, that's normal - bots constantly scan for SSH servers. That's why fail2ban is installed!

---

### Currently Logged In Users

**See who's logged in right now:**

```bash
# Simple view
who

# Detailed view with idle time
w

# Example output:
# USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
# admin    pts/0    192.168.1.100    14:32    0.00s  0.04s  0.00s w
# john     pts/1    203.0.113.50     15:10    5:00   0.02s  0.02s vim
```

**See active SSH connections:**
```bash
# List all SSH connections
ss -tnp | grep :1077

# Or using netstat
sudo netstat -tnp | grep :1077
```

---

## Fail2ban Management

Fail2ban automatically blocks IPs that show malicious behavior (too many failed login attempts).

### Checking Ban Status

**View fail2ban status:**

```bash
# Overall status
sudo fail2ban-client status

# Example output:
Status
|- Number of jail:	3
`- Jail list:	port-scan, ssh-username-scan, sshd
```

**Check specific jail (sshd) status:**
```bash
sudo fail2ban-client status port-scan

# Example output:
Status for the jail: port-scan
|- Filter
|  |- Currently failed:	598
|  |- Total failed:	27725
|  `- Journal matches:
`- Actions
   |- Currently banned:	354
   |- Total banned:	354
   `- Banned IP list:	198.12.88.142 80.94.95.115 91.191.209.54 [....]
```

**View fail2ban logs:**
```bash
# Recent fail2ban activity
sudo tail -n 50 /var/log/fail2ban.log

# Search for bans
sudo grep "Ban" /var/log/fail2ban.log | tail -n 20

# Search for unbans
sudo grep "Unban" /var/log/fail2ban.log | tail -n 20
```

---

### Unbanning an IP Address

If someone (or you!) gets accidentally banned, here's how to unban them:

**Unban a specific IP:**
```bash
# Unban the IP
# sudo fail2ban-client set [jail-name] unbanip [ip-to-unban]
sudo fail2ban-client set sshd unbanip 203.0.113.45

# Verify it's unbanned
sudo fail2ban-client status sshd
```

**Unban all IPs (nuclear option):**
```bash
# Stop fail2ban
sudo systemctl stop fail2ban

# Start fail2ban (all bans cleared)
sudo systemctl start fail2ban
```

> 💡 **TIP:** Write down the unbanning command somewhere safe in case you lock yourself out!

---

### Whitelisting IP Addresses

Prevent specific IPs from ever being banned (like your office IP or home IP).

#### Step 1: Edit fail2ban Configuration

```bash
# Create local config (preferred method)
sudo nano /etc/fail2ban/jail.local
```

#### Step 2: Add Whitelist

```ini
# Add these lines at the top
[DEFAULT]
ignoreip = 127.0.0.1/8 ::1
           YOUR_HOME_IP
           YOUR_OFFICE_IP
           192.168.1.0/24

# Example:
ignoreip = 127.0.0.1/8 ::1
           203.0.113.45
           198.51.100.23
           192.168.1.0/24
```

**IP format options:**
- Single IP: `203.0.113.45`
- IP range (CIDR): `192.168.1.0/24` (all IPs from .1 to .254)
- Multiple IPs: List on separate lines with proper indentation

#### Step 3: Restart fail2ban

```bash
# Test configuration first
sudo fail2ban-client -t

# If no errors, restart
sudo systemctl restart fail2ban

# Verify it's running
sudo systemctl status fail2ban
```

#### Step 4: Verify Whitelist

```bash
# Check if your IPs are in the ignore list
sudo fail2ban-client get sshd ignoreip
```

---

### Adjusting Ban Time

**Change how long IPs stay banned:**

```bash
# Edit local config
sudo nano /etc/fail2ban/jail.local
```

Add or modify:
```ini
[sshd]
enabled = true
port = 1077
bantime = 3600        # Ban for 1 hour (in seconds)
findtime = 600        # Look for failures in 10-minute window
maxretry = 5          # Ban after 5 failed attempts
```

**Common bantime values:**
- `600` = 10 minutes
- `3600` = 1 hour
- `86400` = 24 hours
- `-1` = Permanent ban

**Restart fail2ban after changes:**
```bash
sudo systemctl restart fail2ban
```
> ℹ️ **INFO:** I've set up your server to ban any IP permanently.

---

## Firewall Basics (UFW)

UFW (Uncomplicated Firewall) protects your server by controlling which ports are accessible.

### Checking Firewall Status

**View firewall status:**
```bash
sudo ufw status

# Example output:
Status: active

To                         Action      From
--                         ------      ----
80/tcp                     ALLOW       Anywhere                   # http
443/tcp                    ALLOW       Anywhere                   # https
1077/tcp                   ALLOW       Anywhere                   # ssh
80/tcp (v6)                ALLOW       Anywhere (v6)              # http
443/tcp (v6)               ALLOW       Anywhere (v6)              # https
1077/tcp (v6)              ALLOW       Anywhere (v6)              # ssh
```

```bash
# Since you have fail2ban, you will have many rule based on banned IPs, in order to show only opened port, you can filter out the REJECT rules
sudo ufw status | grep ALLOW

# Example output:
80/tcp                     ALLOW       Anywhere                   # http
443/tcp                    ALLOW       Anywhere                   # https
1077/tcp                   ALLOW       Anywhere                   # ssh
80/tcp (v6)                ALLOW       Anywhere (v6)              # http
443/tcp (v6)               ALLOW       Anywhere (v6)              # https
1077/tcp (v6)              ALLOW       Anywhere (v6)              # ssh
```

**View numbered rules (easier to delete):**
```bash
sudo ufw status numbered

# Example output:
[356] 80/tcp                     ALLOW IN    Anywhere                   # http
[357] 443/tcp                    ALLOW IN    Anywhere                   # https
[358] 1077/tcp                   ALLOW IN    Anywhere                   # ssh
[359] Anywhere (v6)              REJECT IN   fe80::f816:3eff:fe15:f6bd  # by Fail2Ban after 5 attempts against port-scan
[360] 80/tcp (v6)                ALLOW IN    Anywhere (v6)              # http
[361] 443/tcp (v6)               ALLOW IN    Anywhere (v6)              # https
[362] 1077/tcp (v6)              ALLOW IN    Anywhere (v6)              # ssh
```

**View detailed status:**
```bash
sudo ufw status verbose
```

---

### Opening Ports

> ⚠️ **WARNING:** Only open ports you actually need. Every open port is a potential security risk.

**Open a single port:**
```bash
# Allow port 8080
sudo ufw allow 8080/tcp

# Verify it was added
sudo ufw status
```

**Open a port range:**
```bash
# Allow ports 8000-9000
sudo ufw allow 8000:9000/tcp
```

**Allow from specific IP:**
```bash
# Only allow 203.0.113.45 to access port 3306
sudo ufw allow from 203.0.113.45 to any port 3306
```

**Common ports you might need:**
- `80` - HTTP (web traffic)
- `443` - HTTPS (secure web traffic)
- `1077` - SSH (already open)
- `3306` - MySQL (database)
- `5432` - PostgreSQL (database)
- `6379` - Redis (cache)

---

### Closing Ports

**Delete a rule by number:**
```bash
# First, view numbered rules
sudo ufw status numbered

# Delete rule number 4
sudo ufw delete 4

# Confirm when prompted
```

**Delete a rule by specification:**
```bash
# Remove port 8080
sudo ufw delete allow 8080/tcp

# Remove specific IP rule
sudo ufw delete allow from 203.0.113.45 to any port 3306
```

**Deny a port (block explicitly):**
```bash
# Block port 25 (SMTP)
sudo ufw deny 25/tcp
```

---

### Firewall Best Practices

**1. Default deny incoming:**
```bash
# This should already be set
sudo ufw default deny incoming
sudo ufw default allow outgoing
```

**2. Always keep SSH open:**
```bash
# Before making changes, ensure SSH is allowed
sudo ufw allow 1077/tcp
```

**3. Enable logging (helpful for debugging):**
```bash
# Enable logging
sudo ufw logging on

# View logs
sudo tail -f /var/log/ufw.log
```

**4. Reload after changes:**
```bash
# Reload firewall rules
sudo ufw reload
```

> ⚠️ **CRITICAL WARNING:** Never disable UFW or delete the SSH port rule (1077/tcp) or you'll lock yourself out!

> 🚨 **CRITICAL ALERT:** If you add new Docker container and expose their port, Docker will automatically open their port, and it will have priority on any pre-existing firewall rules. 
---

### Emergency: I Locked Myself Out!

If you accidentally block SSH and can't connect:

**Option 1: Cloud Provider Console**
- Log into your hosting provider's control panel
- Access the server console (web-based terminal)
- Re-enable SSH access:
```bash
sudo ufw allow 1077/tcp
sudo ufw reload
```

**Option 2: Disable UFW temporarily**
```bash
# From console
sudo ufw disable
# Fix your rules
sudo ufw allow 1077/tcp
sudo ufw enable
```

---

## 🎓 What You've Learned

You should now be able to:
- ✅ Add and remove SSH keys for team members
- ✅ Manage passwords and authentication methods
- ✅ Review who's accessing your server
- ✅ Monitor and manage fail2ban
- ✅ Control firewall rules with UFW
- ✅ Troubleshoot access issues

---

## 📌 Security Checklist

Use this checklist monthly:

- [ ] Review authorized SSH keys (remove old team members)
- [ ] Check failed login attempts (sudo grep "Failed" /var/log/auth.log)
- [ ] Review fail2ban status (sudo fail2ban-client status sshd)
- [ ] Verify firewall rules (sudo ufw status)
- [ ] Check for security updates (sudo apt update && sudo apt list --upgradable)
- [ ] Review recent logins (last -n 50)
- [ ] Verify strong passwords are set
- [ ] Backup authorized_keys file

---

## 📌 Quick Reference Commands

```bash
# SSH Keys
cat ~/.ssh/authorized_keys                    # View authorized keys
nano ~/.ssh/authorized_keys                   # Edit authorized keys
chmod 600 ~/.ssh/authorized_keys              # Fix permissions

# Password
passwd                                         # Change your password

# Login History
last -n 20                                    # Recent logins
lastlog                                       # Last login per user
who                                           # Currently logged in
sudo grep "Failed" /var/log/auth.log          # Failed attempts

# Fail2ban
sudo fail2ban-client status sshd              # Check bans
sudo fail2ban-client set sshd unbanip IP      # Unban IP

# Firewall
sudo ufw status                               # Check firewall
sudo ufw allow PORT/tcp                       # Open port
sudo ufw delete allow PORT/tcp                # Close port
```

---

**Next Guide:** [04 - Linux Command Cheatsheet](04-linux-cheatsheet.md)  
**Previous Guide:** [02 - SSH Connection Guide](02-ssh-guide.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: 29/12/2025*
