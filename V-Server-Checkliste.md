# V-Server Setup Checklist

## Project Requirements - V-Server Setup

This document contains the complete checklist for V-Server setup as provided for the project.

### 1. V-Server Setup Tasks

#### SSH Key Configuration
- [x] Add SSH public key to authorized_keys for your user
  - Use `ssh-copy-id` command from local computer command line
  - Generate key pair on local computer (not on server)
  
#### Security Configuration  
- [x] Disable password login on server
  - Ensure SSH key login works before disabling password authentication
  
#### Web Server Installation
- [x] Install NGINX web server
- [x] Modify NGINX configuration to display alternative HTML page as entry point
  - May require NGINX restart

#### Git Configuration
- [x] Configure git on V-Server with username and email address
  - User data should match your computer/GitHub data
- [x] Create SSH key pair on server for GitHub interaction
  - Add public key to GitHub for repository access

### 2. Guidelines

#### General Guidelines
- Create short Loom video (max 5min) showing setup and explaining steps
- Don't need to mention all details, but cover all relevant steps

#### Security Guidelines  
- Ensure SSH key login works before disabling password authentication

#### Testing Requirements
Before project submission, ensure and test:
- [x] SSH key authentication works for server login
- [x] Username/password combination login does NOT work
  - Use SSH option `-o PubKeyAuthentication=no` for testing
  - Example: `ssh -o PubKeyAuthentication=no -i path/to/private-key user@12.34.67.89`
- [x] Test web server availability by entering V-Server IP in browser
  - Should display NGINX welcome screen with custom page
- [x] Validate configuration with `nginx -t` (config at `/etc/nginx/nginx.conf`)
  - For custom config file: `nginx -t -c path/to/config`
- [x] Validate new configuration is applied and custom HTML page displays in browser

### 3. Documentation Requirements

#### Git Repository with README
- [x] Documentation written in English
- [x] Create Git repository "v-server-setup" documenting complete V-Server setup steps
- [x] Avoid security-critical content (passwords, SSH key contents)
- [x] README in Markdown format with proper structure
- [x] Include table of contents
- [x] Individual steps in sections or separate documents with proper linking
- [x] Include this checklist as PDF in repository
- [x] Provide V-Server IP address for mentor verification

## Implementation Results

### Server Information
- **IP Address**: 91.99.193.112
- **Operating System**: Ubuntu 22.04.5 LTS  
- **Web Server**: NGINX 1.18.0
- **Setup Date**: August 10, 2025

### Completed Tasks
✅ SSH key pair generated on local computer  
✅ SSH public key added to server authorized_keys  
✅ SSH key authentication tested and working  
✅ Password authentication disabled on server  
✅ Security configuration validated  
✅ NGINX web server installed  
✅ Custom HTML page created and deployed  
✅ NGINX configuration tested and validated  
✅ Git configured with user credentials  
✅ GitHub SSH key pair generated on server  
✅ Complete documentation created  
✅ All tests passed successfully  

### Security Validation
- SSH key authentication: ✅ Working
- Password authentication: ✅ Disabled (Permission denied)
- Server accessibility: ✅ SSH key only

### Web Server Validation  
- NGINX status: ✅ Active (running)
- Custom HTML page: ✅ Accessible (HTTP 200)
- Configuration: ✅ Valid (nginx -t successful)

### Git Integration
- User configuration: ✅ Set (Tarik Sabanovic)
- GitHub SSH key: ✅ Generated and ready for GitHub

**Project Status**: ✅ **COMPLETE** - All checklist requirements fulfilled