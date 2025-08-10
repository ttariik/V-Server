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

- **IP Address**: 91.99.193.112
- **Operating System**: Ubuntu 22.04.5 LTS
- **Username**: tsabanovic

## Prerequisites

- Local computer with SSH client
- Access to V-Server with initial password authentication
- Git installed locally and on server

## Setup Steps

### SSH Key Configuration

#### 1. Generate SSH Key Pair (Local Computer)
```bash
# Generate new SSH key pair on local computer
ssh-keygen -t rsa -b 4096 -C "tsabanovic@v-server-setup" -f ~/.ssh/v_server_key
```

#### 2. Copy Public Key to Server
```bash
# Copy public key to server
ssh-copy-id -i ~/.ssh/v_server_key.pub tsabanovic@91.99.193.112
```

#### 3. Test SSH Key Authentication
```bash
# Test SSH key login
ssh -i ~/.ssh/v_server_key tsabanovic@91.99.193.112
```

### Security Configuration

#### 4. Disable Password Authentication
Backup and edit SSH configuration on the server:
```bash
# Create backup
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup

# Disable password authentication
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config

# Ensure public key authentication is enabled
sudo sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/PubkeyAuthentication no/PubkeyAuthentication yes/' /etc/ssh/sshd_config

# Verify configuration
sudo grep -E '(PasswordAuthentication|PubkeyAuthentication)' /etc/ssh/sshd_config

# Restart SSH service
sudo systemctl restart ssh
```

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

### Git Configuration

#### 7. Configure Git User Information
```bash
git config --global user.name "Tarik Sabanovic"
git config --global user.email "tsabanovic@example.com"
```

### GitHub Integration

#### 8. Generate SSH Key for GitHub (Server)
```bash
ssh-keygen -t rsa -b 4096 -C "tsabanovic@v-server-github" -f ~/.ssh/github_key -N ""
cat ~/.ssh/github_key.pub
```

**GitHub Public Key:**
```
ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQDJJDp9TKd6qAju8twV8Y+D31089m0GfIpRtkTzP3FVn43cxX05/NPoce3cBtn7th1Wq8TKrFBQ9LOaUYBmBSuFivn3IE560Pp20OJV1bPMpDHci7iRoz/ShhpZKG8v3urakz7UjnizvExExXwg5PSmJh3Di0wWheHndmSROj0liTpHI6S7uDtW8gkDDTXfFkzjHbZy+xV5OP5Lce8TnRQ5/sfaHwNK08Yq48m6pH7HiAn7N251av67qPWA6s0usrCbPfiPigLKk/FM3Px83krcLRlm24/McyAdXphP0XQpqGpF+In6fkxLX1H+JURPsXqQ5MKfKSKc3GuJLhs226EVEqG/eC92VdeUBXcD4vHz6bbZ5bHlH7/nbTwr5Q9TKMUunCFOLW/EVP3YiJaPWCfdUO9p2cMbWpQBKavyqG415sIzdRdPRWtsPx/CMvedyE0tKKRJI1A7jed5wUxwLnXkU09AWIf6UMXblxKTwnDOekoKabnFT9AsyEnSZVXGLVebXWu6MmE1cJK7b6InM4YWzGqe9Rcwxtd00X1QgoGJdwDLyEcTZ+zg9crQgEkrYGZS81FFzYLNuZAyliX9ErUHfb3hdOeea9PgV3T4o5a2IQlB8eOFrox7l9M66XC383EmdDLFR23wU1ZwvpuDblw3l9EDwaxoiIN0XIX+1myKTQ== tsabanovic@v-server-github
```

Add this public key to your GitHub account under Settings > SSH and GPG keys.

## Testing

### Security Tests
- [x] SSH key authentication works ✓
  ```bash
  ssh -i ~/.ssh/v_server_key tsabanovic@91.99.193.112
  ```
- [x] Password authentication is disabled ✓
  ```bash
  ssh -o PubkeyAuthentication=no -o PasswordAuthentication=yes tsabanovic@91.99.193.112
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
  curl -I http://91.99.193.112
  # Result: HTTP/1.1 200 OK
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
  # Result: user.name=Tarik Sabanovic, user.email=tsabanovic@example.com
  ```
- [x] GitHub SSH key generated and ready ✓

### Final Validation Results
- **Server IP**: 91.99.193.112 ✅ Accessible
- **SSH Security**: Password login disabled, key-only access ✅
- **Web Server**: NGINX running with custom HTML page ✅
- **Git Setup**: Configured with user credentials ✅
- **GitHub Integration**: SSH key generated for repository access ✅

## Troubleshooting

Common issues and solutions will be documented here as they arise.

---

**Important**: This documentation excludes sensitive information such as private keys and passwords for security reasons.