# vaultwarden Installation Guide

vaultwarden is a free and open-source unofficial Bitwarden server implementation. Written in Rust, Vaultwarden is a lightweight alternative implementation of the Bitwarden server API, perfect for self-hosting password management

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
  - Storage: 1GB for data
  - Network: HTTPS required
- **Operating System**: 
  - Linux: Any modern distribution (RHEL, Debian, Ubuntu, CentOS, Fedora, Arch, Alpine, openSUSE)
  - macOS: 10.14+ (Mojave or newer)
  - Windows: Windows Server 2016+ or Windows 10
  - FreeBSD: 11.0+
- **Network Requirements**:
  - Port 80/443 (default vaultwarden port)
  - WebSocket on 3012
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

# Install vaultwarden
sudo dnf install -y vaultwarden

# Enable and start service
sudo systemctl enable --now vaultwarden

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
vaultwarden --version
```

### Debian/Ubuntu

```bash
# Update package index
sudo apt update

# Install vaultwarden
sudo apt install -y vaultwarden

# Enable and start service
sudo systemctl enable --now vaultwarden

# Configure firewall
sudo ufw allow 80/443

# Verify installation
vaultwarden --version
```

### Arch Linux

```bash
# Install vaultwarden
sudo pacman -S vaultwarden

# Enable and start service
sudo systemctl enable --now vaultwarden

# Verify installation
vaultwarden --version
```

### Alpine Linux

```bash
# Install vaultwarden
apk add --no-cache vaultwarden

# Enable and start service
rc-update add vaultwarden default
rc-service vaultwarden start

# Verify installation
vaultwarden --version
```

### openSUSE/SLES

```bash
# Install vaultwarden
sudo zypper install -y vaultwarden

# Enable and start service
sudo systemctl enable --now vaultwarden

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
sudo firewall-cmd --reload

# Verify installation
vaultwarden --version
```

### macOS

```bash
# Using Homebrew
brew install vaultwarden

# Start service
brew services start vaultwarden

# Verify installation
vaultwarden --version
```

### FreeBSD

```bash
# Using pkg
pkg install vaultwarden

# Enable in rc.conf
echo 'vaultwarden_enable="YES"' >> /etc/rc.conf

# Start service
service vaultwarden start

# Verify installation
vaultwarden --version
```

### Windows

```bash
# Using Chocolatey
choco install vaultwarden

# Or using Scoop
scoop install vaultwarden

# Verify installation
vaultwarden --version
```

## Initial Configuration

### Basic Configuration

```bash
# Create configuration directory
sudo mkdir -p /etc/vaultwarden

# Set up basic configuration
# See official documentation for detailed configuration options

# Test configuration
vaultwarden --version
```

## 5. Service Management

### systemd (RHEL, Debian, Ubuntu, Arch, openSUSE)

```bash
# Enable service
sudo systemctl enable vaultwarden

# Start service
sudo systemctl start vaultwarden

# Stop service
sudo systemctl stop vaultwarden

# Restart service
sudo systemctl restart vaultwarden

# Check status
sudo systemctl status vaultwarden

# View logs
sudo journalctl -u vaultwarden -f
```

### OpenRC (Alpine Linux)

```bash
# Enable service
rc-update add vaultwarden default

# Start service
rc-service vaultwarden start

# Stop service
rc-service vaultwarden stop

# Restart service
rc-service vaultwarden restart

# Check status
rc-service vaultwarden status
```

### rc.d (FreeBSD)

```bash
# Enable in /etc/rc.conf
echo 'vaultwarden_enable="YES"' >> /etc/rc.conf

# Start service
service vaultwarden start

# Stop service
service vaultwarden stop

# Restart service
service vaultwarden restart

# Check status
service vaultwarden status
```

### launchd (macOS)

```bash
# Using Homebrew services
brew services start vaultwarden
brew services stop vaultwarden
brew services restart vaultwarden

# Check status
brew services list | grep vaultwarden
```

### Windows Service Manager

```powershell
# Start service
net start vaultwarden

# Stop service
net stop vaultwarden

# Using PowerShell
Start-Service vaultwarden
Stop-Service vaultwarden
Restart-Service vaultwarden

# Check status
Get-Service vaultwarden
```

## Advanced Configuration

See the official documentation for advanced configuration options.

## Reverse Proxy Setup

### nginx Configuration

```nginx
upstream vaultwarden_backend {
    server 127.0.0.1:80/443;
}

server {
    listen 80;
    server_name vaultwarden.example.com;
    return 301 https://$server_name$request_uri;
}

