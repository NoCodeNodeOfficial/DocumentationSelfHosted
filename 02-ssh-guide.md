# 02 - SSH Connection Guide

**Difficulty Level:** 🟢 Beginner  
**Risk Level:** 🟢 Safe

Learn how to generate SSH keys and connect securely to your server from any operating system.

---

## 📋 Table of Contents

1. [Understanding SSH Keys](#understanding-ssh-keys)
2. [Generating SSH Keys](#generating-ssh-keys)
   - [Windows](#windows)
   - [macOS](#macos)
   - [Linux](#linux)
3. [Connecting to Your Server](#connecting-to-your-server)
   - [Terminal (Mac/Linux)](#terminal-maclinux)
   - [PowerShell (Windows)](#powershell-windows)
   - [PuTTY (Windows)](#putty-windows)
   - [MobaXterm (Windows)](#mobaxterm-windows)
4. [Troubleshooting Connection Issues](#troubleshooting-connection-issues)

---

## Understanding SSH Keys

Your server uses **SSH key authentication** instead of just passwords for enhanced security.

**What you need to know:**
- Your server is configured on **port 1077** (not the default 22)
- You need an **SSH private key** to connect
- Password authentication is disabled by default (can be re-enabled if needed)
- The recommended key type is **Ed25519** (modern, secure, fast)

> 💡 **TIP:** Think of SSH keys like a physical key to your house. The private key stays with you (never share it!), and the public key is like a lock installed on the server.

---

## Generating SSH Keys

Choose your operating system below:

### Windows

#### Option 1: Using PowerShell (Built-in, Recommended)

**Windows 10/11 comes with SSH built-in!**

1. **Open PowerShell**
   - Press `Win + X`
   - Select "Windows PowerShell" or "Terminal"

2. **Generate the key:**
```powershell
# Create .ssh folder if it doesn't exist
mkdir ~/.ssh -ErrorAction SilentlyContinue

# Generate Ed25519 key
ssh-keygen -t ed25519 -C "your_email@example.com"
```

3. **Follow the prompts:**
   - **File location:** Press Enter to use default (`C:\Users\YourName\.ssh\id_ed25519`)
   - **Passphrase:** Optional but recommended for extra security
   - **Confirm passphrase:** Enter same passphrase again

4. **Locate your keys:**
```powershell
# View your public key (this is what you share with servers)
cat ~/.ssh/id_ed25519.pub

# Your keys are stored at:
# Private key: C:\Users\YourName\.ssh\id_ed25519
# Public key:  C:\Users\YourName\.ssh\id_ed25519.pub
```

> ⚠️ **WARNING:** Never share your private key (`id_ed25519`)! Only share the public key (`.pub` file).

---

#### Option 2: Using PuTTY Key Generator (GUI)

**If you prefer a graphical interface:**

1. **Download PuTTYgen:** 
   - Visit: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
   - Download `puttygen.exe` for your platform

2. **Generate the key:**
   - Open PuTTYgen
   - Select "EdDSA" and "Ed25519" curve
   - Click "Generate"
   - Move your mouse randomly in the blank area (generates randomness)

3. **Save your keys:**
   - **Key comment:** Enter your email or identifier
   - **Key passphrase:** Optional but recommended
   - Click "Save public key" → Save as `id_ed25519.pub`
   - Click "Save private key" → Save as `id_ed25519.ppk`
   - **Important:** Also copy the text from "Public key for pasting" box

> 💡 **TIP:** PuTTY uses `.ppk` format. OpenSSH uses different format. Keep both if switching between tools!

---

#### Option 3: Using MobaXterm (All-in-One Tool)

**MobaXterm includes SSH client + key generation:**
> 💡 **TIP:** To be honest, MobaXterm is by far, one of the best terminal for Windows users. 

1. **Download MobaXterm:**
   - Visit: https://mobaxterm.mobatek.net/download.html
   - Download "Home Edition" (Free)
   - Install and open MobaXterm

2. **Generate SSH key:**
   - Click "Tools" menu → "MobaKeyGen (SSH key generator)"
   - Select "EdDSA" with "Ed25519" curve
   - Click "Generate"
   - Move mouse for randomness
   - Add passphrase (optional but recommended)
   - Click "Save private key" → Save as `id_ed25519`
   - Click "Save public key" → Save as `id_ed25519.pub`

---

### macOS

**macOS has SSH built into Terminal:**

1. **Open Terminal:**
   - Press `Cmd + Space`
   - Type "Terminal"
   - Press Enter

2. **Generate the key:**
```bash
# Create .ssh folder if it doesn't exist
mkdir -p ~/.ssh

# Generate Ed25519 key
ssh-keygen -t ed25519 -C "your_email@example.com"
```

3. **Follow the prompts:**
   - **File location:** Press Enter for default (`/Users/yourname/.ssh/id_ed25519`)
   - **Passphrase:** Optional but recommended
   - **Confirm passphrase:** Enter same passphrase again

4. **View your public key:**
```bash
# Display your public key
cat ~/.ssh/id_ed25519.pub

# Copy to clipboard (optional)
pbcopy < ~/.ssh/id_ed25519.pub
```

> 💡 **TIP:** Use `pbcopy` to copy your public key directly to clipboard without selecting text!

---

### Linux

**All Linux distributions have SSH built-in:**

1. **Open Terminal:**
   - Usually `Ctrl + Alt + T`
   - Or search for "Terminal" in your applications

2. **Generate the key:**
```bash
# Create .ssh folder with correct permissions
mkdir -p ~/.ssh
chmod 700 ~/.ssh

# Generate Ed25519 key
ssh-keygen -t ed25519 -C "your_email@example.com"
```

3. **Follow the prompts:**
   - **File location:** Press Enter for default (`/home/yourname/.ssh/id_ed25519`)
   - **Passphrase:** Optional but recommended
   - **Confirm passphrase:** Enter same passphrase again

4. **View your public key:**
```bash
# Display your public key
cat ~/.ssh/id_ed25519.pub
```

---

## Connecting to Your Server

Now that you have SSH keys, here's how to connect:

### Terminal (Mac/Linux)

**Basic connection:**
```bash
ssh -p 1077 USERNAME@YOUR_SERVER_IP
```

**Using a specific key file:**
```bash
# This is where you have to use the PRIVATE file
ssh -i ~/.ssh/id_ed25519 -p 1077 USERNAME@YOUR_SERVER_IP
```

**First-time connection:**
- You'll see a message about authenticity of host
- Type `yes` and press Enter
- This is normal for first-time connections

**Example session:**
```bash
$ ssh -p 1077 -i ~/.ssh/id_ed25519 root@192.168.1.100
The authenticity of host '[192.168.1.100]:1077' can't be established.
ED25519 key fingerprint is SHA256:xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '[192.168.1.100]:1077' (ED25519) to the list of known hosts.
Welcome to Ubuntu 24.04 LTS

root@server:~$
```

> 💡 **TIP:** Create an SSH config file to simplify connection:

```bash
# Edit SSH config
nano ~/.ssh/config

# Add this content:
Host myserver
    HostName YOUR_SERVER_IP
    Port 1077
    User USERNAME
    IdentityFile ~/.ssh/id_ed25519

# Now you can connect with just:
ssh myserver
```

---

### PowerShell (Windows)

**Basic connection:**
```powershell
ssh -p 1077 USERNAME@YOUR_SERVER_IP
```

**Using a specific key file:**
```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 -p 1077 USERNAME@YOUR_SERVER_IP
```

**Create an SSH config file (Windows):**
```powershell
# Create config file
notepad $env:USERPROFILE\.ssh\config

# Add this content:
Host myserver
    HostName YOUR_SERVER_IP
    Port 1077
    User USERNAME
    IdentityFile C:\Users\YourName\.ssh\id_ed25519

# Save and close, then connect with:
ssh myserver
```

> 💡 **TIP:** If you get "command not found", you may need to enable OpenSSH Client in Windows Features.

---

### PuTTY (Windows)

**PuTTY is a popular Windows SSH client:**

1. **Download PuTTY:**
   - Visit: https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html
   - Download `putty.exe`

2. **Convert your key (skip this step if your key is already in .pkk format):**
   - Open PuTTYgen
   - Click "Load"
   - Select your private key (`id_ed25519`)
   - Click "Save private key"
   - Save as `id_ed25519.ppk`

3. **Configure connection:**
   - Open PuTTY
   - **Host Name:** `YOUR_SERVER_IP`
   - **Port:** `1077`
   - **Connection type:** SSH
   
4. **Add your private key:**
   - In left menu: Connection → SSH → Auth → Credentials
   - Click "Browse" next to "Private key file"
   - Select your `.ppk` file

5. **Save session (optional):**
   - Go back to "Session" category
   - Enter name in "Saved Sessions"
   - Click "Save"
   - Next time, just double-click the saved session

6. **Connect:**
   - Click "Open"
   - Enter your username when prompted
   - Enter key passphrase if you set one

---

### MobaXterm (Windows)

**MobaXterm is an all-in-one terminal with built-in SSH:**

1. **Start a new session:**
   - Click "Session" button
   - Select "SSH"

2. **Configure connection:**
   - **Remote host:** `YOUR_SERVER_IP`
   - **Specify username:** Check box and enter `USERNAME`
   - **Port:** `1077`

3. **Advanced SSH settings:**
   - Click "Advanced SSH settings" tab
   - Check "Use private key"
   - Browse to your private key file (`id_ed25519` or `id_ed25519.ppk`)

4. **Save and connect:**
   - Click "OK"
   - MobaXterm will save this session
   - Double-click saved session to reconnect anytime

> 💡 **TIP:** MobaXterm shows a file browser sidebar - makes it easy to transfer files!

---

## Troubleshooting Connection Issues

### Common Issues and Solutions

#### ❌ "Connection refused" or "Connection timed out"

**Possible causes:**
- Wrong port number (remember: port 1077, not 22)
- Server firewall blocking connection
- Wrong IP address

**Solutions:**
```bash
# Verify you're using correct port
ssh -p 1077 USERNAME@YOUR_SERVER_IP

# Test if port is open
telnet YOUR_SERVER_IP 1077
# Or on Windows PowerShell:
Test-NetConnection -ComputerName YOUR_SERVER_IP -Port 1077
```

---

#### ❌ "Permission denied (publickey)"

**Possible causes:**
- Wrong username
- SSH key not added to server
- Using wrong private key

**Solutions:**
```bash
# Verify username is correct
ssh -v -p 1077 USERNAME@YOUR_SERVER_IP
# The -v flag shows verbose output for debugging

# Try specifying key explicitly
ssh -i ~/.ssh/id_ed25519 -p 1077 USERNAME@YOUR_SERVER_IP

# Check key permissions (Unix/Mac/Linux only)
chmod 600 ~/.ssh/id_ed25519
chmod 644 ~/.ssh/id_ed25519.pub
```

---

#### ❌ "WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED"

**This appears if:**
- Server was reinstalled
- IP address was reused for different server
- Potential security issue (rare)

**Solution:**
```bash
# Remove old host key
ssh-keygen -R [YOUR_SERVER_IP]:1077

# On Windows PowerShell:
ssh-keygen -R "[YOUR_SERVER_IP]:1077"

# Then try connecting again
```

> ⚠️ **WARNING:** Only do this if you're certain the server change is legitimate!

---

#### ❌ "Could not resolve hostname"

**Possible causes:**
- Typo in hostname/IP address
- DNS not configured yet
- Network connectivity issue

**Solutions:**
```bash
# Use IP address instead of domain name
ssh -p 1077 USERNAME@192.168.1.100

# Test DNS resolution
ping yourdomain.website
nslookup yourdomain.website
```

---

#### ❌ Key file not being recognized (PuTTY)

**Issue:** PuTTY requires `.ppk` format, not OpenSSH format

**Solution:**
1. Open PuTTYgen
2. Click "Load" → Select your OpenSSH key
3. Click "Save private key" → Save as `.ppk`
4. Use the `.ppk` file in PuTTY

---

### Getting Verbose Output for Debugging

**See detailed connection information:**

```bash
# Mac/Linux/PowerShell
ssh -vvv -p 1077 USERNAME@YOUR_SERVER_IP

# This shows:
# - Which keys are being tried
# - Authentication methods attempted
# - Where the connection is failing
```

---

### Testing Your Connection

**Before trying to SSH, test basic connectivity:**

```bash
# Test if server is reachable
ping YOUR_SERVER_IP

# Test if SSH port is open (Mac/Linux)
nc -zv YOUR_SERVER_IP 1077

# Test if SSH port is open (Windows PowerShell)
Test-NetConnection -ComputerName YOUR_SERVER_IP -Port 1077

# Try connecting with maximum verbosity
ssh -vvv -p 1077 USERNAME@YOUR_SERVER_IP
```

---

## 🎓 What You've Learned

You should now be able to:
- ✅ Generate SSH keys on any operating system
- ✅ Connect to your server using various SSH clients
- ✅ Troubleshoot common connection issues
- ✅ Use both terminal and GUI SSH clients

---

## 📌 Quick Reference

**Connection command (Terminal/PowerShell):**
```bash
ssh -p 1077 USERNAME@YOUR_SERVER_IP
```

**Your SSH key locations:**
- **Windows:** `C:\Users\YourName\.ssh\id_ed25519`
- **Mac:** `/Users/yourname/.ssh/id_ed25519`
- **Linux:** `/home/yourname/.ssh/id_ed25519`

**Important reminders:**
- Always use **port 1077**
- Never share your private key
- Only share the `.pub` (public) key file
- Keep your private key passphrase secure

---

**Next Guide:** [03 - Security & Access Management](03-security-access.md)  
**Previous Guide:** [01 - Getting Started](01-getting-started.md)  
**Back to:** [Documentation Home](README.md)

---

*Last Updated: 29/12/2025*