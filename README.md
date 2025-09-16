# saltstack Installation Guide

saltstack is a free and open-source infrastructure automation. SaltStack provides powerful automation for infrastructure management

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
  - Storage: 10GB for data
  - Network: ZeroMQ protocol
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 4505 (default saltstack port)
  - Publish on 4506
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

# Install saltstack
sudo dnf install -y saltstack

# Enable and start service
sudo systemctl enable --now saltstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=4505/tcp
sudo firewall-cmd --reload

# Verify installation
saltstack --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install saltstack
sudo apt install -y saltstack

# Enable and start service
sudo systemctl enable --now saltstack

# Configure firewall
sudo ufw allow 4505

# Verify installation
saltstack --version
```

### Arch Linux

```bash
# Install saltstack
sudo pacman -S saltstack

# Enable and start service
sudo systemctl enable --now saltstack

# Verify installation
saltstack --version
```

### Alpine Linux

```bash
# Install saltstack
apk add --no-cache saltstack

# Enable and start service
rc-update add saltstack default
rc-service saltstack start

# Verify installation
saltstack --version
```

### openSUSE/SLES

```bash
# Install saltstack
sudo zypper install -y saltstack

# Enable and start service
sudo systemctl enable --now saltstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=4505/tcp
sudo firewall-cmd --reload

# Verify installation
saltstack --version
```

### macOS

```bash
# Using Homebrew
brew install saltstack

# Start service
brew services start saltstack

# Verify installation
saltstack --version
```

### FreeBSD

```bash
# Using pkg
pkg install saltstack

# Enable in rc.conf
echo 'saltstack_enable="YES"' >> /etc/rc.conf

# Start service
service saltstack start

# Verify installation
saltstack --version
```

### Windows

```bash
# Using Chocolatey
choco install saltstack

# Or using Scoop
scoop install saltstack

# Verify installation
saltstack --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/saltstack

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
saltstack --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable saltstack

# Start service
sudo systemctl start saltstack

# Stop service
sudo systemctl stop saltstack

# Restart service
sudo systemctl restart saltstack

# Check status
sudo systemctl status saltstack

# View logs
sudo journalctl -u saltstack -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add saltstack default

# Start service
rc-service saltstack start

# Stop service
rc-service saltstack stop

# Restart service
rc-service saltstack restart

# Check status
rc-service saltstack status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'saltstack_enable="YES"' >> /etc/rc.conf

# Start service
service saltstack start

# Stop service
service saltstack stop

# Restart service
service saltstack restart

# Check status
service saltstack status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start saltstack
brew services stop saltstack
brew services restart saltstack

# Check status
brew services list | grep saltstack
```

### Windows Service Manager

```powershell
# Start service
net start saltstack

# Stop service
net stop saltstack

# Using PowerShell
Start-Service saltstack
Stop-Service saltstack
Restart-Service saltstack

# Check status
Get-Service saltstack
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream saltstack_backend {
    server 127.0.0.1:4505;
}

server {
    listen 80;
    server_name saltstack.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name saltstack.example.com;

    ssl_certificate /etc/ssl/certs/saltstack.example.com.crt;
    ssl_certificate_key /etc/ssl/private/saltstack.example.com.key;

    location / {
        proxy_pass http://saltstack_backend;
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
    ServerName saltstack.example.com
    Redirect permanent / https://saltstack.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName saltstack.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/saltstack.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/saltstack.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:4505/
    ProxyPassReverse / http://127.0.0.1:4505/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend saltstack_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/saltstack.pem
    redirect scheme https if !{ ssl_fc }
    default_backend saltstack_backend

backend saltstack_backend
    balance roundrobin
    server saltstack1 127.0.0.1:4505 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R saltstack:saltstack /etc/saltstack
sudo chmod 750 /etc/saltstack

# Configure firewall
sudo firewall-cmd --permanent --add-port=4505/tcp
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
sudo systemctl status saltstack

# View logs
sudo journalctl -u saltstack -f

# Monitor resource usage
top -p $(pgrep saltstack)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/saltstack"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/saltstack-backup-$DATE.tar.gz" /etc/saltstack /var/lib/saltstack

echo "Backup completed: $BACKUP_DIR/saltstack-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop saltstack

# Restore from backup
tar -xzf /backup/saltstack/saltstack-backup-*.tar.gz -C /

# Start service
sudo systemctl start saltstack
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u saltstack -n 100
sudo tail -f /var/log/saltstack/saltstack.log

# Check configuration
saltstack --version

# Check permissions
ls -la /etc/saltstack
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 4505

# Test connectivity
telnet localhost 4505

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep saltstack)

# Check disk I/O
iotop -p $(pgrep saltstack)

# Check connections
ss -an | grep 4505
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  saltstack:
    image: saltstack:latest
    ports:
      - "4505:4505"
    volumes:
      - ./config:/etc/saltstack
      - ./data:/var/lib/saltstack
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update saltstack

# Debian/Ubuntu
sudo apt update && sudo apt upgrade saltstack

# Arch Linux
sudo pacman -Syu saltstack

# Alpine Linux
apk update && apk upgrade saltstack

# openSUSE
sudo zypper update saltstack

# FreeBSD
pkg update && pkg upgrade saltstack

# Always backup before updates
tar -czf /backup/saltstack-pre-update-$(date +%Y%m%d).tar.gz /etc/saltstack

# Restart after updates
sudo systemctl restart saltstack
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/saltstack

# Clean old logs
find /var/log/saltstack -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/saltstack
```

## Additional Resources

- Official Documentation: https://docs.saltstack.org/
- GitHub Repository: https://github.com/saltstack/saltstack
- Community Forum: https://forum.saltstack.org/
- Best Practices Guide: https://docs.saltstack.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