server {
    listen 443 ssl http2;
    server_name vaultwarden.example.com;

    ssl_certificate /etc/ssl/certs/vaultwarden.example.com.crt;
    ssl_certificate_key /etc/ssl/private/vaultwarden.example.com.key;

    location / {
        proxy_pass http://vaultwarden_backend;
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
    ServerName vaultwarden.example.com
    Redirect permanent / https://vaultwarden.example.com/
</VirtualHost>

<VirtualHost *:443>
    ServerName vaultwarden.example.com
    
    SSLEngine on
    SSLCertificateFile /etc/ssl/certs/vaultwarden.example.com.crt
    SSLCertificateKeyFile /etc/ssl/private/vaultwarden.example.com.key
    
    ProxyRequests Off
    ProxyPreserveHost On
    
    ProxyPass / http://127.0.0.1:80/443/
    ProxyPassReverse / http://127.0.0.1:80/443/
</VirtualHost>
```

### HAProxy Configuration

```haproxy
frontend vaultwarden_frontend
    bind *:80
    bind *:443 ssl crt /etc/ssl/certs/vaultwarden.pem
    redirect scheme https if !{ ssl_fc }
    default_backend vaultwarden_backend

backend vaultwarden_backend
    balance roundrobin
    server vaultwarden1 127.0.0.1:80/443 check
```

## Security Configuration

### Basic Security Setup

```bash
# Set appropriate permissions
sudo chown -R vaultwarden:vaultwarden /etc/vaultwarden
sudo chmod 750 /etc/vaultwarden

# Configure firewall
sudo firewall-cmd --permanent --add-port=80/443/tcp
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
sudo systemctl status vaultwarden

# View logs
sudo journalctl -u vaultwarden -f

# Monitor resource usage
top -p $(pgrep vaultwarden)
```

## 9. Backup and Restore

### Backup Script

```bash
#!/bin/bash
# Basic backup script
BACKUP_DIR="/backup/vaultwarden"
DATE=$(date +%Y%m%d_%H%M%S)

mkdir -p "$BACKUP_DIR"
tar -czf "$BACKUP_DIR/vaultwarden-backup-$DATE.tar.gz" /etc/vaultwarden /var/lib/vaultwarden

echo "Backup completed: $BACKUP_DIR/vaultwarden-backup-$DATE.tar.gz"
```

### Restore Procedure

```bash
# Stop service
sudo systemctl stop vaultwarden

# Restore from backup
tar -xzf /backup/vaultwarden/vaultwarden-backup-*.tar.gz -C /

# Start service
sudo systemctl start vaultwarden
```

## 6. Troubleshooting

### Common Issues

1. **Service won't start**:
```bash
# Check logs
sudo journalctl -u vaultwarden -n 100
sudo tail -f /var/log/vaultwarden/vaultwarden.log

# Check configuration
vaultwarden --version

# Check permissions
ls -la /etc/vaultwarden
```

2. **Connection issues**:
```bash
# Check if service is listening
sudo ss -tlnp | grep 80/443

# Test connectivity
telnet localhost 80/443

# Check firewall
sudo firewall-cmd --list-all
```

3. **Performance issues**:
```bash
# Check resource usage
top -p $(pgrep vaultwarden)

# Check disk I/O
iotop -p $(pgrep vaultwarden)

# Check connections
ss -an | grep 80/443
```

## Integration Examples

### Docker Compose Example

```yaml
version: '3.8'
services:
  vaultwarden:
    image: vaultwarden:latest
    ports:
      - "80/443:80/443"
    volumes:
      - ./config:/etc/vaultwarden
      - ./data:/var/lib/vaultwarden
    restart: unless-stopped
```

## Maintenance

### Update Procedures

```bash
# RHEL/CentOS/Rocky/AlmaLinux
sudo dnf update vaultwarden

# Debian/Ubuntu
sudo apt update && sudo apt upgrade vaultwarden

# Arch Linux
sudo pacman -Syu vaultwarden

# Alpine Linux
apk update && apk upgrade vaultwarden

# openSUSE
sudo zypper update vaultwarden

# FreeBSD
pkg update && pkg upgrade vaultwarden

# Always backup before updates
tar -czf /backup/vaultwarden-pre-update-$(date +%Y%m%d).tar.gz /etc/vaultwarden

# Restart after updates
sudo systemctl restart vaultwarden
```

### Regular Maintenance

```bash
# Log rotation
sudo logrotate -f /etc/logrotate.d/vaultwarden

# Clean old logs
find /var/log/vaultwarden -name "*.log" -mtime +30 -delete

# Check disk usage
du -sh /var/lib/vaultwarden
```

## Additional Resources

- Official Documentation: https://docs.vaultwarden.org/
- GitHub Repository: https://github.com/vaultwarden/vaultwarden
- Community Forum: https://forum.vaultwarden.org/
- Best Practices Guide: https://docs.vaultwarden.org/best-practices

---

**Note:** This guide is part of the [HowToMgr](https://howtomgr.github.io) collection. Always refer to official documentation for the most up-to-date information.
