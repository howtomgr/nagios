# nagios Installation Guide

nagios is a free and open-source system monitoring. Nagios provides comprehensive IT infrastructure monitoring

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
  - RAM: 2GB minimum
  - Storage: 5GB for data
  - Network: HTTP/check protocols
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80 (default nagios port)
  - NRPE on 5666
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

# Install nagios
sudo dnf install -y nagios

# Enable and start service
sudo systemctl enable --now nagios

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
nagios --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install nagios
sudo apt install -y nagios

# Enable and start service
sudo systemctl enable --now nagios

# Configure firewall
sudo ufw allow 80

# Verify installation
nagios --version
```

### Arch Linux

```bash
# Install nagios
sudo pacman -S nagios

# Enable and start service
sudo systemctl enable --now nagios

# Verify installation
nagios --version
```

### Alpine Linux

```bash
# Install nagios
apk add --no-cache nagios

# Enable and start service
rc-update add nagios default
rc-service nagios start

# Verify installation
nagios --version
```

### openSUSE/SLES

```bash
# Install nagios
sudo zypper install -y nagios

# Enable and start service
sudo systemctl enable --now nagios

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/tcp
sudo firewall-cmd --reload

# Verify installation
nagios --version
```

### macOS

```bash
# Using Homebrew
brew install nagios

# Start service
brew services start nagios

# Verify installation
nagios --version
```

### FreeBSD

```bash
# Using pkg
pkg install nagios

# Enable in rc.conf
echo 'nagios_enable="YES"' >> /etc/rc.conf

# Start service
service nagios start

# Verify installation
nagios --version
```

### Windows

```bash
# Using Chocolatey
choco install nagios

# Or using Scoop
scoop install nagios

# Verify installation
nagios --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/nagios

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
nagios --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable nagios

# Start service
sudo systemctl start nagios

# Stop service
sudo systemctl stop nagios

# Restart service
sudo systemctl restart nagios

# Check status
sudo systemctl status nagios

# View logs
sudo journalctl -u nagios -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add nagios default

# Start service
rc-service nagios start

# Stop service
rc-service nagios stop

# Restart service
rc-service nagios restart

# Check status
rc-service nagios status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'nagios_enable="YES"' >> /etc/rc.conf

# Start service
service nagios start

# Stop service
service nagios stop

# Restart service
service nagios restart

# Check status
service nagios status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start nagios
brew services stop nagios
brew services restart nagios

# Check status
brew services list | grep nagios
```

### Windows Service Manager

```powershell
# Start service
net start nagios

# Stop service
net stop nagios

# Using PowerShell
Start-Service nagios
Stop-Service nagios
Restart-Service nagios

# Check status
Get-Service nagios
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream nagios_backend {
    server 127.0.0.1:80;
}

server {
    listen 80;
    server_name nagios.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name nagios.example.com;

    ssl_certificate /etc/ssl/certs/nagios.example.com.crt;
    ssl_certificate_key /etc/ssl/private/nagios.example.com.key;

    location / {
        proxy_pass http://nagios_backend;
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
    ServerName nagios.example.com
    Redirect permanent / https://nagios.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName nagios.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/nagios.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/nagios.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/
    ProxyPassReverse / http://127.0.0.1:80/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend nagios_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/nagios.pem
    redirect scheme https if !{ ssl_fc }
    default_backend nagios_backend

backend nagios_backend
    balance roundrobin
    server nagios1 127.0.0.1:80 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R nagios:nagios /etc/nagios
sudo chmod 750 /etc/nagios

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
sudo systemctl status nagios

# View logs
sudo journalctl -u nagios -f

# Monitor resource usage
top -p $(pgrep nagios)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/nagios"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/nagios-backup-$DATE.tar.gz" /etc/nagios /var/lib/nagios

echo "Backup completed: $BACKUP_DIR/nagios-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop nagios

# Restore from backup
tar -xzf /backup/nagios/nagios-backup-*.tar.gz -C /

# Start service
sudo systemctl start nagios
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u nagios -n 100
sudo tail -f /var/log/nagios/nagios.log

# Check configuration
nagios --version

# Check permissions
ls -la /etc/nagios
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
top -p $(pgrep nagios)

# Check disk I/O
iotop -p $(pgrep nagios)

# Check connections
ss -an | grep 80
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  nagios:
    image: nagios:latest
    ports:
      - "80:80"
    volumes:
      - ./config:/etc/nagios
      - ./data:/var/lib/nagios
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update nagios

# Debian/Ubuntu
sudo apt update && sudo apt upgrade nagios

# Arch Linux
sudo pacman -Syu nagios

# Alpine Linux
apk update && apk upgrade nagios

# openSUSE
sudo zypper update nagios

# FreeBSD
pkg update && pkg upgrade nagios

# Always backup before updates
tar -czf /backup/nagios-pre-update-$(date +%Y%m%d).tar.gz /etc/nagios

# Restart after updates
sudo systemctl restart nagios
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/nagios

# Clean old logs
find /var/log/nagios -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/nagios
```

## Additional Resources

- Official Documentation: https://docs.nagios.org/
- GitHub Repository: https://github.com/nagios/nagios
- Community Forum: https://forum.nagios.org/
- Best Practices Guide: https://docs.nagios.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
