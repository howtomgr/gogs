# gogs Installation Guide

gogs is a free and open-source self-hosted Git service. Gogs provides a painless self-hosted Git service written in Go

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
  - RAM: 256MB minimum
  - Storage: 1GB for repos
  - Network: HTTP/SSH access
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 3000 (default gogs port)
  - SSH on port 22
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

# Install gogs
sudo dnf install -y gogs

# Enable and start service
sudo systemctl enable --now gogs

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
gogs --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install gogs
sudo apt install -y gogs

# Enable and start service
sudo systemctl enable --now gogs

# Configure firewall
sudo ufw allow 3000

# Verify installation
gogs --version
```

### Arch Linux

```bash
# Install gogs
sudo pacman -S gogs

# Enable and start service
sudo systemctl enable --now gogs

# Verify installation
gogs --version
```

### Alpine Linux

```bash
# Install gogs
apk add --no-cache gogs

# Enable and start service
rc-update add gogs default
rc-service gogs start

# Verify installation
gogs --version
```

### openSUSE/SLES

```bash
# Install gogs
sudo zypper install -y gogs

# Enable and start service
sudo systemctl enable --now gogs

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
sudo firewall-cmd --reload

# Verify installation
gogs --version
```

### macOS

```bash
# Using Homebrew
brew install gogs

# Start service
brew services start gogs

# Verify installation
gogs --version
```

### FreeBSD

```bash
# Using pkg
pkg install gogs

# Enable in rc.conf
echo 'gogs_enable="YES"' >> /etc/rc.conf

# Start service
service gogs start

# Verify installation
gogs --version
```

### Windows

```bash
# Using Chocolatey
choco install gogs

# Or using Scoop
scoop install gogs

# Verify installation
gogs --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/gogs

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
gogs --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable gogs

# Start service
sudo systemctl start gogs

# Stop service
sudo systemctl stop gogs

# Restart service
sudo systemctl restart gogs

# Check status
sudo systemctl status gogs

# View logs
sudo journalctl -u gogs -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add gogs default

# Start service
rc-service gogs start

# Stop service
rc-service gogs stop

# Restart service
rc-service gogs restart

# Check status
rc-service gogs status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'gogs_enable="YES"' >> /etc/rc.conf

# Start service
service gogs start

# Stop service
service gogs stop

# Restart service
service gogs restart

# Check status
service gogs status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start gogs
brew services stop gogs
brew services restart gogs

# Check status
brew services list | grep gogs
```

### Windows Service Manager

```powershell
# Start service
net start gogs

# Stop service
net stop gogs

# Using PowerShell
Start-Service gogs
Stop-Service gogs
Restart-Service gogs

# Check status
Get-Service gogs
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream gogs_backend {
    server 127.0.0.1:3000;
}

server {
    listen 80;
    server_name gogs.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name gogs.example.com;

    ssl_certificate /etc/ssl/certs/gogs.example.com.crt;
    ssl_certificate_key /etc/ssl/private/gogs.example.com.key;

    location / {
        proxy_pass http://gogs_backend;
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
    ServerName gogs.example.com
    Redirect permanent / https://gogs.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName gogs.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/gogs.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/gogs.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:3000/
    ProxyPassReverse / http://127.0.0.1:3000/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend gogs_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/gogs.pem
    redirect scheme https if !{ ssl_fc }
    default_backend gogs_backend

backend gogs_backend
    balance roundrobin
    server gogs1 127.0.0.1:3000 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R gogs:gogs /etc/gogs
sudo chmod 750 /etc/gogs

# Configure firewall
sudo firewall-cmd --permanent --add-port=3000/tcp
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
sudo systemctl status gogs

# View logs
sudo journalctl -u gogs -f

# Monitor resource usage
top -p $(pgrep gogs)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/gogs"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/gogs-backup-$DATE.tar.gz" /etc/gogs /var/lib/gogs

echo "Backup completed: $BACKUP_DIR/gogs-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop gogs

# Restore from backup
tar -xzf /backup/gogs/gogs-backup-*.tar.gz -C /

# Start service
sudo systemctl start gogs
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u gogs -n 100
sudo tail -f /var/log/gogs/gogs.log

# Check configuration
gogs --version

# Check permissions
ls -la /etc/gogs
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 3000

# Test connectivity
telnet localhost 3000

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep gogs)

# Check disk I/O
iotop -p $(pgrep gogs)

# Check connections
ss -an | grep 3000
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  gogs:
    image: gogs:latest
    ports:
      - "3000:3000"
    volumes:
      - ./config:/etc/gogs
      - ./data:/var/lib/gogs
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update gogs

# Debian/Ubuntu
sudo apt update && sudo apt upgrade gogs

# Arch Linux
sudo pacman -Syu gogs

# Alpine Linux
apk update && apk upgrade gogs

# openSUSE
sudo zypper update gogs

# FreeBSD
pkg update && pkg upgrade gogs

# Always backup before updates
tar -czf /backup/gogs-pre-update-$(date +%Y%m%d).tar.gz /etc/gogs

# Restart after updates
sudo systemctl restart gogs
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/gogs

# Clean old logs
find /var/log/gogs -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/gogs
```

## Additional Resources

- Official Documentation: https://docs.gogs.org/
- GitHub Repository: https://github.com/gogs/gogs
- Community Forum: https://forum.gogs.org/
- Best Practices Guide: https://docs.gogs.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
