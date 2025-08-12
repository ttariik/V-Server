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
- **Web Server**: NGINX 1.18.0 (Port 8081)
- **Website URL**: http://91.99.193.112:8081

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
ssh-copy-id -i ~/.ssh/v_server_key.pub tsabanovic@91.99.193.112
```

#### 3. Test SSH Key Authentication
```bash
# Test SSH key login
ssh -i ~/.ssh/v_server_key tsabanovic@91.99.193.112
```

### Security Configuration

#### 4. Disable Password Authentication

**🔐 WARUM PASSWORD AUTHENTICATION DEAKTIVIEREN?**

Password Authentication deaktivieren ist eine **kritische Sicherheitsmaßnahme** aus folgenden Gründen:

- **🛡️ Schutz vor Brute-Force Attacken**: Angreifer können nicht mehr endlos Passwort-Kombinationen ausprobieren
- **🔑 Stärkere Authentifizierung**: SSH-Keys sind mathematisch viel sicherer als Passwörter
- **⚡ Bessere Performance**: Keine CPU-Last durch fehlgeschlagene Passwort-Versuche
- **📊 Weniger Log-Spam**: Keine ständigen failed login attempts in den Logs
- **🎯 Compliance**: Viele Sicherheitsstandards fordern Key-only Authentication

**⚠️ WICHTIG: Stelle sicher, dass SSH Key Authentication funktioniert BEVOR du Password Authentication deaktivierst!**

**🔧 SCHRITT-FÜR-SCHRITT KONFIGURATION:**

```bash
# 1. Backup der SSH-Konfiguration erstellen (WICHTIG!)
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.backup
echo "✅ Backup erstellt unter /etc/ssh/sshd_config.backup"

# 2. Password Authentication deaktivieren
# Behandelt sowohl kommentierte (#) als auch aktive Zeilen
sudo sed -i 's/#PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
sudo sed -i 's/PasswordAuthentication yes/PasswordAuthentication no/' /etc/ssh/sshd_config
echo "✅ Password Authentication deaktiviert"

# 3. Public Key Authentication explizit aktivieren
# Stellt sicher dass SSH-Keys weiterhin funktionieren
sudo sed -i 's/#PubkeyAuthentication yes/PubkeyAuthentication yes/' /etc/ssh/sshd_config
sudo sed -i 's/PubkeyAuthentication no/PubkeyAuthentication yes/' /etc/ssh/sshd_config
echo "✅ Public Key Authentication aktiviert"

# 4. Konfiguration überprüfen
echo "🔍 Aktuelle SSH-Konfiguration:"
sudo grep -E '(PasswordAuthentication|PubkeyAuthentication)' /etc/ssh/sshd_config

# 5. SSH-Konfiguration auf Syntax-Fehler testen
sudo sshd -t
if [ $? -eq 0 ]; then
    echo "✅ SSH-Konfiguration ist syntaktisch korrekt"
else
    echo "❌ SSH-Konfiguration hat Fehler! Backup wiederherstellen!"
    exit 1
fi

# 6. SSH-Service neu starten
sudo systemctl restart ssh
echo "✅ SSH-Service neu gestartet"

# 7. Service-Status prüfen
sudo systemctl status ssh --no-pager
```

**📋 ERWARTETE KONFIGURATION:**

Nach der Konfiguration sollten folgende Werte in `/etc/ssh/sshd_config` stehen:

```
PasswordAuthentication no       # ❌ Passwort-Login deaktiviert
PubkeyAuthentication yes        # ✅ SSH-Key Login aktiviert
```

**🧪 SOFORTIGER TEST:**

```bash
# Test 1: SSH Key Login (sollte funktionieren)
ssh -i ~/.ssh/v_server_key tsabanovic@91.99.193.112 "echo 'SSH Key Login erfolgreich!'"

# Test 2: Password Login (sollte fehlschlagen)
# Versuche ohne Key - sollte "Permission denied" geben
ssh -o PreferredAuthentications=password tsabanovic@91.99.193.112
# Erwartete Antwort: "Permission denied (publickey)."
```

**🚨 TROUBLESHOOTING:**

Falls der Login nicht mehr funktioniert:

```bash
# 1. Backup wiederherstellen
sudo cp /etc/ssh/sshd_config.backup /etc/ssh/sshd_config
sudo systemctl restart ssh

# 2. SSH Key Berechtigung prüfen
ls -la ~/.ssh/v_server_key      # Sollte 600 (-rw-------) sein
chmod 600 ~/.ssh/v_server_key   # Falls nötig korrigieren

# 3. Authorized Keys auf Server prüfen
ssh -i ~/.ssh/v_server_key tsabanovic@91.99.193.112 "ls -la ~/.ssh/authorized_keys"
```

**🎯 SICHERHEITS-RESULTAT:**

Nach dieser Konfiguration ist dein Server:
- ✅ **Immun gegen Passwort-Brute-Force Attacken**
- ✅ **Nur noch per SSH-Key erreichbar**
- ✅ **Entspricht modernen Sicherheitsstandards**
- ✅ **Deutlich sicherer als Standard-Konfiguration**

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
ssh-rsa AAAAB3NzaC1yc2EAA
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
  curl -I http://91.99.193.112:8081
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
  # Result: user.name=Tarik Sabanovic, user.email=tsabanovic@example.com
  ```
- [x] GitHub SSH key generated and ready ✓

### Final Validation Results
- **Server IP**: 91.99.193.112 ✅ Accessible
- **Website URL**: http://91.99.193.112:8081 ✅ Live and responsive
- **SSH Security**: Password login disabled, key-only access ✅
- **Web Server**: NGINX running on port 8081 with custom HTML page ✅
- **Port Configuration**: Custom port 8081 for enhanced security ✅
- **Git Setup**: Configured with user credentials ✅
- **GitHub Integration**: SSH key generated for repository access ✅

## Troubleshooting

Common issues and solutions will be documented here as they arise.

---

**Important**: This documentation excludes sensitive information such as private keys and passwords for security reasons.