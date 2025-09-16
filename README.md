# drupal Installation Guide

drupal is a free and open-source content management framework. Drupal provides flexible CMS framework for ambitious digital experiences

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
  - CPU: 2+ cores
  - RAM: 1GB minimum
  - Storage: 2GB for data
  - Network: HTTP/HTTPS access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default drupal port)
  - None
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

# Install drupal
sudo dnf install -y drupal

# Enable and start service
sudo systemctl enable --now drupal

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
drupal --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install drupal
sudo apt install -y drupal

# Enable and start service
sudo systemctl enable --now drupal

# Configure firewall
sudo ufw allow 80

# Verify installation
drupal --version
```

### Arch Linux

```bash
# Install drupal
sudo pacman -S drupal

# Enable and start service
sudo systemctl enable --now drupal

# Verify installation
drupal --version
```

### Alpine Linux

```bash
# Install drupal
apk add --no-cache drupal

# Enable and start service
rc-update add drupal default
rc-service drupal start

# Verify installation
drupal --version
```

### openSUSE/SLES

```bash
# Install drupal
sudo zypper install -y drupal

# Enable and start service
sudo systemctl enable --now drupal

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
drupal --version
```

### macOS

```bash
# Using Homebrew
brew install drupal

# Start service
brew services start drupal

# Verify installation
drupal --version
```

### FreeBSD

```bash
# Using pkg
pkg install drupal

# Enable in rc.conf
echo 'drupal_enable="YES"' >> /etc/rc.conf

# Start service
service drupal start

# Verify installation
drupal --version
```

### Windows

```bash
# Using Chocolatey
choco install drupal

# Or using Scoop
scoop install drupal

# Verify installation
drupal --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/drupal

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
drupal --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable drupal

# Start service
sudo systemctl start drupal

# Stop service
sudo systemctl stop drupal

# Restart service
sudo systemctl restart drupal

# Check status
sudo systemctl status drupal

# View logs
sudo journalctl -u drupal -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add drupal default

# Start service
rc-service drupal start

# Stop service
rc-service drupal stop

# Restart service
rc-service drupal restart

# Check status
rc-service drupal status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'drupal_enable="YES"' >> /etc/rc.conf

# Start service
service drupal start

# Stop service
service drupal stop

# Restart service
service drupal restart

# Check status
service drupal status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start drupal
brew services stop drupal
brew services restart drupal

# Check status
brew services list | grep drupal
```

### Windows Service Manager

```powershell
# Start service
net start drupal

# Stop service
net stop drupal

# Using PowerShell
Start-Service drupal
Stop-Service drupal
Restart-Service drupal

# Check status
Get-Service drupal
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream drupal_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name drupal.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name drupal.example.com;

    ssl_certificate /etc/ssl/certs/drupal.example.com.crt;
    ssl_certificate_key /etc/ssl/private/drupal.example.com.key;

    location / {
        proxy_pass http://drupal_backend;
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
    ServerName drupal.example.com
    Redirect permanent / https://drupal.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName drupal.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/drupal.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/drupal.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend drupal_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/drupal.pem
    redirect scheme https if !{ ssl_fc }
    default_backend drupal_backend

backend drupal_backend
    balance roundrobin
    server drupal1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R drupal:drupal /etc/drupal
sudo chmod 750 /etc/drupal

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
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
sudo systemctl status drupal

# View logs
sudo journalctl -u drupal -f

# Monitor resource usage
top -p $(pgrep drupal)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/drupal"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/drupal-backup-$DATE.tar.gz" /etc/drupal /var/lib/drupal

echo "Backup completed: $BACKUP_DIR/drupal-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop drupal

# Restore from backup
tar -xzf /backup/drupal/drupal-backup-*.tar.gz -C /

# Start service
sudo systemctl start drupal
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u drupal -n 100
sudo tail -f /var/log/drupal/drupal.log

# Check configuration
drupal --version

# Check permissions
ls -la /etc/drupal
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80

# Test connectivity
telnet localhost 80

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep drupal)

# Check disk I/O
iotop -p $(pgrep drupal)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  drupal:
    image: drupal:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/drupal
      - ./data:/var/lib/drupal
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update drupal

# Debian/Ubuntu
sudo apt update && sudo apt upgrade drupal

# Arch Linux
sudo pacman -Syu drupal

# Alpine Linux
apk update && apk upgrade drupal

# openSUSE
sudo zypper update drupal

# FreeBSD
pkg update && pkg upgrade drupal

# Always backup before updates
tar -czf /backup/drupal-pre-update-$(date +%Y%m%d).tar.gz /etc/drupal

# Restart after updates
sudo systemctl restart drupal
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/drupal

# Clean old logs
find /var/log/drupal -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/drupal
```

## Additional Resources

- Official Documentation: https://docs.drupal.org/
- GitHub Repository: https://github.com/drupal/drupal
- Community Forum: https://forum.drupal.org/
- Best Practices Guide: https://docs.drupal.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
