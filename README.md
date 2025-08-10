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
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

#### 2. Copy Public Key to Server
```bash
# Copy public key to server
ssh-copy-id tsabanovic@91.99.193.112
```

#### 3. Test SSH Key Authentication
```bash
# Test SSH key login
ssh tsabanovic@91.99.193.112
```

### Security Configuration

#### 4. Disable Password Authentication
Edit SSH configuration on the server:
```bash
sudo nano /etc/ssh/sshd_config
```

Key changes:
- `PasswordAuthentication no`
- `PubkeyAuthentication yes`

Restart SSH service:
```bash
sudo systemctl restart ssh
```

### Web Server Installation

#### 5. Install NGINX
```bash
sudo apt update
sudo apt install nginx -y
```

#### 6. Configure Custom HTML Page
Create custom HTML page and configure NGINX to serve it.

### Git Configuration

#### 7. Configure Git User Information
```bash
git config --global user.name "Your Name"
git config --global user.email "your-email@example.com"
```

### GitHub Integration

#### 8. Generate SSH Key for GitHub (Server)
```bash
ssh-keygen -t rsa -b 4096 -C "your-email@example.com"
```

Add public key to GitHub account for repository access.

## Testing

### Security Tests
- [ ] SSH key authentication works
- [ ] Password authentication is disabled
- [ ] Server is accessible via SSH key only

### Web Server Tests
- [ ] NGINX is running and accessible
- [ ] Custom HTML page is displayed
- [ ] Configuration is valid

### Git Tests
- [ ] Git is configured with correct user information
- [ ] GitHub SSH access works from server

## Troubleshooting

Common issues and solutions will be documented here as they arise.

---

**Important**: This documentation excludes sensitive information such as private keys and passwords for security reasons.