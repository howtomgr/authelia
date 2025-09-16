# authelia Installation Guide

authelia is a free and open-source authentication and authorization server. Authelia provides 2FA and SSO for your applications via a web portal, serving as an open-source alternative to Auth0 or Okta

## Table of Contents
1. [Prerequisites](#prerequisites)
2. [Supported Operating Systems](#supported-operating-systems)
3. [Installation](#installation)
4. [Configuration](#configuration)
5. [Service Management](#service-management)
6. [Troubleshooting](#troubleshooting)
7. [Security Considerations](#security-considerations)
8. [Performance Tuning](#performance-tuning)
9. [Backup and Restore](#backup-and-restore)
10. [System Requirements](#system-requirements)
11. [Support](#support)
12. [Contributing](#contributing)
13. [License](#license)
14. [Acknowledgments](#acknowledgments)
15. [Version History](#version-history)
16. [Appendices](#appendices)

## 1. Prerequisites

- **Hardware Requirements**:
  - CPU: 1 core minimum
  - RAM: 512MB minimum
  - Storage: 100MB for installation
  - Network: HTTPS for authentication
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 9091 (default authelia port)
  - LDAP backend if used
- **Dependencies**:
  - See official documentation for specific requirements
- **System Access**: root or sudo privileges required


## 2. Supported Operating Systems

This guide supports installation on:
- RHEL 8/9 and derivatives (CentOS Stream, Rocky Linux, AlmaLinux)
- Debian 11/12
- Ubuntu 20.04/22.04/24.04 LTS
- Arch Linux (rolling release)
- Alpine Linux 3.18+
- openSUSE Leap 15.5+ / Tumbleweed
- SUSE Linux Enterprise Server (SLES) 15+
- macOS 12+ (Monterey and later) 
- FreeBSD 13+
- Windows 10/11/Server 2019+ (where applicable)

## 3. Installation

### RHEL/CentOS/Rocky Linux/AlmaLinux

```bash
# Install EPEL repository if needed
sudo dnf install -y epel-release

# Install authelia
sudo dnf install -y authelia

# Enable and start service
sudo systemctl enable --now authelia

# Configure firewall
sudo firewall-cmd --permanent --add-port=9091/tcp
sudo firewall-cmd --reload

# Verify installation
authelia --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install authelia
sudo apt install -y authelia

# Enable and start service
sudo systemctl enable --now authelia

# Configure firewall
sudo ufw allow 9091

# Verify installation
authelia --version
```

### Arch Linux

```bash
# Install authelia
sudo pacman -S authelia

# Enable and start service
sudo systemctl enable --now authelia

# Verify installation
authelia --version
```

### Alpine Linux

```bash
# Install authelia
apk add --no-cache authelia

# Enable and start service
rc-update add authelia default
rc-service authelia start

# Verify installation
authelia --version
```

### openSUSE/SLES

```bash
# Install authelia
sudo zypper install -y authelia

# Enable and start service
sudo systemctl enable --now authelia

# Configure firewall
sudo firewall-cmd --permanent --add-port=9091/tcp
sudo firewall-cmd --reload

# Verify installation
authelia --version
```

### macOS

```bash
# Using Homebrew
brew install authelia

# Start service
brew services start authelia

# Verify installation
authelia --version
```

### FreeBSD

```bash
# Using pkg
pkg install authelia

# Enable in rc.conf
echo 'authelia_enable="YES"' >> /etc/rc.conf

# Start service
service authelia start

# Verify installation
authelia --version
```

### Windows

```bash
# Using Chocolatey
choco install authelia

# Or using Scoop
scoop install authelia

# Verify installation
authelia --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/authelia

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
authelia --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable authelia

# Start service
sudo systemctl start authelia

# Stop service
sudo systemctl stop authelia

# Restart service
sudo systemctl restart authelia

# Check status
sudo systemctl status authelia

# View logs
sudo journalctl -u authelia -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add authelia default

# Start service
rc-service authelia start

# Stop service
rc-service authelia stop

# Restart service
rc-service authelia restart

# Check status
rc-service authelia status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'authelia_enable="YES"' >> /etc/rc.conf

# Start service
service authelia start

# Stop service
service authelia stop

# Restart service
service authelia restart

# Check status
service authelia status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start authelia
brew services stop authelia
brew services restart authelia

# Check status
brew services list | grep authelia
```

### Windows Service Manager

```powershell
# Start service
net start authelia

# Stop service
net stop authelia

# Using PowerShell
Start-Service authelia
Stop-Service authelia
Restart-Service authelia

# Check status
Get-Service authelia
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream authelia_backend {
    server 127.0.0.1:9091;
}

server {
    listen 80;
    server_name authelia.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name authelia.example.com;

    ssl_certificate /etc/ssl/certs/authelia.example.com.crt;
    ssl_certificate_key /etc/ssl/private/authelia.example.com.key;

    location / {
        proxy_pass http://authelia_backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

### Apache Configuration

```apache
<VirtualHost *:80>
    ServerName authelia.example.com
    Redirect permanent / https://authelia.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName authelia.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/authelia.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/authelia.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:9091/
    ProxyPassReverse / http://127.0.0.1:9091/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend authelia_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/authelia.pem
    redirect scheme https if !{ ssl_fc }
    default_backend authelia_backend

backend authelia_backend
    balance roundrobin
    server authelia1 127.0.0.1:9091 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R authelia:authelia /etc/authelia
sudo chmod 750 /etc/authelia

# Configure firewall
sudo firewall-cmd --permanent --add-port=9091/tcp
sudo firewall-cmd --reload

# Enable SELinux policies (if applicable)
sudo setsebool -P httpd_can_network_connect on
```

## Database Setup

See official documentation for database configuration requirements.

## Performance Optimization

### System Tuning

```bash
# Basic system tuning
echo 'net.core.somaxconn = 65535' | sudo tee -a /etc/sysctl.conf
echo 'net.ipv4.tcp_max_syn_backlog = 65535' | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

## Monitoring

### Basic Monitoring

```bash
# Check service status
sudo systemctl status authelia

# View logs
sudo journalctl -u authelia -f

# Monitor resource usage
top -p $(pgrep authelia)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/authelia"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/authelia-backup-$DATE.tar.gz" /etc/authelia /var/lib/authelia

echo "Backup completed: $BACKUP_DIR/authelia-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop authelia

# Restore from backup
tar -xzf /backup/authelia/authelia-backup-*.tar.gz -C /

# Start service
sudo systemctl start authelia
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u authelia -n 100
sudo tail -f /var/log/authelia/authelia.log

# Check configuration
authelia --version

# Check permissions
ls -la /etc/authelia
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 9091

# Test connectivity
telnet localhost 9091

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep authelia)

# Check disk I/O
iotop -p $(pgrep authelia)

# Check connections
ss -an | grep 9091
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  authelia:
    image: authelia:latest
    ports:
      - "9091:9091"
    volumes:
      - ./config:/etc/authelia
      - ./data:/var/lib/authelia
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update authelia

# Debian/Ubuntu
sudo apt update && sudo apt upgrade authelia

# Arch Linux
sudo pacman -Syu authelia

# Alpine Linux
apk update && apk upgrade authelia

# openSUSE
sudo zypper update authelia

# FreeBSD
pkg update && pkg upgrade authelia

# Always backup before updates
tar -czf /backup/authelia-pre-update-$(date +%Y%m%d).tar.gz /etc/authelia

# Restart after updates
sudo systemctl restart authelia
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/authelia

# Clean old logs
find /var/log/authelia -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/authelia
```

## Additional Resources

- Official Documentation: https://docs.authelia.org/
- GitHub Repository: https://github.com/authelia/authelia
- Community Forum: https://forum.authelia.org/
- Best Practices Guide: https://docs.authelia.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
