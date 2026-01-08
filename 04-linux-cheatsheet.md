# 04 - Linux Command Cheatsheet

**Difficulty Level:** 🟢 Beginner  
**Risk Level:** 🟢 Safe - *These are read-only or basic commands*

Essential Linux commands you'll need for daily server management. No prior Linux experience required!

---

## 📋 Table of Contents

1. [File System Navigation](#file-system-navigation)
2. [Viewing & Reading Files](#viewing--reading-files)
3. [Creating & Editing Files](#creating--editing-files)
4. [Copying, Moving & Deleting](#copying-moving--deleting)
5. [File Permissions](#file-permissions)
6. [Searching & Finding](#searching--finding)
7. [System Information](#system-information)
8. [Process Management](#process-management)
9. [Disk Usage](#disk-usage)
10. [User & Group Management](#user--group-management)
11. [Package Management](#package-management)
12. [Network Commands](#network-commands)
13. [Helpful Tips & Tricks](#helpful-tips--tricks)

---

## File System Navigation

Understanding where you are and how to move around is the foundation of Linux.

### Understanding the Linux File System

```
/                           # Root directory (top of everything)
├── home/                   # User home directories
│   └── USERNAME/           # Your home directory
│       └── docker/         # Your Docker setup
│           ├── maintenance/# Maintenance scripts
│           └── backups/    # Backups
├── etc/                    # System configuration files
├── var/                    # Variable data (logs, databases)
│   └── log/                # Log files
└── opt/                    # Optional software packages
```

---

### Basic Navigation Commands

**pwd** - Print Working Directory (where am I?)

```bash
pwd
# Output: 
/home/USERNAME/docker
```

**ls** - List files and directories

```bash
# Simple list
ls

# Detailed list with permissions, size, date
ls -l

# Include hidden files (start with .)
ls -a

# Human-readable sizes
ls -lh

# Sort by modification time (newest first)
ls -lt

# Combine options
ls -lah
```

**Example `ls -lh` output:**
```
drwxr-xr-x 2 username username 4.0K Jan 15 10:30 docker
-rw-r--r-- 1 username username 2.1K Jan 15 10:25 docker-compose.yml
-rw------- 1 username username  512 Jan 15 09:15 .env
drwxr-xr-x 3 username username 4.0K Jan 14 15:20 my.cnf
```

**Understanding the output:**
- `drwxr-xr-x` - Permissions (d=directory, -=file)
- `username username` - Owner and group
- `4.0K` - File/folder size
- `Jan 15 10:30` - Last modified date/time
- `docker` - Name

---

**cd** - Change Directory

```bash
# Go to your home directory
cd
cd ~

# Go to docker folder
cd ~/docker
cd /home/USERNAME/docker

# Go up one level
cd ..

# Go up two levels
cd ../..

# Go to previous directory
cd -

# Go to root directory
cd /

# Navigate to specific path
cd /var/log
```

**Pro tips:**
```bash
# See where you'll end up before changing
ls /path/to/directory
cd /path/to/directory

# Use tab completion (press Tab key)
cd ~/doc[TAB]  # Autocompletes to ~/docker/
```

---

### Path Types

**Absolute path** - Starts from root (/)
```bash
cd /home/USERNAME/docker
```

**Relative path** - Relative to current location
```bash
# If you're in /home/USERNAME
cd docker

# If you're in /home
cd USERNAME/docker
```

**Special shortcuts:**
- `~` - Your home directory (`/home/USERNAME`)
- `.` - Current directory
- `..` - Parent directory (one level up)
- `-` - Previous directory

---

## Viewing & Reading Files

### cat - View entire file

```bash
# Display entire file
cat docker-compose.yml

# Display with line numbers
cat -n docker-compose.yml

# Display multiple files
cat file1.txt file2.txt
```

> ⚠️ **NOTE:** `cat` shows the entire file at once. For large files, use `less` instead.

---

### less - View file page by page

```bash
# View file with pagination
less docker-compose.yml
```

**Navigation inside `less`:**
- `Space` or `Page Down` - Next page
- `b` or `Page Up` - Previous page
- `G` - Go to end of file
- `g` - Go to beginning of file
- `/searchterm` - Search forward
- `?searchterm` - Search backward
- `n` - Next search result
- `q` - Quit

---

### head - View first lines

```bash
# First 10 lines (default)
head docker-compose.yml

# First 20 lines
head -n 20 docker-compose.yml

# First 5 lines
head -5 docker-compose.yml
```

---

### tail - View last lines

```bash
# Last 10 lines (default)
tail docker-compose.yml

# Last 20 lines
tail -n 20 docker-compose.yml

# Follow file in real-time (great for logs!)
tail -f /var/log/auth.log

# Follow with line numbers
tail -fn 50 /var/log/auth.log

# Stop following: Ctrl + C
```

> 💡 **TIP:** `tail -f` is perfect for watching log files live!

---

### grep - Search inside files

```bash
# Search for text in a file
grep "n8n" docker-compose.yml

# Case-insensitive search
grep -i "N8N" docker-compose.yml

# Show line numbers
grep -n "n8n" docker-compose.yml

# Search recursively in directory
grep -r "error" /var/log/

# Search with context (3 lines before/after)
grep -C 3 "error" logfile.txt

# Count matches
grep -c "n8n" docker-compose.yml

# Invert match (lines that DON'T contain text)
grep -v "comment" docker-compose.yml

# Multiple patterns
grep -E "n8n|mysql|traefik" docker-compose.yml
```

**Real-world examples:**
```bash
# Find your IP in logs
grep "YOUR_IP" /var/log/auth.log

# Find errors in last 100 lines
tail -n 100 /var/log/syslog | grep -i error

# Find failed login attempts
sudo grep "Failed password" /var/log/auth.log

# Find Docker containers in compose file
grep "container_name:" docker-compose.yml
```

---

## Creating & Editing Files

### nano - Simple text editor (recommended for beginners)

```bash
# Create or edit a file
nano filename.txt

# Edit with sudo (for system files)
sudo nano /etc/hosts
```

**nano keyboard shortcuts:** (do not change CTRL for ⌘ on MACOS)
- `Ctrl + O` - Save (Write Out)
- `Ctrl + X` - Exit
- `Ctrl + K` - Cut line
- `Ctrl + U` - Paste line
- `Ctrl + W` - Search
- `Ctrl + \` - Search and replace
- `Ctrl + G` - Show help
- `Alt + U` - Undo
- `Alt + E` - Redo

**Pro tips:**
```bash
# Open file at specific line number
nano +25 docker-compose.yml

# Create backup when saving
nano -B filename.txt
```

---

### touch - Create empty file

```bash
# Create new empty file
touch newfile.txt

# Create multiple files
touch file1.txt file2.txt file3.txt

# Update timestamp of existing file
touch existing-file.txt
```

---

### mkdir - Create directory

```bash
# Create single directory
mkdir new-folder

# Create nested directories
mkdir -p parent/child/grandchild

# Create with specific permissions
mkdir -m 755 new-folder

# Create multiple directories
mkdir folder1 folder2 folder3
```

---

### echo - Create/append to file

```bash
# Create file with content
echo "Hello World" > file.txt

# Append to existing file
echo "New line" >> file.txt

# Create multi-line file
cat > file.txt << 'EOF'
Line 1
Line 2
Line 3
EOF
```

> ⚠️ **WARNING:** `>` overwrites the file, `>>` appends to it.

---

## Copying, Moving & Deleting

### cp - Copy files

```bash
# Copy file
cp source.txt destination.txt

# Copy file to directory
cp file.txt /home/USERNAME/docker/

# Copy directory recursively
cp -r source-folder/ destination-folder/

# Copy with verbose output
cp -v file.txt backup/

# Copy preserving attributes (permissions, timestamps)
cp -p file.txt backup/

# Interactive (ask before overwrite)
cp -i file.txt destination.txt

# Create backup of existing file
cp -b file.txt destination/
```

**Real-world examples:**
```bash
# Backup docker-compose file
cp docker-compose.yml docker-compose.yml.backup

# Copy .env file before editing
cp .env .env.backup.$(date +%Y%m%d)

# Copy entire docker folder
cp -r ~/docker ~/docker-backup
```

---

### mv - Move/rename files

```bash
# Rename file
mv oldname.txt newname.txt

# Move file to directory
mv file.txt /home/USERNAME/docker/

# Move multiple files
mv file1.txt file2.txt /destination/

# Move with verbose output
mv -v file.txt /destination/

# Interactive (ask before overwrite)
mv -i file.txt /destination/

# Move directory
mv old-folder/ new-folder/
```

**Real-world examples:**
```bash
# Rename backup file with date
mv .env.backup .env.backup.$(date +%Y%m%d)

# Move log files to archive
mv *.log /var/log/archive/

# Rename docker folder
mv docker docker-old
```

---

### rm - Remove files (DANGEROUS!)

```bash
# Remove single file
rm file.txt

# Remove multiple files
rm file1.txt file2.txt file3.txt

# Interactive (ask for confirmation)
rm -i file.txt

# Remove directory and contents
rm -r folder/

# Force remove (no confirmation)
rm -f file.txt

# Remove directory forcefully
rm -rf folder/

# Verbose output
rm -v file.txt

# Remove with wildcard
rm *.log
```

> ⚠️ **DANGER:** `rm -rf` is EXTREMELY DANGEROUS! It deletes without confirmation and can't be undone!

**Safe removal practices:**
```bash
# ALWAYS use -i for interactive mode when starting
alias rm='rm -i'

# Better: Move to trash instead
mkdir -p ~/.trash
mv file.txt ~/.trash/

# Preview what will be deleted
ls *.log  # See what matches
rm *.log  # Then delete

# Never run these without thinking:
rm -rf /       # DESTROYS YOUR ENTIRE SYSTEM
rm -rf /*      # DESTROYS YOUR ENTIRE SYSTEM
rm -rf ~       # DELETES YOUR HOME DIRECTORY
```

> 💡 **TIP:** There's no "undo" or "recycle bin" in Linux by default. Deleted = gone forever!

---

## File Permissions

### Understanding Permissions

```
-rwxrw-r--
│││││││││└─ Other: read
││││││││└── Other: write (no)
│││││││└─── Other: execute (no)
││││││└──── Group: read
│││││└───── Group: write
││││└────── Group: execute (no)
│││└─────── Owner: read
││└──────── Owner: write
│└───────── Owner: execute
└────────── File type (- = file, d = directory, l = link)
```

**Permission numbers:**
- `r` (read) = 4
- `w` (write) = 2
- `x` (execute) = 1

**Common permission combinations:**
- `755` = `rwxr-xr-x` - Owner: full, Others: read+execute
- `644` = `rw-r--r--` - Owner: read+write, Others: read only
- `600` = `rw-------` - Owner: read+write, Others: nothing
- `777` = `rwxrwxrwx` - Everyone: full access (DANGEROUS!)

---

### chmod - Change permissions

```bash
# Numeric mode
chmod 755 script.sh
chmod 644 file.txt
chmod 600 .env

# Symbolic mode
chmod u+x script.sh        # Add execute for owner
chmod g-w file.txt         # Remove write for group
chmod o-r file.txt         # Remove read for others
chmod a+r file.txt         # Add read for all

# Recursive
chmod -R 755 folder/

# Common fixes
chmod 600 ~/.ssh/id_ed25519          # SSH private key
chmod 644 ~/.ssh/id_ed25519.pub      # SSH public key
chmod 700 ~/.ssh                      # SSH directory
chmod 600 ~/.ssh/authorized_keys      # Authorized keys
```

**Real-world examples:**
```bash
# Fix .env file permissions
chmod 600 ~/docker/.env

# Make script executable
chmod +x backup.sh

# Fix SSH permissions after copying
chmod 700 ~/.ssh
chmod 600 ~/.ssh/*
chmod 644 ~/.ssh/*.pub
```

---

### chown - Change ownership

```bash
# Change owner
sudo chown username file.txt

# Change owner and group
sudo chown username:groupname file.txt

# Recursive
sudo chown -R username:groupname folder/

# Change only group
sudo chown :groupname file.txt
```

> ⚠️ **NOTE:** You need `sudo` to change ownership (except for your own files).

---

## Searching & Finding

### find - Find files

```bash
# Find by name
find /path/to/search -name "filename.txt"

# Case-insensitive name search
find . -iname "*.log"

# Find directories only
find . -type d

# Find files only
find . -type f

# Find by size
find . -size +100M              # Larger than 100MB
find . -size -1M                # Smaller than 1MB

# Find by modification time
find . -mtime -7                # Modified in last 7 days
find . -mtime +30               # Modified more than 30 days ago

# Find and delete
find . -name "*.tmp" -delete

# Find and execute command
find . -name "*.log" -exec gzip {} \;
```

**Real-world examples:**
```bash
# Find large log files
find /var/log -type f -size +100M

# Find docker-compose files
find ~ -name "docker-compose.yml"

# Find files modified today
find ~/docker -type f -mtime -1

# Find and list .env files with details
find ~ -name ".env" -exec ls -lh {} \;

# Find empty directories
find . -type d -empty
```

---

### locate - Fast file search

```bash
# Search file database (faster than find)
locate docker-compose.yml

# Case-insensitive
locate -i dockerfile

# Update database first (run with sudo)
sudo updatedb
locate docker-compose.yml
```

> 💡 **TIP:** `locate` is faster than `find` but requires updating its database with `updatedb`.

---

### which - Find command location

```bash
# Find where a command is located
which docker
which nano
which python3

# Find all matches
which -a python
```

---

## System Information

### uname - System information

```bash
# Kernel name
uname

# All information
uname -a

# Kernel version
uname -r

# Machine hardware
uname -m

# Operating system
uname -o
```

---

### hostname - System name

```bash
# Show hostname
hostname

# Show IP address
hostname -I

# Show all addresses
hostname -A
```

---

### uptime - System uptime

```bash
# Show how long system has been running
uptime

# Pretty format
uptime -p
```

**Example output:**
```
15:24:32 up 42 days,  3:15,  2 users,  load average: 0.15, 0.20, 0.18
```

---

### free - Memory usage

```bash
# Show memory in MB
free -m

# Show memory in GB
free -g

# Human-readable format
free -h

# Show total line
free -ht
```

**Example output:**
```
              total        used        free      shared  buff/cache   available
Mem:           7.8G        2.1G        3.2G        156M        2.5G        5.3G
Swap:          2.0G          0B        2.0G
```

---

### df - Disk space

```bash
# Show disk usage
df

# Human-readable format
df -h

# Show specific filesystem
df -h /

# Show inodes
df -i
```

**Example output:**
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/vda1        50G   15G   33G  31% /
```

---

## Process Management

### ps - View processes

```bash
# Current user's processes
ps

# All processes
ps aux

# All processes in tree format
ps auxf

# Process by name
ps aux | grep docker

# Detailed process info
ps -ef

# Sort by memory usage
ps aux --sort=-%mem | head

# Sort by CPU usage
ps aux --sort=-%cpu | head
```

---

### top - Interactive process viewer

```bash
# Launch top
top

# Sort by memory (press M)
# Sort by CPU (press P)
# Kill process (press k, then enter PID)
# Quit (press q)
```

**Better alternative: htop**
```bash
# Install htop (more user-friendly)
sudo apt install htop

# Run htop
htop
```

---

### kill - Stop processes

```bash
# Replace PID by the PID number given by the ps command seen above.
# Graceful stop (SIGTERM)
kill PID

# Force kill (SIGKILL)
kill -9 PID

# Kill by name
pkill process-name

# Kill all processes matching pattern
killall process-name

# Interactive kill
kill -15 PID  # Ask nicely
sleep 5
kill -9 PID   # Force if still running
```

**Find PID of a process:**
```bash
# Method 1: Using ps
ps aux | grep docker

# Method 2: Using pgrep
pgrep docker

# Method 3: Using pidof
pidof dockerd
```

---

### systemctl - Manage services

```bash
# Check service status
sudo systemctl status docker

# Start service
sudo systemctl start docker

# Stop service
sudo systemctl stop docker

# Restart service
sudo systemctl restart docker

# Enable service (start on boot)
sudo systemctl enable docker

# Disable service (don't start on boot)
sudo systemctl disable docker

# Reload service configuration
sudo systemctl reload docker

# View service logs
sudo journalctl -u docker

# Follow service logs in real-time
sudo journalctl -fu docker
```

**Common services:**
- `docker` - Docker daemon
- `sshd` - SSH server
- `fail2ban` - Fail2ban service
- `ufw` - Firewall

---

## Disk Usage

### du - Directory size

```bash
# Size of current directory
du -sh .

# Size of specific directory
du -sh ~/docker

# Size of all subdirectories
du -h

# Show only totals
du -sh */

# Sort by size
du -h | sort -h

# Top 10 largest directories
du -h | sort -hr | head -10

# Include hidden files
du -sh .[^.]* *
```

**Real-world examples:**
```bash
# Check docker folder size
du -sh ~/docker

# Find largest folders in home directory
du -h ~ | sort -hr | head -20

# Check log folder size
sudo du -sh /var/log

# Check Docker volumes size
sudo du -sh /var/lib/docker/volumes/*
```

---

### ncdu - Interactive disk usage (better than du)

```bash
# Install ncdu
sudo apt install ncdu

# Analyze current directory
ncdu

# Analyze specific path
ncdu /var/log

# Inside ncdu:
# Arrow keys to navigate
# Enter to explore directory
# d to delete
# q to quit
```

> 💡 **TIP:** `ncdu` is much more user-friendly than `du` for finding large files!

---

## User & Group Management

### whoami - Current user

```bash
# Show current username
whoami

# Show current user ID
id

# Show all user info
id -a
```

---

### users - Logged in users

```bash
# List logged in users
users

# Detailed list
who

# Who with more details
w
```

---

### sudo - Run as superuser

```bash
# Run single command as root
sudo command

# Switch to root user
sudo -i
sudo su

# Run command as another user
sudo -u USERNAME command

# Check sudo access
sudo -v

# List sudo privileges
sudo -l
```

> ⚠️ **WARNING:** Doing your maintenance as a SUDO user is very convenient but is considered as a very bad practice.

---

## Package Management

### apt - Package manager

```bash
# Update package list
sudo apt update

# Upgrade all packages
sudo apt upgrade

# Upgrade with automatic "yes"
sudo apt upgrade -y

# Install package
sudo apt install package-name

# Remove package
sudo apt remove package-name

# Remove package and config files
sudo apt purge package-name

# Remove unused packages
sudo apt autoremove

# Search for package
apt search package-name

# Show package info
apt show package-name

# List installed packages
apt list --installed

# List upgradable packages
apt list --upgradable
```

**Real-world examples:**
```bash
# Update system
sudo apt update && sudo apt upgrade -y

# Install useful tools
sudo apt install htop ncdu

# Remove old packages
sudo apt autoremove

# Check what can be updated
apt list --upgradable
```

---

## Network Commands

### ping - Test connectivity

```bash
# Ping a host
ping google.com

# Ping 4 times only
ping -c 4 google.com

# Ping with timestamp
ping -D google.com

# Stop pinging: Ctrl + C
```

---

### curl - HTTP requests

Very useful in a n8n environment as you can use it to simulate and try API calls.  
```bash
# Download file
curl -O https://example.com/file.txt

# Save to specific name
curl -o myfile.txt https://example.com/file.txt

# Show only headers
curl -I https://example.com

# Follow redirects
curl -L https://example.com

# POST request
curl -X POST -d "data=value" https://api.example.com

# With authentication
curl -u USERNAME:password https://example.com

# JSON POST
curl -X POST -H "Content-Type: application/json" -d '{"key":"value"}' https://api.example.com
```

---

### wget - Download files

```bash
# Download file
wget https://example.com/file.txt

# Save to specific name
wget -O myfile.txt https://example.com/file.txt

# Continue interrupted download
wget -c https://example.com/largefile.zip

# Download in background
wget -b https://example.com/file.txt

# Download multiple files
wget -i urls.txt
```

---

### ip - Network configuration

```bash
# Show all interfaces
ip addr

# Show specific interface
ip addr show eth0

# Show routing table
ip route

# Show network statistics
ip -s link
```

---

### ss - Socket statistics

```bash
# Show all connections
ss -a

# Show listening ports
ss -l

# Show TCP connections
ss -t

# Show listening TCP ports
ss -tl

# Show UDP connections
ss -u

# Show process using ports
ss -tulpn

# Show specific port
ss -tunlp | grep :80
```

---

### netstat - Network statistics (older alternative to ss)

```bash
# Show all connections
netstat -a

# Show listening ports
netstat -l

# Show TCP connections
netstat -t

# Show with process info
netstat -tulpn

# Show specific port
netstat -tunlp | grep :80
```

---

## Helpful Tips & Tricks

### Command History

```bash
# Show command history
history

# Show last 20 commands
history 20

# Search history
history | grep docker

# Re-run command by number
!123

# Re-run last command
!!

# Re-run last command with sudo
sudo !!

# Search history interactively
Ctrl + R  # Then type to search

# Clear history
history -c
```

---

### Keyboard Shortcuts

**Navigation:**
- `Ctrl + A` - Beginning of line
- `Ctrl + E` - End of line
- `Ctrl + Left/Right` - Move word by word
- `Alt + B` - Back one word
- `Alt + F` - Forward one word

**Editing:**
- `Ctrl + K` - Cut from cursor to end
- `Ctrl + U` - Cut from cursor to beginning
- `Ctrl + W` - Cut word before cursor
- `Ctrl + Y` - Paste cut text
- `Ctrl + L` - Clear screen (same as `clear`)

**Process control:**
- `Ctrl + C` - Stop current command
- `Ctrl + Z` - Suspend current command
- `Ctrl + D` - Exit shell or EOF

---

### Pipe & Redirect

**Pipe (|)** - Send output to another command:
```bash
# Count lines
cat file.txt | wc -l

# Search in output
docker ps | grep n8n

# Sort and show unique
cat file.txt | sort | uniq

# Chain multiple commands
ps aux | grep docker | grep -v grep | awk '{print $2}'
```

**Redirect (>)** - Save output to file:
```bash
# Overwrite file
ls > file-list.txt

# Append to file
ls >> file-list.txt

# Redirect errors
command 2> error.log

# Redirect both output and errors
command > output.log 2>&1

# Discard output
command > /dev/null

# Discard errors
command 2> /dev/null

# Discard everything
command > /dev/null 2>&1
```

---

### Command Chaining

```bash
# Run second command only if first succeeds (&&)
sudo apt update && sudo apt upgrade

# Run second command only if first fails (||)
command1 || command2

# Run both regardless (;)
command1 ; command2

# Run in background (&)
long-running-command &
```

---

### Aliases

**Create shortcuts for common commands:**

```bash
# Temporary alias (this session only)
alias ll='ls -lah'
alias update='sudo apt update && sudo apt upgrade'

# Permanent alias (add to ~/.bashrc)
echo "alias ll='ls -lah'" >> ~/.bashrc
source ~/.bashrc

# View all aliases
alias

# Remove alias
unalias ll
```

**Useful aliases:**
```bash
alias ll='ls -lah'
alias la='ls -A'
alias l='ls -CF'
alias ..='cd ..'
alias ...='cd ../..'
alias grep='grep --color=auto'
alias update='sudo apt update && sudo apt upgrade -y'
alias dps='docker ps'
alias dlog='docker compose logs -f'

# To add at the end of your .bashrc
# the command lsc will then list the content of a folder with all the directories first, then the hidden files, then the normal files
lsc() {
    (ls -lAhd */ 2>/dev/null; ls -lAh | grep "^-" | grep "^\-.*\s\.\|^-.*\s\."; ls -lAh | grep "^-" | grep -v "^\-.*\s\.") | awk 'NR==1 || !/^total/'
}

```

---

### Tab Completion

**Make your life easier:**

```bash
# Type partial command/path and press Tab
cd ~/doc[TAB]  # Completes to ~/docker/

# Press Tab twice to see all options
cd ~/d[TAB][TAB]  # Shows all directories starting with 'd'

# Complete command options
docker [TAB][TAB]  # Shows all docker commands
```

---

### Wildcards

```bash
# * matches any characters
ls *.txt        # All .txt files
rm backup*      # All files starting with 'backup'

# ? matches single character
ls file?.txt    # file1.txt, file2.txt, etc.

# [] matches character set
ls file[12].txt # file1.txt and file2.txt
ls file[a-z].txt # filea.txt through filez.txt

# {} matches patterns
cp file.{txt,log} backup/  # Copies file.txt and file.log
```

---

### Command Substitution

```bash
# Use output of command as argument
echo "Today is $(date)"

# Older syntax (backticks)
echo "Current directory: `pwd`"

# Examples
cp file.txt file.txt.backup.$(date +%Y%m%d)
tar -czf backup-$(date +%Y%m%d).tar.gz ~/docker/
```

---

### Environment Variables

```bash
# Show environment variable
echo $HOME
echo $USER
echo $PATH

# Show all environment variables
env
printenv

# Set temporary variable (this session)
export MY_VAR="value"

# Set permanent variable (add to ~/.bashrc)
echo 'export MY_VAR="value"' >> ~/.bashrc
source ~/.bashrc

# Common variables
$HOME     # Your home directory
$USER     # Your USERNAME
$PATH     # Command search paths
$PWD      # Current directory
$SHELL    # Your shell
```

---

## 📌 Quick Command Reference

```bash
# Navigation
pwd                    # Where am I?
ls -lah               # List files
cd ~/docker           # Change directory

# File operations
cat file.txt          # View file
nano file.txt         # Edit file
cp source dest        # Copy
mv old new            # Move/rename
rm file.txt           # Delete (CAREFUL!)

# Searching
find . -name "*.txt"  # Find files
grep "text" file      # Search in file

# System info
df -h                 # Disk space
free -h               # Memory
ps aux                # Processes
top                   # Live processes

# Permissions
chmod 644 file        # Change permissions
chown user:group file # Change owner

# Package management
sudo apt update       # Update package list
sudo apt upgrade      # Upgrade packages
sudo apt install pkg  # Install package

# Network
ping google.com       # Test connectivity
curl example.com      # HTTP request
ss -tulpn            # Show ports
```

---

## 🎓 Practice Exercises

Try these to build confidence:

1. **Navigate your system:**
   ```bash
   cd ~
   pwd
   ls -lah
   cd docker
   pwd
   cd ..
   ```

2. **View and search files:**
   ```bash
   cat docker-compose.yml
   grep "n8n" docker-compose.yml
   tail -f /var/log/syslog
   ```

3. **Check system resources:**
   ```bash
   df -h
   free -h
   ps aux | grep docker
   ```

4. **Create and manage files:**
   ```bash
   touch test.txt
   nano test.txt  # Add some text
   cat test.txt
   cp test.txt test-backup.txt
   rm test.txt
   ```

---

**Next Guide:** [05 - Docker Management Cheatsheet](05-docker-cheatsheet.md)  
**Previous Guide:** [03 - Security & Access Management](03-security-access.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: [07/01/2026]*
