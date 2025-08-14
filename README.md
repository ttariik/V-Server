# V-Server Setup Documentation

## Table of Contents
1. [Overview](#overview)
2. [Server Information](#server-information)
3. [Prerequisites](#prerequisites)
4. [Setup Steps](#setup-steps)
   - [SSH Key Configuration](#ssh-key-configuration)
   - [Security Configuration](#security-configuration)
   - [Web Server Installation](#web-server-installation)
   - [Git Configuration](#git-configuration)
   - [GitHub Integration](#github-integration)
5. [Testing](#testing)
6. [Troubleshooting](#troubleshooting)

## Overview

This repository documents the complete setup process for a V-Server (Virtual Private Server) following security best practices and required configurations.

## Server Information

- **IP Address**: <your_ip_adress>
- **Operating System**: Ubuntu 22.04.5 LTS
- **Username**: <your_user_name>
- **Web Server**: NGINX 1.18.0 (Port 8081)
- **Website URL**: http://<your_ip_adress>:8081

## Prerequisites

- Local computer with SSH client
- Access to V-Server with initial password authentication
- Git installed locally and on server

## Setup Steps

### SSH Key Configuration

#### 1. Generate SSH Key Pair (Local Computer)
```bash
# Generate new SSH key pair on local computer
ssh-keygen -t rsa -b 4096 -C "<your_user_name>@v-server-setup" -f ~/.ssh/v_server_key
```

**Key Type Decision: RSA 4096-bit vs ED25519**

I chose RSA 4096-bit over the more modern ED25519 for the following reasons:
- **Maximum Compatibility**: RSA works on all systems, including legacy infrastructure
- **Explicit Security**: The 4096-bit key length clearly demonstrates strong encryption
- **Enterprise Standard**: Many organizations still prefer RSA for server access
- **Educational Value**: Shows understanding of key strength parameters

*Note: While ED25519 is more modern and efficient, RSA 4096-bit provides equivalent security with broader compatibility.*

#### 2. Copy Public Key to Server
```bash
# Copy public key to server
ssh-copy-id -i ~/.ssh/v_server_key.pub <your_user_name>@<your_ip_adress>
```

#### 3. Test SSH Key Authentication
```bash
# Test SSH key login
ssh -i ~/.ssh/v_server_key <your_user_name>@<your_ip_adress>
```

### Security Configuration

#### 4. Disable Password Authentication

**🔐 WHY DISABLE PASSWORD AUTHENTICATION?**

Disabling password authentication is a **critical security measure** for the following reasons:

- **🛡️ Protection against Brute-Force Attacks**: Attackers can no longer endlessly try password combinations
- **🔑 Stronger Authentication**: SSH keys are mathematically much more secure than passwords
- **⚡ Better Performance**: No CPU load from failed password attempts
- **📊 Less Log Spam**: No constant failed login attempts in the logs
- **🎯 Compliance**: Many security standards require key-only authentication

**⚠️ IMPORTANT: Ensure SSH Key Authentication works BEFORE disabling Password Authentication!**

**🔧 STEP-BY-STEP CONFIGURATION:**

```bash
# 1. Create SSH configuration backup (CRITICAL!)
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
echo "✅ Backup created at /etc/ssh/sshd_config.backup"

# 2. Disable password authentication
# Handles both commented (#) and active lines
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
echo "✅ Password Authentication disabled"

# 3. Explicitly enable public key authentication
# Ensures SSH keys continue to work
sudo sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/PubkeyAuthentication no/PubkeyAuthentication yes/' /etc/ssh/sshd_config
echo "✅ Public Key Authentication enabled"

# 4. Verify configuration
echo "🔍 Current SSH configuration:"
sudo grep -E '(PasswordAuthentication|PubkeyAuthentication)' /etc/ssh/sshd_config

# 5. Test SSH configuration for syntax errors
sudo sshd -t
if [ $? -eq 0 ]; then
    echo "✅ SSH configuration is syntactically correct"
else
    echo "❌ SSH configuration has errors! Restore backup!"
    exit 1
fi

# 6. Restart SSH service
sudo systemctl restart ssh
echo "✅ SSH service restarted"

# 7. Check service status
sudo systemctl status ssh --no-pager
```

**📋 EXPECTED CONFIGURATION:**

After configuration, the following values should be in `/etc/ssh/sshd_config`:

```
PasswordAuthentication no       # ❌ Password login disabled
PubkeyAuthentication yes        # ✅ SSH key login enabled
```

**🧪 IMMEDIATE TESTING:**

```bash
# Test 1: SSH Key Login (should work)
ssh -i ~/.ssh/v_server_key <your_user_name>@<your_ip_adress> "echo 'SSH Key Login successful!'"

# Test 2: Password Login (should fail)
# Try without key - should return "Permission denied"
ssh -o PreferredAuthentications=password <your_user_name>@<your_ip_adress>
# Expected response: "Permission denied (publickey)."
```

**🚨 TROUBLESHOOTING:**

If login no longer works:

```bash
# 1. Restore backup
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart ssh

# 2. Check SSH key permissions
ls -la ~/.ssh/v_server_key      # Should be 600 (-rw-------)
chmod 600 ~/.ssh/v_server_key   # Correct if necessary

# 3. Check authorized keys on server
ssh -i ~/.ssh/v_server_key <your_user_name>@<your_ip_adress> "ls -la ~/.ssh/authorized_keys"
```

**🎯 SECURITY RESULT:**

After this configuration, your server is:
- ✅ **Immune to password brute-force attacks**
- ✅ **Only accessible via SSH key**
- ✅ **Compliant with modern security standards**
- ✅ **Significantly more secure than default configuration**

### Web Server Installation

#### 5. Install NGINX
```bash
sudo apt update
sudo apt install nginx -y
```

#### 6. Configure Custom HTML Page
Created a modern, responsive HTML page with gradient background and professional styling:
```bash
# Create custom HTML page (content shown in custom_index.html)
sudo mv custom_index.html /var/www/html/index.html

# Test configuration and reload
sudo nginx -t
sudo systemctl reload nginx
```

#### 6a. Configure Custom Port (8081)
For enhanced security and service isolation, NGINX was configured to run on port 8081:
```bash
# Backup original configuration
sudo cp /etc/nginx/sites-available/default /etc/nginx/sites-available/default.backup

# Change port from 80 to 8081
sudo sed -i 's/listen 80 default_server;/listen 8081 default_server;/' /etc/nginx/sites-available/default
sudo sed -i 's/listen \[::\]:80 default_server;/listen [::]:8081 default_server;/' /etc/nginx/sites-available/default

# Test and restart NGINX
sudo nginx -t
sudo systemctl restart nginx
```

**Port Selection Rationale:**
- **Security through Obscurity**: Non-standard ports reduce automated scanning
- **Service Isolation**: Separates web service from standard HTTP traffic
- **Enterprise Practice**: Common pattern for development/testing environments
- **Demonstration**: Shows advanced NGINX configuration skills

### Git Configuration

#### 7. Configure Git User Information
```bash
git config --global user.name "First name Last name"
git config --global user.email "<your_user_name>@example.com"
```

### GitHub Integration

#### 8. Generate SSH Key for GitHub (Server)
```bash
ssh-keygen -t rsa -b 4096 -C "<your_user_name>@v-server-github" -f ~/.ssh/github_key -N ""
cat ~/.ssh/github_key.pub
```

**GitHub Public Key:**
```
ssh-rsa AAAAB3Nz
```

Add this public key to your GitHub account under Settings > SSH and GPG keys.

## Testing

### Security Tests
- [x] SSH key authentication works ✓
  ```bash
  ssh -i ~/.ssh/v_server_key <your_user_name>@<your_ip_adress>
  ```
- [x] Password authentication is disabled ✓
  ```bash
  ssh -o PubkeyAuthentication=no -o PasswordAuthentication=yes <your_user_name>@<your_ip_adress>
  # Result: Permission denied (publickey)
  ```
- [x] Server is accessible via SSH key only ✓

### Web Server Tests
- [x] NGINX is running and accessible ✓
  ```bash
  systemctl status nginx
  # Result: Active: active (running)
  ```
- [x] Custom HTML page is displayed ✓
  ```bash
  curl -I http://<your_ip_adress>:8081
  # Result: HTTP/1.1 200 OK (Port 8081)
  ```
- [x] Configuration is valid ✓
  ```bash
  nginx -t
  # Result: configuration file test is successful
  ```

### Git Tests
- [x] Git is configured with correct user information ✓
  ```bash
  git config --global --list | grep user
  # Result: user.name=First name Last name, user.email=<your_user_name>@example.com
  ```
- [x] GitHub SSH key generated and ready ✓

### Final Validation Results
- **Server IP**: <your_ip_adress> ✅ Accessible
- **Website URL**: http://<your_ip_adress>:8081 ✅ Live and responsive
- **SSH Security**: Password login disabled, key-only access ✅
- **Web Server**: NGINX running on port 8081 with custom HTML page ✅
- **Port Configuration**: Custom port 8081 for enhanced security ✅
- **Git Setup**: Configured with user credentials ✅
- **GitHub Integration**: SSH key generated for repository access ✅

## Troubleshooting

Common issues and solutions will be documented here as they arise.

---

**Important**: This documentation excludes sensitive information such as private keys and passwords for security reasons.